# 入札公告要件判定システム - ワークフロー詳細設計書

## 関連ドキュメント
- **全体設計書**: `/Project_design.md`
- **要件定義書**: `/docs/requirements/要件定義書.md`
- **TODO リスト**: `/docs/todo/Client_TODO.md`

---

## 1. ワークフロー概要

### 1.1 目的
100件以上の入札公告を効率的に処理し、GASの実行時間制限（6分）を考慮した堅牢なバッチ処理システムを実現する。

### 1.2 設計方針
- **非同期処理**: 長時間処理に対応するトリガーベースの再開機能
- **状態管理**: 処理進捗の永続化と復元
- **エラー耐性**: 部分的な失敗を許容し、処理を継続

---

## 2. ワークフロー全体像

### 2.1 処理ステージ
```
┌─────────────────────────────────────────────────────────────┐
│                    ワークフローステージ                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Stage 1: 初期化                                           │
│  ├─ 入力検証                                               │
│  ├─ 状態初期化                                             │
│  └─ トリガー設定                                           │
│                                                             │
│  Stage 2: データ準備                                       │
│  ├─ 公告データ読込                                         │
│  ├─ バッチ分割                                             │
│  └─ 企業リスト作成                                         │
│                                                             │
│  Stage 3: 転記処理                                         │
│  ├─ 判定前→入札公告マスター                               │
│  ├─ データ正規化                                           │
│  └─ チェックポイント保存                                   │
│                                                             │
│  Stage 4: OCR処理                                          │
│  ├─ PDF解析要求                                            │
│  ├─ 処理状態監視                                           │
│  └─ 結果取得・保存                                         │
│                                                             │
│  Stage 5: 要件分類                                         │
│  ├─ LLM要件抽出                                            │
│  ├─ 要件種別判定                                           │
│  └─ 公告要件マスター登録                                   │
│                                                             │
│  Stage 6: 判定処理                                         │
│  ├─ 欠格要件判定（一括）                                   │
│  ├─ 中間テーブル登録                                       │
│  └─ その他要件判定（個別）                                 │
│                                                             │
│  Stage 7: レポート生成                                     │
│  ├─ 結果集計                                               │
│  ├─ レポート作成                                           │
│  └─ 通知送信                                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 状態遷移図
```
[初期状態]
    │
    ├─→ [初期化中]
    │       ├─→ [エラー] → [終了]
    │       └─→ [準備完了]
    │
    ├─→ [転記処理中]
    │       ├─→ [タイムアウト] → [中断・再開待ち]
    │       └─→ [転記完了]
    │
    ├─→ [OCR処理中]
    │       ├─→ [タイムアウト] → [中断・再開待ち]
    │       ├─→ [API エラー] → [リトライ待ち]
    │       └─→ [OCR完了]
    │
    ├─→ [要件分類中]
    │       ├─→ [タイムアウト] → [中断・再開待ち]
    │       └─→ [分類完了]
    │
    ├─→ [判定処理中]
    │       ├─→ [タイムアウト] → [中断・再開待ち]
    │       └─→ [判定完了]
    │
    └─→ [完了]
            └─→ [レポート生成済]
```

---

## 3. MainController 詳細設計

### 3.1 クラス構成
```javascript
/**
 * ワークフロー制御クラス
 */
class MainController {
  constructor() {
    this.stateManager = new StateManager();
    this.timeoutHandler = new TimeoutHandler();
    this.batchProcessor = new BatchProcessor();
  }

  /**
   * ワークフロー実行メイン
   * @param {Object} options - 実行オプション
   * @returns {Object} 実行結果
   */
  executeWorkflow(options = {}) {
    try {
      // 1. 実行コンテキストの準備
      const context = this.prepareContext(options);
      
      // 2. 状態の復元または初期化
      const state = this.stateManager.loadOrInitialize(context);
      
      // 3. タイムアウトハンドラー設定
      this.timeoutHandler.setup(state);
      
      // 4. ステージ別処理実行
      this.processStages(state);
      
      // 5. 完了処理
      return this.finalizeWorkflow(state);
      
    } catch (error) {
      this.handleFatalError(error);
      throw error;
    }
  }
}
```

### 3.2 主要メソッド詳細

#### 3.2.1 prepareContext
```javascript
/**
 * 実行コンテキストの準備
 * @private
 */
