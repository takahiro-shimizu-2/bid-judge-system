# 入札公告要件判定システム - UI設計書

## 関連ドキュメント
- **全体設計書**: `/docs/design/Project_design.md`
- **ワークフロー設計**: `/docs/design/Workflow_design.md`
- **要件定義書**: `/docs/requirements/要件定義書.md`
- **クライアント側TODO**: `/docs/todo/Client_TODO.md`

---

## 1. 概要

### 1.1 目的
Google Sheetsの標準UIを活用しながら、カスタムメニューとダイアログを通じて直感的で効率的な操作体験を提供する。

### 1.2 設計方針
- **シンプルさ**: Google Sheetsの使い慣れたUIを最大限活用
- **視認性**: 色分けやアイコンによる状態の可視化
- **レスポンシブ**: 処理状況のリアルタイム表示
- **エラーフレンドリー**: 分かりやすいエラーメッセージとガイダンス

---

## 2. 画面構成

### 2.1 全体レイアウト

#### 2.1.1 スプレッドシート構成
```
入札判定システム_利用者
├─ 📋 判定前公告一覧（メイン画面）
├─ 📊 入札公告マスター
├─ 📑 公告要件マスター
├─ ✅ 企業公告判定マスター
├─ ✔️ 充足要件リスト
├─ ❌ 不足要件リスト
├─ 🔄 要件拠点中間テーブル
├─ 📈 報告用シート
├─ 📝 LOG
└─ ⚙️ 設定
```

#### 2.1.2 メイン画面（判定前公告一覧）
| 列 | 項目名 | 説明 | 表示形式 |
|----|--------|------|----------|
| A | ステータス | 処理状態アイコン | 🔵未処理 🟡処理中 🟢完了 🔴エラー |
| B | 選択 | チェックボックス | ☐ |
| C | 公告ID | 自動採番 | ANN-20250723-0001 |
| D | 案件名 | 入力必須 | テキスト |
| E | 発注機関 | プルダウン選択 | ドロップダウン |
| F | 公告URL | URL入力 | ハイパーリンク |
| G | 工事種別 | プルダウン選択 | ドロップダウン |
| H | 工事場所 | テキスト入力 | テキスト |
| I | 開札日 | 日付選択 | 日付形式 |
| J | 処理日時 | 自動記録 | 日時形式 |
| K | エラー内容 | エラー時のみ | テキスト（赤字） |

### 2.2 カスタムメニュー構成

```javascript
入札公告判定 ▼
├─ 📋 判定処理
│   ├─ ▶️ 選択行を判定実行
│   ├─ ⏩ 未処理を一括判定
│   └─ 🔄 エラー行を再判定
├─ 📊 レポート
│   ├─ 📈 判定結果サマリー
│   ├─ 📑 詳細レポート生成
│   └─ 💾 レポートエクスポート
├─ 🔧 データ管理
│   ├─ 📥 CSVインポート
│   ├─ 🗑️ 処理済みデータクリア
│   └─ 🔄 マスター同期確認
├─ ⚙️ 設定
│   ├─ 🏢 企業情報設定
│   ├─ 🔑 API設定
│   └─ 📧 通知設定
└─ ❓ ヘルプ
    ├─ 📖 使い方ガイド
    ├─ 🐛 トラブルシューティング
    └─ ℹ️ バージョン情報
```

---

## 3. ダイアログデザイン

### 3.1 プログレスダイアログ

#### 3.1.1 基本レイアウト
```html
<div class="progress-dialog">
  <div class="header">
    <h2>🔄 判定処理実行中</h2>
    <span class="close-btn" onclick="cancelProcess()">✕</span>
  </div>
  
  <div class="content">
    <!-- 全体進捗 -->
    <div class="overall-progress">
      <h3>全体進捗</h3>
      <div class="progress-bar">
        <div class="progress-fill" style="width: 45%">45%</div>
      </div>
      <p class="progress-text">100件中45件完了</p>
    </div>
    
    <!-- 現在の処理 -->
    <div class="current-task">
      <h3>現在の処理</h3>
      <div class="task-info">
        <span class="task-icon">🔍</span>
        <span class="task-name">OCR処理中: ○○建設工事.pdf</span>
      </div>
      <div class="sub-progress-bar">
        <div class="progress-fill" style="width: 75%"></div>
      </div>
    </div>
    
    <!-- 処理ログ -->
    <div class="process-log">
      <h3>処理ログ</h3>
      <div class="log-container">
        <div class="log-entry success">✅ 10:15:30 - 公告ID: ANN-001 判定完了</div>
        <div class="log-entry warning">⚠️ 10:15:25 - 公告ID: ANN-002 OCRリトライ中</div>
        <div class="log-entry info">ℹ️ 10:15:20 - バッチ処理開始（20件）</div>
      </div>
    </div>
    
    <!-- 推定時間 -->
    <div class="time-estimate">
      <p>⏱️ 推定残り時間: 約3分</p>
      <p>⚡ 処理速度: 15件/分</p>
    </div>
  </div>
  
  <div class="footer">
    <button class="btn-secondary" onclick="pauseProcess()">⏸️ 一時停止</button>
    <button class="btn-primary" onclick="hideDialog()">バックグラウンドで実行</button>
  </div>
</div>
```

