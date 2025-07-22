# 入札公告要件判定システム - クライアント側TODO

## 関連ドキュメント
- **全体TODO**: `/Project_TODO.md`
- **設計書**: `/docs/design/Workflow_design.md`
- **UI設計**: `/docs/design/UI_design.md`

---

## 概要
クライアント側（利用者向けスプレッドシート）の開発タスク一覧。チェックボックスで進捗管理。

---

## 1. 環境設定 & 初期構築

### 1.1 プロジェクト設定
[ ] GASプロジェクト（クライアント側）の作成
[ ] サーバー側ライブラリの参照設定
[ ] プロジェクトプロパティの設定
[ ] 実行権限の設定

### 1.2 スプレッドシート準備
[ ] クライアント用スプレッドシートの作成
[ ] 必要なシートの追加
  [ ] 判定前公告一覧
  [ ] 入札公告マスター
  [ ] 公告要件マスター
  [ ] 企業公告判定マスター
  [ ] 充足要件リスト
  [ ] 不足要件リスト
  [ ] 要件拠点中間テーブル
  [ ] 報告用シート
  [ ] LOG
  [ ] 設定

### 1.3 シートフォーマット定義
[ ] 各シートのヘッダー設定
[ ] データ検証ルールの設定
[ ] 条件付き書式の設定
[ ] 保護範囲の設定

---

## 2. UI実装

### 2.1 UIManager.gs（メニュー管理）
```javascript
[ ] // スプレッドシート開始時処理
    function onOpen() {
      // カスタムメニュー作成
      // 初期化処理
      // 権限チェック
    }

[ ] // カスタムメニュー作成
    function createCustomMenu() {
      const ui = SpreadsheetApp.getUi();
      ui.createMenu('入札公告判定')
        .addItem('判定実行', 'executeJudgment')
        .addItem('レポート生成', 'generateReport')
        .addSeparator()
        .addItem('設定', 'showSettings')
        .addItem('ヘルプ', 'showHelp')
        .addToUi();
    }

[ ] // 設定ダイアログ
    function showSettingsDialog() {
      // HTMLサービスでダイアログ作成
      // 設定項目の表示/編集
    }

[ ] // ヘルプダイアログ
    function showHelpDialog() {
      // 使い方ガイド表示
    }
```

### 2.2 進捗表示UI
```javascript
[ ] // プログレスバー表示
    function showProgressBar(title, message) {
      // モーダルダイアログ表示
      // 進捗率の更新機能
    }

[ ] // 進捗更新
    function updateProgress(percentage, message) {
      // プログレスバー更新
      // メッセージ更新
    }

[ ] // 完了ダイアログ
    function showCompletionDialog(results) {
      // 処理結果サマリー表示
      // 次のアクション選択
    }

[ ] // エラーダイアログ
    function showErrorDialog(error) {
      // エラー内容表示
      // 対処法の提示
    }
```

### 2.3 HTML/CSS作成
```html
[ ] <!-- settings.html -->
    <div class="settings-container">
      <h2>システム設定</h2>
      <div class="form-group">
        <label>バッチサイズ:</label>
        <input type="number" id="batchSize" min="5" max="50">
      </div>
      <!-- その他設定項目 -->
    </div>

[ ] <!-- progress.html -->
    <div class="progress-container">
      <h3 id="progressTitle">処理中...</h3>
      <div class="progress-bar">
        <div class="progress-fill" id="progressFill"></div>
      </div>
      <p id="progressMessage"></p>
    </div>

[ ] <!-- style.css -->
    .progress-bar {
      width: 100%;
      height: 20px;
      background-color: #f0f0f0;
      border-radius: 10px;
    }
    .progress-fill {
      height: 100%;
      background-color: #4CAF50;
      transition: width 0.3s ease;
    }
```

---

## 3. データ管理

