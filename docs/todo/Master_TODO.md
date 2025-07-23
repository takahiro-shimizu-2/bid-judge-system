# 入札公告要件判定システム - マスター管理TODO

## 関連ドキュメント
- **全体TODO**: `/docs/todo/Project_TODO.md`
- **要件定義書**: `/docs/requirements/要件定義書.md`
- **マスター項目一覧**: `/docs/requirements/appendix/別紙1：マスター項目一覧.md`
- **セキュリティ設計**: `/docs/design/Security_design.md`

---

## 概要
マスターデータの管理機能に関する開発タスク一覧。管理者向けの機能開発とデータ整合性の維持に焦点を当てる。

---

## 優先度凡例
- 🔴 Critical: データ整合性に関わる重要タスク
- 🟡 High: 主要管理機能の実装
- 🟢 Medium: 利便性向上機能
- ⚪ Low: 最適化・改善タスク

---

## 1. 環境構築とアクセス管理 【優先度: 🔴】

### 1.1 マスタースプレッドシート設定
- [ ] マスタースプレッドシートの作成
  - [ ] 企業マスター（M_COMPANY）
  - [ ] 拠点マスター（M_OFFICE）
  - [ ] 従業員マスター（M_EMPLOYEE）
  - [ ] 拠点登録許可（T_OFFICE_LICENSE）
  - [ ] 拠点工事実績（T_OFFICE_EXPERIENCE）
  - [ ] 従業員資格（T_EMPLOYEE_QUAL）
  - [ ] 発注機関マスター（M_AGENCY）
  - [ ] 営業品目マスター（M_CONSTRUCTION_TYPE）
  - [ ] 技術者資格マスター（M_QUALIFICATION）

### 1.2 アクセス権限設定
- [ ] 読み取り専用権限の設定（ライブラリ用）
- [ ] 管理者編集権限の設定
- [ ] バックアップ用サービスアカウントの設定
- [ ] 監査ログ用の権限設定

### 1.3 初期データ投入
- [ ] サンプルデータの準備
- [ ] マスターデータのインポート機能
- [ ] データ検証スクリプトの実行
- [ ] 初期インデックスの構築

---

## 2. 管理者UI開発 【優先度: 🟡】

### 2.1 管理者メニュー（AdminMenu.ts）
```typescript
- [ ] カスタムメニューの実装
    function createAdminMenu() {
      const ui = SpreadsheetApp.getUi();
      ui.createMenu('マスター管理')
        .addItem('データインポート', 'showImportDialog')
        .addItem('データエクスポート', 'showExportDialog')
        .addItem('データ検証', 'runDataValidation')
        .addItem('バックアップ', 'createBackup')
        .addSeparator()
        .addItem('ユーザー管理', 'showUserManager')
        .addItem('ログ確認', 'showAuditLog')
        .addToUi();
    }

- [ ] 権限チェック機能
    function checkAdminAccess() {
      // 管理者権限の確認
      // 権限がない場合はエラー表示
    }
```

### 2.2 データ管理ダイアログ
- [ ] インポートダイアログの実装
  - [ ] CSVファイルアップロード
  - [ ] データマッピング設定
  - [ ] プレビュー機能
  - [ ] 検証結果表示
- [ ] エクスポートダイアログの実装
  - [ ] エクスポート範囲選択
  - [ ] ファイル形式選択（CSV/Excel）
  - [ ] データマスキング設定
- [ ] 一括編集ダイアログの実装
  - [ ] 条件指定
  - [ ] 更新内容入力
  - [ ] 影響範囲プレビュー

---

## 3. データ管理機能 【優先度: 🔴】

### 3.1 データインポート（DataImporter.ts）
```typescript
- [ ] CSVインポート機能
    function importFromCSV(fileBlob: Blob, targetSheet: string) {
      // CSVパース
      // データ検証
      // 重複チェック
      // トランザクション処理
      // インポート実行
    }

- [ ] データマッピング機能
    function mapCSVColumns(csvHeaders: string[], sheetHeaders: string[]) {
      // カラムマッピング
      // 必須項目チェック
      // データ型変換
    }

- [ ] バルクインポート最適化
    function bulkImport(data: any[][], sheet: Sheet) {
      // バッチ処理
      // プログレス表示
      // エラーハンドリング
    }
```

### 3.2 データエクスポート（DataExporter.ts）
```typescript
- [ ] セキュアエクスポート機能
    function exportData(sheet: string, options: ExportOptions) {
      // 権限チェック
      // データ取得
      // マスキング処理
      // ファイル生成
      // 監査ログ記録
    }

- [ ] フォーマット変換
    function convertToCSV(data: any[][]) {
      // CSV形式への変換
      // 文字エンコーディング処理
      // 特殊文字エスケープ
    }

- [ ] 大量データ対応
    function exportLargeDataset(query: Query) {
      // ページング処理
      // ストリーミング出力
      // メモリ管理
    }
```

---

## 4. データ検証機能 【優先度: 🔴】

### 4.1 整合性チェック（DataValidator.ts）
```typescript
- [ ] 参照整合性チェック
    function checkReferentialIntegrity() {
      // 企業→拠点の関係
      // 拠点→従業員の関係
      // 外部キー検証
    }

- [ ] データ品質チェック
    function checkDataQuality() {
      // 必須項目の欠損
      // データ型の不整合
      // 値の範囲チェック
      // 重複データ検出
    }

- [ ] ビジネスルール検証
    function validateBusinessRules() {
      // 欠格フラグの論理チェック
      // 日付の前後関係
      // ステータスの遷移
    }
```