prepareContext(options) {
  return {
    executionId: options.executionId || Utilities.getUuid(),
    isResume: options.isResume || false,
    startTime: new Date(),
    config: this.loadConfig(),
    user: Session.getActiveUser().getEmail()
  };
}
```

#### 3.2.2 processStages
```javascript
/**
 * ステージ別処理実行
 * @private
 */
processStages(state) {
  const stages = [
    { name: 'TRANSFER', handler: this.processTransfer },
    { name: 'OCR', handler: this.processOCR },
    { name: 'CLASSIFY', handler: this.processClassify },
    { name: 'JUDGE', handler: this.processJudgment },
    { name: 'REPORT', handler: this.processReport }
  ];

  for (const stage of stages) {
    if (state.currentStage === stage.name || 
        state.completedStages.includes(stage.name)) {
      continue;
    }

    state.currentStage = stage.name;
    this.stateManager.save(state);

    const result = stage.handler.call(this, state);
    
    if (result.status === 'TIMEOUT') {
      this.handleTimeout(state);
      return;
    }

    state.completedStages.push(stage.name);
    state.stageResults[stage.name] = result;
  }
}
```

---

## 4. StateManager 詳細設計

### 4.1 状態管理スキーマ
```javascript
/**
 * ワークフロー状態スキーマ
 */
const WorkflowState = {
  // 基本情報
  executionId: String,        // 実行ID
  startTime: Date,           // 開始時刻
  lastUpdateTime: Date,      // 最終更新時刻
  
  // 進捗情報
  currentStage: String,      // 現在のステージ
  completedStages: Array,    // 完了済みステージ
  currentBatch: Number,      // 現在のバッチ番号
  totalBatches: Number,      // 総バッチ数
  
  // 処理データ
  announcements: Array,      // 公告リスト
  processedCount: Number,    // 処理済み件数
  errorCount: Number,        // エラー件数
  
  // ステージ別結果
  stageResults: {
    TRANSFER: Object,
    OCR: Object,
    CLASSIFY: Object,
    JUDGE: Object,
    REPORT: Object
  },
  
  // エラー情報
  errors: Array,
  
  // チェックポイント
  checkpoints: {
    lastProcessedRow: Number,
    lastOCRRequestId: String,
    lastJudgmentId: String
  }
};
```

### 4.2 状態永続化
```javascript
class StateManager {
  constructor() {
    this.storage = PropertiesService.getScriptProperties();
    this.cacheService = CacheService.getScriptCache();
  }

  /**
   * 状態の保存
   */
  save(state) {
    // 1. キャッシュに保存（高速アクセス用）
    this.cacheService.put(
      `workflow_state_${state.executionId}`,
      JSON.stringify(state),
      21600 // 6時間
    );
    
    // 2. PropertiesServiceに保存（永続化）
    this.storage.setProperty(
      `workflow_state_${state.executionId}`,
      JSON.stringify(state)
    );
    
    // 3. ログシートにも記録
    this.logStateToSheet(state);
  }

  /**
   * 状態の読み込み
   */
  load(executionId) {
    // 1. キャッシュから取得を試みる
    let stateJson = this.cacheService.get(`workflow_state_${executionId}`);
    
    // 2. キャッシュになければPropertiesServiceから
    if (!stateJson) {
      stateJson = this.storage.getProperty(`workflow_state_${executionId}`);
    }
    
    return stateJson ? JSON.parse(stateJson) : null;
  }
}
```

---

## 5. TimeoutHandler 詳細設計

### 5.1 タイムアウト管理
```javascript
class TimeoutHandler {
  constructor() {
    this.SAFE_EXECUTION_TIME = 300000; // 5分（ミリ秒）
    this.TRIGGER_DELAY = 60000; // 1分後に再実行
  }

  /**
   * タイムアウトチェック
   */
  checkTimeout(startTime) {
    const elapsed = Date.now() - startTime.getTime();
    return elapsed >= this.SAFE_EXECUTION_TIME;
  }