### 3.1 DataManager.gs（データ読み書き）
```javascript
[ ] // 判定前公告読み込み
    function loadPreAnnouncements() {
      // シート取得
      // データ読み込み
      // オブジェクト変換
      // バリデーション
    }

[ ] // 入札公告マスター保存
    function saveAnnouncementMaster(announcements) {
      // データ整形
      // シート書き込み
      // 書式設定
    }

[ ] // 判定結果保存
    function saveJudgmentResults(results) {
      // 評価マスター更新
      // 充足/不足リスト更新
      // タイムスタンプ記録
    }

[ ] // レポートデータ作成
    function createReportData(results) {
      // 集計処理
      // グラフデータ生成
      // サマリー作成
    }
```

### 3.2 DataValidator.gs（入力検証）
```javascript
[ ] // 必須項目チェック
    function validateRequiredFields(data) {
      const required = ['案件名', '公告URL', '企業ID'];
      // 各項目の存在確認
      // エラーメッセージ生成
    }

[ ] // データ形式チェック
    function validateDataFormat(data) {
      // URL形式検証
      // 日付形式検証
      // 数値形式検証
    }

[ ] // 日付項目検証
    function validateDateFields(data) {
      // 開札日
      // 工期開始/終了
      // 論理チェック（開始<終了）
    }

[ ] // データクレンジング
    function cleanInputData(data) {
      // 空白除去
      // 全角半角統一
      // 改行コード統一
    }

[ ] // 重複チェック
    function checkDuplicates(announcements) {
      // 案件名重複
      // URL重複
      // 警告表示
    }
```

---

## 4. ワークフロー制御

### 4.1 MainController.gs（メイン制御）
```javascript
[ ] // ワークフロー実行エントリーポイント
    function executeWorkflow() {
      try {
        // UI初期化
        showProgressBar('判定処理', '初期化中...');
        
        // オプション取得
        const options = getExecutionOptions();
        
        // ワークフロー実行
        const controller = new WorkflowController();
        const result = controller.execute(options);
        
        // 完了処理
        showCompletionDialog(result);
        
      } catch (error) {
        handleError(error);
      }
    }

[ ] // 実行オプション取得
    function getExecutionOptions() {
      // 設定シートから読み込み
      // デフォルト値適用
      return {
        batchSize: 20,
        skipOCR: false,
        testMode: false
      };
    }

[ ] // ワークフローコントローラー
    class WorkflowController {
      execute(options) {
        // 状態管理初期化
        // ステージ実行
        // 結果返却
      }
    }

[ ] // エラーハンドリング
    function handleError(error) {
      // エラーログ記録
      // ユーザー通知
      // リカバリ提案
    }
```

### 4.2 StateManager.gs（状態管理）
```javascript
[ ] // 状態保存
    function saveState(state) {
      // PropertiesServiceに保存
      // バックアップも作成
    }

[ ] // 状態読み込み
    function loadState() {
      // 保存された状態を復元
      // 整合性チェック
    }

[ ] // 状態クリア
    function clearState() {
      // 完了後のクリーンアップ
      // ログは保持
    }

[ ] // 進捗追跡
    function trackProgress(stage, processed, total) {
      // 進捗率計算
      // UI更新
      // ログ記録
    }

[ ] // チェックポイント作成
    function createCheckpoint(stage, data) {
      // 現在の状態を保存
      // 再開用情報記録
    }
```

### 4.3 TimeoutHandler.gs（タイムアウト処理）
```javascript
[ ] // タイムアウトトリガー設定
    function setTimeoutTrigger() {
      // 既存トリガー削除
      // 新規トリガー作成（7分後）
      // トリガーID保存
    }

[ ] // タイムアウト処理
    function handleTimeout() {
      // 現在の状態保存
      // 処理中断
      // 再開トリガー設定
    }

[ ] // 再開処理
    function resumeFromCheckpoint(e) {
      // トリガーイベント処理
      // 状態復元
      // 処理継続
    }

[ ] // トリガー削除
    function deleteTrigger(triggerId) {
      // 指定トリガー削除
      // クリーンアップ
    }

[ ] // タイムアウトチェック
    function checkExecutionTime(startTime) {
      // 経過時間計算
      // 閾値（5分30秒）チェック
    }
```

---

## 5. 処理ステージ実装