#### 3.1.2 スタイル定義
```css
.progress-dialog {
  width: 500px;
  max-height: 600px;
  font-family: 'Noto Sans JP', sans-serif;
}

.header {
  background: #1a73e8;
  color: white;
  padding: 15px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.progress-bar {
  height: 24px;
  background: #e0e0e0;
  border-radius: 12px;
  overflow: hidden;
  margin: 10px 0;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #1a73e8 0%, #4285f4 100%);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  transition: width 0.3s ease;
}

.log-container {
  height: 150px;
  overflow-y: auto;
  border: 1px solid #ddd;
  padding: 10px;
  background: #f8f9fa;
  font-size: 12px;
}

.log-entry {
  margin: 5px 0;
  padding: 5px;
  border-radius: 4px;
}

.log-entry.success { background: #e6f4ea; color: #1e8e3e; }
.log-entry.warning { background: #fef7e0; color: #f9ab00; }
.log-entry.error { background: #fce8e6; color: #d33b2c; }
.log-entry.info { background: #e8f0fe; color: #1a73e8; }
```

### 3.2 設定ダイアログ

#### 3.2.1 タブ構成
```html
<div class="settings-dialog">
  <div class="tabs">
    <div class="tab active" onclick="showTab('general')">⚙️ 基本設定</div>
    <div class="tab" onclick="showTab('company')">🏢 企業情報</div>
    <div class="tab" onclick="showTab('api')">🔑 API設定</div>
    <div class="tab" onclick="showTab('notification')">📧 通知設定</div>
  </div>
  
  <div class="tab-content" id="general-tab">
    <h3>基本設定</h3>
    
    <div class="form-group">
      <label>バッチサイズ</label>
      <input type="number" id="batchSize" min="5" max="50" value="20">
      <span class="help-text">一度に処理する公告数（5-50）</span>
    </div>
    
    <div class="form-group">
      <label>タイムアウト設定</label>
      <select id="timeoutSetting">
        <option value="300">5分（推奨）</option>
        <option value="240">4分（安全）</option>
        <option value="360">6分（最大）</option>
      </select>
    </div>
    
    <div class="form-group">
      <label>
        <input type="checkbox" id="autoRetry" checked>
        エラー時の自動リトライ
      </label>
    </div>
  </div>
</div>
```

### 3.3 エラーダイアログ

#### 3.3.1 エラー表示
```html
<div class="error-dialog">
  <div class="error-header">
    <span class="error-icon">⚠️</span>
    <h2>処理中にエラーが発生しました</h2>
  </div>
  
  <div class="error-content">
    <div class="error-summary">
      <h3>エラー概要</h3>
      <p class="error-message">OCR処理がタイムアウトしました</p>
    </div>
    
    <div class="error-details">
      <h3>詳細情報</h3>
      <div class="detail-box">
        <p><strong>エラーコード:</strong> OCR_TIMEOUT_001</p>
        <p><strong>発生箇所:</strong> 公告ID: ANN-20250723-0015</p>
        <p><strong>発生時刻:</strong> 2025/07/23 10:45:30</p>
      </div>
    </div>
    
    <div class="error-solution">
      <h3>解決方法</h3>
      <ul>
        <li>✓ ネットワーク接続を確認してください</li>
        <li>✓ PDFファイルのサイズが10MB以下であることを確認してください</li>
        <li>✓ 時間をおいて再度実行してください</li>
      </ul>
    </div>
  </div>
  
  <div class="error-footer">
    <button class="btn-secondary" onclick="copyErrorInfo()">📋 エラー情報をコピー</button>
    <button class="btn-primary" onclick="retryProcess()">🔄 再試行</button>
    <button class="btn-default" onclick="closeDialog()">閉じる</button>
  </div>
</div>
```

---

## 4. 条件付き書式