  /**
   * 再開トリガーの設定
   */
  setResumeTrigger(state) {
    // 既存のトリガーを削除
    this.deleteExistingTriggers(state.executionId);
    
    // 新しいトリガーを作成
    const trigger = ScriptApp.newTrigger('resumeWorkflow')
      .timeBased()
      .after(this.TRIGGER_DELAY)
      .create();
    
    // トリガーIDを状態に保存
    state.resumeTriggerId = trigger.getUniqueId();
    
    // トリガーに実行IDを紐付け
    PropertiesService.getScriptProperties().setProperty(
      `trigger_${trigger.getUniqueId()}`,
      state.executionId
    );
  }

  /**
   * トリガーのクリーンアップ
   */
  cleanup(state) {
    if (state.resumeTriggerId) {
      const triggers = ScriptApp.getProjectTriggers();
      triggers.forEach(trigger => {
        if (trigger.getUniqueId() === state.resumeTriggerId) {
          ScriptApp.deleteTrigger(trigger);
        }
      });
    }
  }
}
```

### 5.2 再開処理
```javascript
/**
 * ワークフロー再開エントリーポイント
 */
function resumeWorkflow(e) {
  const triggerId = e.triggerUid;
  const executionId = PropertiesService.getScriptProperties()
    .getProperty(`trigger_${triggerId}`);
  
  if (!executionId) {
    console.error('実行IDが見つかりません');
    return;
  }
  
  const controller = new MainController();
  controller.executeWorkflow({
    executionId: executionId,
    isResume: true
  });
}
```

---

## 6. BatchProcessor 詳細設計

### 6.1 バッチ分割ロジック
```javascript
class BatchProcessor {
  constructor() {
    this.OPTIMAL_BATCH_SIZE = 20; // 基本バッチサイズ
    this.MAX_BATCH_SIZE = 50;     // 最大バッチサイズ
  }

  /**
   * 動的バッチサイズ決定
   */
  determineBatchSize(totalCount, averageProcessingTime) {
    // 処理時間に基づいて動的に調整
    const availableTime = 300000; // 5分
    const estimatedBatchSize = Math.floor(
      availableTime / averageProcessingTime
    );
    
    return Math.min(
      Math.max(this.OPTIMAL_BATCH_SIZE, estimatedBatchSize),
      this.MAX_BATCH_SIZE
    );
  }

  /**
   * データのバッチ分割
   */
  splitIntoBatches(data, batchSize) {
    const batches = [];
    for (let i = 0; i < data.length; i += batchSize) {
      batches.push({
        batchNumber: Math.floor(i / batchSize) + 1,
        items: data.slice(i, i + batchSize),
        startIndex: i,
        endIndex: Math.min(i + batchSize - 1, data.length - 1)
      });
    }
    return batches;
  }

  /**
   * バッチ処理実行
   */
  processBatch(batch, processorFunction, state) {
    const results = [];
    const errors = [];

    for (const item of batch.items) {
      try {
        const result = processorFunction(item, state);
        results.push(result);
        state.processedCount++;
      } catch (error) {
        errors.push({
          item: item,
          error: error.message,
          timestamp: new Date()
        });
        state.errorCount++;
      }

      // 定期的に状態を保存
      if (state.processedCount % 5 === 0) {
        new StateManager().save(state);
      }
    }

    return { results, errors };
  }
}
```

---

## 7. エラーハンドリング詳細

### 7.1 エラー分類と対応
```javascript
class ErrorHandler {
  static handleError(error, context) {
    const errorType = this.classifyError(error);
    
    switch (errorType) {
      case 'TIMEOUT':
        return { action: 'SUSPEND', retry: true };
        
      case 'API_RATE_LIMIT':
        return { action: 'WAIT', retryAfter: 60000 };
        
      case 'NETWORK_ERROR':
        return { action: 'RETRY', maxRetries: 3 };
        
      case 'DATA_VALIDATION':
        return { action: 'SKIP', log: true };
        
      case 'FATAL':
        return { action: 'ABORT', notify: true };
        
      default:
        return { action: 'LOG', continue: true };
    }
  }