### 5.1 転記処理
```javascript
[ ] // 転記メイン処理
    function processTransfer(state) {
      // 判定前公告読み込み
      // データ変換
      // 入札公告マスター書き込み
      // 進捗更新
    }

[ ] // データマッピング
    function mapPreToMaster(preAnnouncement) {
      return {
        案件ID: generateAnnouncementId(),
        案件名: preAnnouncement.案件名,
        公告URL: preAnnouncement.URL,
        // ... その他フィールド
      };
    }

[ ] // ID生成
    function generateAnnouncementId() {
      // ユニークID生成
      // 形式: ANN-YYYYMMDD-XXXX
    }

[ ] // バッチ転記
    function transferBatch(batch) {
      // トランザクション的処理
      // エラー時ロールバック
    }
```

### 5.2 OCR処理連携
```javascript
[ ] // OCR処理メイン
    function processOCR(state) {
      // 未処理公告の取得
      // OCR送信
      // 結果待機
      // 結果保存
    }

[ ] // OCR送信
    function sendToOCRService(announcement) {
      // サーバーライブラリ呼び出し
      const result = LibraryName.sendToOCR(
        announcement.公告URL
      );
      return result;
    }

[ ] // OCR結果処理
    function processOCRResult(ocrResult, announcement) {
      // テキスト抽出
      // 構造化データ作成
      // 公告要件マスター更新
    }

[ ] // OCRエラー処理
    function handleOCRError(error, announcement) {
      // エラーログ
      // スキップ or リトライ
      // ユーザー通知
    }
```

### 5.3 要件分類処理
```javascript
[ ] // 要件分類メイン
    function processClassification(state) {
      // OCR結果取得
      // LLM分類実行
      // 分類結果保存
    }

[ ] // LLM要件抽出
    function extractRequirements(ocrText) {
      // サーバーライブラリ呼び出し
      const requirements = LibraryName.extractRequirements(
        ocrText
      );
      return requirements;
    }

[ ] // 要件分類
    function classifyRequirements(requirements) {
      // 6カテゴリへの分類
      // 信頼度チェック
      // 手動確認フラグ
    }

[ ] // 分類結果保存
    function saveClassificationResults(classified) {
      // 公告要件マスター更新
      // 分類ログ記録
    }
```

### 5.4 判定処理実行
```javascript
[ ] // 判定処理メイン
    function processJudgment(state) {
      // 公告×企業の組み合わせ作成
      // 欠格要件一括判定
      // その他要件個別判定
      // 結果保存
    }

[ ] // 欠格要件一括判定
    function batchDisqualificationCheck(companies) {
      // サーバーライブラリ呼び出し
      const results = LibraryName.judgeDisqualificationBatch(
        companies
      );
      return results;
    }

[ ] // 個別要件判定
    function judgeIndividualRequirements(announcement, company) {
      const context = createJudgmentContext(announcement, company);
      
      // 各要件判定
      const grade = LibraryName.judgeGrade(context);
      const region = LibraryName.judgeRegion(context);
      const achievement = LibraryName.judgeAchievement(context);
      const engineer = LibraryName.judgeEngineer(context);
      
      return aggregateResults({grade, region, achievement, engineer});
    }

[ ] // 判定結果集約
    function aggregateJudgmentResults(results) {
      // 総合判定
      // 不足要件抽出
      // 改善提案生成
    }
```

---

## 6. レポート生成

### 6.1 ReportGenerator.gs
```javascript
[ ] // レポート生成メイン
    function generateReport() {
      // 判定結果集計
      // レポートシート作成
      // グラフ生成
      // フォーマット適用
    }

[ ] // サマリーシート作成
    function createSummarySheet(results) {
      // 全体統計
      // 企業別サマリー
      // 要件別分析
    }

[ ] // 詳細シート作成
    function createDetailSheet(results) {
      // 公告別詳細
      // 不足要件一覧
      // 改善提案
    }

[ ] // グラフ生成
    function createCharts(summaryData) {
      // 適格率グラフ
      // 要件別充足率
      // 企業別スコア
    }

[ ] // 条件付き書式
    function applyConditionalFormatting(sheet) {
      // 適格：緑
      // 不適格：赤
      // 要確認：黄
    }

[ ] // PDF出力
    function exportToPDF() {
      // レポートシート選択
      // PDF変換
      // ダウンロードリンク提供
    }
```

