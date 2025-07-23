# 入札公告要件判定システム - 開発TODOリスト

## 関連ドキュメント

### 設計書
- **全体設計**: `/docs/design/Project_design.md`
- **ワークフロー設計**: `/docs/design/Workflow_design.md`
- **判定エンジン設計**: `/docs/design/Judgment_design.md`
- **データアクセス設計**: `/docs/design/DataAccess_design.md`【要作成】
- **UI設計**: `/docs/design/UI_design.md`【要作成】
- **API設計**: `/docs/design/API_design.md`【要作成】
- **セキュリティ設計**: `/docs/design/Security_design.md`【要作成】

### 要件定義
- **要件定義書**: `/docs/requirements/要件定義書.md`
- **マスター項目一覧**: `/docs/requirements/appendix/別紙1：マスター項目一覧.md`
- **判定詳細**: `/docs/requirements/appendix/別紙2：入札公告要件判定詳細.md`

### TODO管理
- **サーバー側TODO**: `/docs/todo/Server_TODO.md`
- **クライアント側TODO**: `/docs/todo/Client_TODO.md`
- **マスター管理TODO**: `/docs/todo/Master_TODO.md`【要作成】
- **統合TODO**: `/docs/todo/Integration_TODO.md`【要作成】

### テスト仕様
- **テスト仕様書**: `/docs/test/Project_Test.md`
- **テストケース表**: `/docs/test/Project_TestCase.md`
- **テスト実装TODO**: `/docs/test/Project_Test_TODO.md`

---

## 開発フェーズ概要

### Phase 0: ドキュメント整備【現在進行中】 ✅
1. 未作成設計書の作成
2. TODO詳細化

### Phase 1: 基盤構築（サーバー側ライブラリ）【次フェーズ】
1. プロジェクト初期設定
2. 共通モジュール開発
3. データアクセス層実装
4. 判定エンジン実装

### Phase 2: クライアント側実装
1. UI/メニュー構築
2. ワークフロー制御
3. データ管理機能

### Phase 3: マスター管理機能
1. 管理者向け機能
2. データメンテナンス

### Phase 4: 統合・テスト
1. 結合テスト
2. システムテスト
3. 受入テスト

### Phase 5: デプロイ・運用準備
1. 本番環境構築
2. 運用ドキュメント作成

---

## Phase 0: ドキュメント整備【優先度: 🔴】

### 完了項目 ✅
- [x] 要件定義書の整合性確認と更新
- [x] TypeScript開発フローのドキュメント化
- [x] CI/CDパイプライン設計の追加
- [x] プロジェクトディレクトリ構造の詳細化
- [x] 不要なドキュメントの統合・削除

### 0.1 設計書作成
- [x] DataAccess_design.md の作成 ✅
  - [x] マスターデータアクセスパターン
  - [x] キャッシュ戦略
  - [x] エラーハンドリング
- [x] UI_design.md の作成 ✅
  - [x] 画面レイアウト
  - [x] カスタムメニュー構成
  - [x] ダイアログデザイン
- [x] API_design.md の作成 ✅
  - [x] OCR API仕様
  - [x] LLM API仕様
  - [x] ライブラリ公開API仕様
- [x] Security_design.md の作成 ✅
  - [x] 認証・認可設計
  - [x] データ保護方針
  - [x] 監査ログ設計

### 0.2 TODO詳細化
- [x] Master_TODO.md の作成 ✅
  - [x] マスター管理機能一覧
  - [x] データ検証ルール
- [x] Integration_TODO.md の作成 ✅
  - [x] デプロイ手順
  - [x] 環境設定

---

## Phase 1: 基盤構築（サーバー側ライブラリ）

### 1.1 プロジェクト初期設定
- [ ] TypeScript開発環境の構築
  - [ ] package.jsonの作成
  - [ ] tsconfig.jsonの設定（target: ES5, module: none）
  - [ ] @types/google-apps-scriptのインストール
  - [ ] ビルドスクリプトの作成（npm scripts）
    - [ ] build: TypeScript → JavaScript変換
    - [ ] push: claspでGASへアップロード
    - [ ] deploy: ビルド + プッシュ
- [ ] サーバー側GASプロジェクトの環境設定
  - [ ] claspプロジェクトの作成
  - [ ] ライブラリID の生成
  - [ ] バージョン管理の設定
  - [ ] アクセス権限の設定
- [ ] マスタースプレッドシートとの接続確認
  - [ ] スプレッドシートIDの設定
  - [ ] 読み取り権限の確認
  - [ ] テスト接続の実施
- [ ] CI/CD環境の構築（詳細は `/docs/todo/Integration_TODO.md` の「2. CI/CDパイプライン構築」参照）

### 1.2 共通モジュール開発

詳細な実装タスクは `/docs/todo/Server_TODO.md` の「2. 共通モジュール開発」セクションを参照してください。

主要モジュール:
- [ ] Constants.ts（定数定義）
- [ ] Utils.ts（ユーティリティ）  
- [ ] Logger.ts（ログ管理）

### 1.3 データアクセス層

詳細な実装タスクは `/docs/todo/Server_TODO.md` の「3. データアクセス層」セクションを参照してください。

主要コンポーネント:
- [ ] MasterDataAccess.ts（マスターデータアクセス）
- [ ] Repository実装（企業、拠点、従業員）
- [ ] キャッシュ機能

### 1.4 判定エンジン

詳細な実装タスクは `/docs/todo/Server_TODO.md` の「4. 判定エンジン実装」セクションを参照してください。