  static classifyError(error) {
    if (error.message.includes('Exceeded maximum execution time')) {
      return 'TIMEOUT';
    }
    if (error.message.includes('429') || error.message.includes('rate limit')) {
      return 'API_RATE_LIMIT';
    }
    if (error.message.includes('UrlFetch') || error.message.includes('timeout')) {
      return 'NETWORK_ERROR';
    }
    if (error.name === 'ValidationError') {
      return 'DATA_VALIDATION';
    }
    if (error.name === 'TypeError' || error.name === 'ReferenceError') {
      return 'FATAL';
    }
    return 'UNKNOWN';
  }
}
```

---

## 8. 進捗管理とモニタリング

### 8.1 進捗追跡
```javascript
class ProgressTracker {
  constructor(totalItems) {
    this.totalItems = totalItems;
    this.processedItems = 0;
    this.startTime = new Date();
    this.checkpoints = [];
  }

  /**
   * 進捗更新
   */
  update(count = 1) {
    this.processedItems += count;
    
    // チェックポイント記録（10%ごと）
    const percentage = (this.processedItems / this.totalItems) * 100;
    const milestone = Math.floor(percentage / 10) * 10;
    
    if (milestone > this.lastMilestone) {
      this.checkpoints.push({
        milestone: milestone,
        timestamp: new Date(),
        processedCount: this.processedItems,
        elapsedTime: Date.now() - this.startTime.getTime()
      });
      this.lastMilestone = milestone;
    }
  }

  /**
   * 推定完了時間
   */
  getEstimatedCompletion() {
    if (this.processedItems === 0) return null;
    
    const elapsedTime = Date.now() - this.startTime.getTime();
    const averageTime = elapsedTime / this.processedItems;
    const remainingItems = this.totalItems - this.processedItems;
    const estimatedRemaining = remainingItems * averageTime;
    
    return new Date(Date.now() + estimatedRemaining);
  }

  /**
   * 進捗レポート生成
   */
  getProgressReport() {
    return {
      percentage: (this.processedItems / this.totalItems) * 100,
      processed: this.processedItems,
      total: this.totalItems,
      remaining: this.totalItems - this.processedItems,
      elapsedTime: Date.now() - this.startTime.getTime(),
      estimatedCompletion: this.getEstimatedCompletion(),
      averageProcessingTime: this.getAverageProcessingTime(),
      checkpoints: this.checkpoints
    };
  }
}
```

---

## 9. 設定管理

### 9.1 設定スキーマ
```javascript
const WorkflowConfig = {
  // 実行設定
  execution: {
    maxExecutionTime: 300000,      // 5分
    triggerDelay: 60000,           // 1分
    maxRetries: 3,
    retryDelay: 30000              // 30秒
  },
  
  // バッチ設定
  batch: {
    defaultSize: 20,
    maxSize: 50,
    minSize: 5
  },
  
  // API設定
  api: {
    ocr: {
      endpoint: 'OCR_API_ENDPOINT',
      timeout: 30000,
      maxConcurrent: 5
    },
    llm: {
      endpoint: 'LLM_API_ENDPOINT',
      timeout: 60000,
      maxTokens: 4000
    }
  },
  
  // ログ設定
  logging: {
    level: 'INFO',
    maxLogEntries: 10000,
    logSheetName: 'LOG'
  },
  
  // 通知設定
  notification: {
    enabled: true,
    recipients: ['admin@example.com'],
    onError: true,
    onCompletion: true
  }
};
```

---

## 10. テスト計画

### 10.1 単体テスト項目
- StateManager の save/load 動作
- TimeoutHandler のトリガー設定/削除
- BatchProcessor のバッチ分割ロジック
- ErrorHandler のエラー分類

### 10.2 結合テスト項目
- タイムアウト発生時の状態保存と再開
- 大量データ（200件）の処理
- エラー発生時の継続処理
- 進捗追跡の正確性

### 10.3 性能テスト項目
- 100件処理の所要時間
- メモリ使用量の推移
- API呼び出しの並列度

---

## 改訂履歴
- 2024-01-XX: 初版作成