---

## 7. エラー処理とログ

### 7.1 エラーハンドリング
```javascript
[ ] // グローバルエラーハンドラー
    function handleGlobalError(error) {
      // エラー分類
      // ログ記録
      // ユーザー通知
      // リカバリ処理
    }

[ ] // 処理別エラーハンドリング
    function handleTransferError(error) { }
    function handleOCRError(error) { }
    function handleJudgmentError(error) { }

[ ] // ユーザーフレンドリーメッセージ
    function getUserMessage(error) {
      // 技術的詳細を隠蔽
      // 対処法を提示
    }

[ ] // エラーリカバリ
    function attemptRecovery(error, context) {
      // 自動リトライ
      // 部分的スキップ
      // 手動介入要求
    }
```

### 7.2 ログ管理
```javascript
[ ] // ログ記録
    function logToSheet(message, level, details) {
      // ログシート取得
      // タイムスタンプ付与
      // 構造化ログ記録
    }

[ ] // ログビューア
    function showLogViewer() {
      // ログフィルタリング
      // 検索機能
      // エクスポート
    }

[ ] // ログローテーション
    function rotateLogs() {
      // 古いログのアーカイブ
      // シートサイズ管理
    }

[ ] // パフォーマンスログ
    function logPerformance(operation, duration) {
      // 処理時間記録
      // ボトルネック分析
    }
```

---

## 8. 設定管理

### 8.1 設定シート管理
```javascript
[ ] // 設定読み込み
    function loadSettings() {
      // 設定シートから読み込み
      // デフォルト値マージ
      // 検証
    }

[ ] // 設定保存
    function saveSettings(settings) {
      // 検証
      // シート更新
      // キャッシュクリア
    }

[ ] // 設定項目定義
    const SETTINGS_SCHEMA = {
      batchSize: { type: 'number', min: 5, max: 50, default: 20 },
      ocrTimeout: { type: 'number', min: 30, max: 300, default: 60 },
      // ...
    };

[ ] // 設定検証
    function validateSettings(settings) {
      // スキーマ検証
      // 依存関係チェック
    }
```

---

## 9. テスト実装

### 9.1 単体テスト
```javascript
[ ] // DataValidator テスト
    function testDataValidator() {
      // 正常系
      // 異常系
      // 境界値
    }

[ ] // StateManager テスト
    function testStateManager() {
      // 保存/読み込み
      // 並行性
      // エラー処理
    }

[ ] // 各処理ステージテスト
    function testTransferProcess() { }
    function testOCRProcess() { }
    function testJudgmentProcess() { }
```

### 9.2 統合テスト
```javascript
[ ] // E2Eワークフローテスト
    function testCompleteWorkflow() {
      // テストデータ準備
      // 全ステージ実行
      // 結果検証
    }

[ ] // タイムアウトテスト
    function testTimeoutRecovery() {
      // タイムアウトシミュレート
      // 再開処理確認
    }

[ ] // 大量データテスト
    function testLargeDataset() {
      // 200件処理
      // パフォーマンス測定
    }
```

---

## 10. ユーザビリティ向上

### 10.1 操作ガイド
[ ] ツールチップの実装
[ ] コンテキストヘルプ
[ ] 操作動画の作成
[ ] FAQの整備

### 10.2 パフォーマンス改善
[ ] 画面更新の最適化
[ ] 非同期処理の活用
[ ] キャッシュ戦略
[ ] 不要な再計算の削減

### 10.3 アクセシビリティ
[ ] キーボードショートカット
[ ] スクリーンリーダー対応
[ ] 色覚多様性への配慮
[ ] フォントサイズ調整

---

## 完了基準
- [ ] 全機能の実装完了
- [ ] 単体テストのパス
- [ ] 統合テストのパス
- [ ] ユーザビリティテスト実施
- [ ] ドキュメント完成

---

## 改訂履歴
- 2024-01-XX: 初版作成