### 4.1 ステータス表示
```javascript
// ステータス列の条件付き書式
const statusRules = [
  {
    condition: 'TEXT_CONTAINS',
    value: '未処理',
    format: { backgroundColor: '#e3f2fd', fontColor: '#1976d2' }
  },
  {
    condition: 'TEXT_CONTAINS',
    value: '処理中',
    format: { backgroundColor: '#fff3e0', fontColor: '#f57c00' }
  },
  {
    condition: 'TEXT_CONTAINS',
    value: '完了',
    format: { backgroundColor: '#e8f5e9', fontColor: '#388e3c' }
  },
  {
    condition: 'TEXT_CONTAINS',
    value: 'エラー',
    format: { backgroundColor: '#ffebee', fontColor: '#d32f2f', fontWeight: 'bold' }
  }
];
```

### 4.2 判定結果の色分け
```javascript
// 判定結果シートの条件付き書式
const resultRules = [
  {
    range: '判定結果列',
    conditions: [
      { value: '適格', color: '#4caf50' },      // 緑
      { value: '不適格', color: '#f44336' },    // 赤
      { value: '要確認', color: '#ff9800' },    // オレンジ
      { value: '判定中', color: '#2196f3' }     // 青
    ]
  }
];
```

---

## 5. インタラクション設計

### 5.1 ユーザーフロー

#### 5.1.1 基本的な判定フロー
```
1. 判定前公告一覧シートを開く
   ↓
2. 公告情報を入力またはインポート
   ↓
3. 判定したい行を選択（チェックボックス）
   ↓
4. メニューから「選択行を判定実行」をクリック
   ↓
5. プログレスダイアログが表示される
   ↓
6. 処理完了後、結果確認ダイアログが表示
   ↓
7. レポートシートで詳細を確認
```

#### 5.1.2 エラー時のフロー
```
エラー発生
   ↓
エラーダイアログ表示
   ↓
├─ 再試行を選択 → 処理を再開
├─ スキップを選択 → 次の公告へ
└─ 中止を選択 → 処理を終了
```

### 5.2 キーボードショートカット
| ショートカット | 機能 |
|----------------|------|
| Ctrl + Shift + R | 選択行を判定実行 |
| Ctrl + Shift + A | 全選択/全解除 |
| Ctrl + Shift + S | 設定ダイアログを開く |
| Ctrl + Shift + H | ヘルプを表示 |

---

## 6. レスポンシブデザイン

### 6.1 画面サイズ対応
- **デスクトップ**: フル機能表示
- **タブレット**: 主要機能のみ、簡略表示
- **モバイル**: 閲覧のみ（編集・実行は非推奨）

### 6.2 ダイアログのレスポンシブ対応
```css
@media (max-width: 768px) {
  .dialog {
    width: 90%;
    max-width: none;
    margin: 10px;
  }
  
  .tabs {
    display: block;
  }
  
  .tab {
    display: block;
    width: 100%;
    margin: 2px 0;
  }
}
```

---

## 7. アクセシビリティ

### 7.1 色覚多様性対応
- 色だけでなくアイコンやテキストでも状態を表現
- コントラスト比 WCAG AA準拠（4.5:1以上）
- カラーブラインドモード対応

### 7.2 スクリーンリーダー対応
- 適切なARIAラベルの設定
- フォーカス順序の最適化
- 音声読み上げ用の代替テキスト

---

## 8. パフォーマンス考慮

### 8.1 初期表示の高速化
- 必要最小限のデータのみ初期ロード
- 遅延読み込みの活用
- プログレッシブレンダリング

### 8.2 操作レスポンス
- デバウンス処理（連続クリック防止）
- オプティミスティックUI（楽観的更新）
- ローディング表示の即座表示

---

## 9. 実装ガイドライン

### 9.1 HTMLテンプレート管理
```javascript
// HtmlService を使用したテンプレート管理
function include(filename) {
  return HtmlService.createHtmlOutputFromFile(filename).getContent();
}

function openProgressDialog() {
  const html = HtmlService.createTemplateFromFile('dialogs/progress')
    .evaluate()
    .setWidth(500)
    .setHeight(600);
    
  SpreadsheetApp.getUi().showModalDialog(html, '処理実行中');
}
```

### 9.2 スタイルガイド
- Material Design原則に準拠
- Google Workspace標準UIとの一貫性
- カスタムカラーパレットの定義

---

## 10. テスト項目

### 10.1 UIテスト
- [ ] 全ダイアログの表示確認
- [ ] ボタン・リンクの動作確認
- [ ] エラー表示の確認
- [ ] プログレス表示の確認

### 10.2 ユーザビリティテスト
- [ ] 5クリック以内で主要タスク完了
- [ ] エラーメッセージの理解度
- [ ] 処理時間の体感速度

---

## 改訂履歴
- 2025-07-23: 初版作成