### 4.2 検証レポート生成
```typescript
- [ ] 検証結果レポート
    function generateValidationReport(results: ValidationResult[]) {
      // エラー集計
      // 重要度別分類
      // 修正提案生成
      // レポートシート作成
    }

- [ ] 自動修正機能
    function autoFixIssues(issues: Issue[]) {
      // 修正可能な問題の特定
      // 修正実行
      // 修正ログ記録
    }
```

---

## 5. バックアップ・リストア機能 【優先度: 🟡】

### 5.1 バックアップ処理（BackupManager.ts）
```typescript
- [ ] 定期バックアップ
    function scheduleBackup() {
      // 日次バックアップトリガー設定
      // 世代管理（30世代）
      // 圧縮処理
    }

- [ ] 手動バックアップ
    function createManualBackup(description: string) {
      // 全マスターデータのコピー
      // メタデータ記録
      // バックアップ先フォルダへ移動
    }

- [ ] 差分バックアップ
    function createIncrementalBackup() {
      // 前回からの変更検出
      // 差分データのみ保存
      // 復元用インデックス作成
    }
```

### 5.2 リストア処理
```typescript
- [ ] ポイントインタイムリカバリ
    function restoreToPoint(timestamp: Date) {
      // バックアップ検索
      // データ復元
      // 整合性確認
    }

- [ ] 部分リストア
    function restorePartial(tables: string[], backupId: string) {
      // 特定テーブルのみ復元
      // 関連データの整合性維持
    }
```

---

## 6. データ同期機能 【優先度: 🟢】

### 6.1 外部システム連携
```typescript
- [ ] 外部データソース同期
    function syncExternalData(source: DataSource) {
      // API接続
      // データ取得
      // 差分検出
      // 更新処理
    }

- [ ] 変更通知
    function notifyDataChanges(changes: Change[]) {
      // Webhook送信
      // メール通知
      // ログ記録
    }
```

### 6.2 キャッシュ管理
```typescript
- [ ] キャッシュ無効化
    function invalidateCaches(tables: string[]) {
      // ライブラリ側キャッシュクリア
      // 通知送信
    }

- [ ] キャッシュ統計
    function getCacheStatistics() {
      // ヒット率計算
      // メモリ使用量
      // パフォーマンス分析
    }
```

---

## 7. ユーザー管理機能 【優先度: 🟡】

### 7.1 ユーザー管理（UserManager.ts）
```typescript
- [ ] ユーザー登録
    function registerUser(userData: User) {
      // メールアドレス検証
      // ロール割り当て
      // アクセス権限設定
    }

- [ ] ロール管理
    function manageRoles() {
      // ロール一覧表示
      // 権限マトリックス管理
      // ロール割り当て
    }

- [ ] アクセス履歴
    function getUserAccessHistory(userId: string) {
      // ログイン履歴
      // 操作履歴
      // 異常検知
    }
```

---

## 8. 監視・アラート機能 【優先度: 🟢】

### 8.1 データ監視
```typescript
- [ ] 異常検知
    function detectAnomalies() {
      // データ量の急激な変化
      // 不正なデータパターン
      // パフォーマンス劣化
    }

- [ ] アラート送信
    function sendAlert(alert: Alert) {
      // メール送信
      // Slack通知
      // ダッシュボード更新
    }
```

### 8.2 パフォーマンス監視
```typescript
- [ ] クエリ分析
    function analyzeQueries() {
      // 遅いクエリの検出
      // インデックス最適化提案
      // 実行計画分析
    }

- [ ] リソース監視
    function monitorResources() {
      // スプレッドシート容量
      // API使用量
      // 実行時間統計
    }
```

---

## 9. データ移行ツール 【優先度: 🟢】

### 9.1 既存データ移行
```typescript
- [ ] レガシーデータ変換
    function migrateFromLegacy(source: LegacyData) {
      // データ形式変換
      // 文字コード変換
      // データクレンジング
    }

- [ ] 移行検証
    function validateMigration(before: Dataset, after: Dataset) {
      // レコード数確認
      // データ内容比較
      // 整合性チェック
    }
```

---

## 10. ドキュメント・ヘルプ 【優先度: ⚪】

### 10.1 管理者向けドキュメント
- [ ] 管理者操作マニュアル作成
- [ ] データ定義書の自動生成
- [ ] API仕様書の作成
- [ ] トラブルシューティングガイド

### 10.2 ヘルプ機能
- [ ] コンテキストヘルプの実装
- [ ] 操作動画の作成
- [ ] FAQの整備

---

## テスト項目

### 単体テスト
- [ ] 各管理機能のテスト
- [ ] データ検証ロジックのテスト
- [ ] エラーハンドリングのテスト

### 統合テスト
- [ ] インポート→検証→エクスポートの一連フロー
- [ ] バックアップ→リストアの動作確認
- [ ] 大量データでの性能テスト

### セキュリティテスト
- [ ] アクセス権限のテスト
- [ ] データマスキングの確認
- [ ] 監査ログの動作確認

---

## 完了基準
- [ ] 全管理機能の実装完了
- [ ] データ検証の自動化
- [ ] バックアップの定期実行設定
- [ ] 管理者向けドキュメントの完成
- [ ] セキュリティ監査の合格

---

## 改訂履歴
- 2025-07-23: 初版作成