主要モジュール:
- [ ] JudgmentEngine.ts（メインエンジン）
- [ ] DisqualificationJudge.ts（欠格要件判定）
- [ ] GradeJudge.ts（等級要件判定）
- [ ] RegionJudge.ts（地域要件判定）
- [ ] AchievementJudge.ts（実績要件判定）
- [ ] EngineerJudge.ts（技術者要件判定）

### 1.5 外部連携

詳細な実装タスクは `/docs/todo/Server_TODO.md` の「5. 外部連携」セクションを参照してください。

主要モジュール:
- [ ] OCRConnector.ts（OCR API連携）
- [ ] LLMConnector.ts（LLM API連携）

---

## Phase 2: クライアント側実装

### 2.1 UI実装

詳細な実装タスクは `/docs/todo/Client_TODO.md` の「2. UI実装」セクションを参照してください。

主要モジュール:
- [ ] UIManager.ts（メニュー管理）
- [ ] 進捗表示UI
- [ ] HTML/CSSテンプレート

### 2.2 データ管理

詳細な実装タスクは `/docs/todo/Client_TODO.md` の「3. データ管理」セクションを参照してください。

主要モジュール:
- [ ] DataManager.ts（データ読み書き）
- [ ] DataValidator.ts（入力検証）

### 2.3 ワークフロー制御

詳細な実装タスクは `/docs/todo/Client_TODO.md` の「4. ワークフロー制御」セクションを参照してください。

主要モジュール:
- [ ] MainController.ts（メイン制御）
- [ ] StateManager.ts（状態管理）
- [ ] TimeoutHandler.ts（タイムアウト処理）

### 2.4 レポート生成

詳細な実装タスクは `/docs/todo/Client_TODO.md` の「6. レポート生成」セクションを参照してください。

主要モジュール:
- [ ] ReportGenerator.ts（レポート生成）

---

## Phase 3: マスター管理機能

詳細な実装タスクは `/docs/todo/Master_TODO.md` を参照してください。

### 3.1 管理者向け機能
- [ ] 管理者メニュー構築
- [ ] データメンテナンス機能

### 3.2 データ整合性管理
- [ ] 整合性チェック機能
- [ ] バックアップ/リストア

---

## Phase 4: 統合・テスト

### 4.1 統合作業
- [ ] ライブラリ公開設定
  - [ ] バージョン作成
  - [ ] アクセス権限設定
  - [ ] ドキュメント作成
- [ ] クライアント側統合
  - [ ] ライブラリ参照設定
  - [ ] 関数呼び出しテスト
  - [ ] エラーハンドリング確認

### 4.2 単体テスト
- [ ] サーバー側テスト（詳細は `/docs/test/Project_Test_TODO.md` 参照）
- [ ] クライアント側テスト（詳細は `/docs/test/Project_Test_TODO.md` 参照）

### 4.3 結合テスト
- [ ] E2Eテスト（詳細は `/docs/test/Project_TestCase.md` 参照）
- [ ] パフォーマンステスト

### 4.4 受入テスト
- [ ] ユーザー受入テスト

---

## Phase 5: デプロイ・運用準備

詳細な実装タスクは `/docs/todo/Integration_TODO.md` を参照してください。

### 5.1 デプロイ準備
- [ ] 本番環境設定
- [ ] データ移行

### 5.2 運用ドキュメント整備
- [ ] Operation_Manual.md の作成
- [ ] Admin_Guide.md の作成
- [ ] User_Guide.md の作成
- [ ] Troubleshooting.md の作成

### 5.3 運用体制構築
- [ ] 監視設定
  - [ ] エラー通知設定
  - [ ] パフォーマンス監視
  - [ ] ログ監視
- [ ] サポート体制
  - [ ] 問い合わせ窓口設定
  - [ ] エスカレーション手順

### 5.4 リリース作業
- [ ] 本番リリース
  - [ ] 最終動作確認
  - [ ] リリース実施
  - [ ] リリース後確認
- [ ] フォローアップ
  - [ ] 初期運用サポート
  - [ ] フィードバック収集
  - [ ] 改善計画策定

---

## 開発上の注意事項

### コーディング規約
1. 各モジュールは300行以内に収める
2. SOLID原則を遵守する
3. TypeScriptで開発し、名前空間パターンを使用（import/export文は使用しない）
4. エラーハンドリングを確実に実装する
5. ログを適切に記録する
6. テストを並行して実施する
7. ドキュメントは実装と並行して作成する

### GAS固有の制約
1. 実行時間制限: 6分（5分でタイムアウト処理実装）
2. トリガー設定: 7分後（安全マージン考慮）
3. スプレッドシートセル数: 1000万セル制限
4. API呼び出し回数制限への配慮

---

## 完了基準
- [ ] 全設計書の作成完了
- [ ] 全機能の実装完了
- [ ] 全テストケースの合格
- [ ] 運用ドキュメントの完成
- [ ] 本番環境での動作確認
- [ ] ユーザー教育の完了

---

## 優先度凡例
- 🔴 Critical: システムの基盤となる重要タスク
- 🟡 High: 主要機能の実装に必要なタスク
- 🟢 Medium: 補助機能や拡張機能のタスク
- ⚪ Low: 改善や最適化のタスク

## 進捗管理
- ✅ 完了項目はチェックボックスをマーク
- 各フェーズの開始時に詳細タスクを再確認
- 週次で進捗レビューを実施

---

## 改訂履歴
- 2025-07-23: TypeScript対応完了、CI/CD設計追加、Phase 0の進捗更新
- 2025-07-23: 欠格要件一括処理、中間テーブル最適化タスク追加
- 2025-07-22: 初版作成