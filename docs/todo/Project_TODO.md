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

### Phase 0: ドキュメント整備【優先】
1. 未作成設計書の作成
2. TODO詳細化

### Phase 1: 基盤構築（サーバー側ライブラリ）
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

### 0.1 設計書作成
- [ ] DataAccess_design.md の作成
  - [ ] マスターデータアクセスパターン
  - [ ] キャッシュ戦略
  - [ ] エラーハンドリング
- [ ] UI_design.md の作成
  - [ ] 画面レイアウト
  - [ ] カスタムメニュー構成
  - [ ] ダイアログデザイン
- [ ] API_design.md の作成
  - [ ] OCR API仕様
  - [ ] LLM API仕様
  - [ ] ライブラリ公開API仕様
- [ ] Security_design.md の作成
  - [ ] 認証・認可設計
  - [ ] データ保護方針
  - [ ] 監査ログ設計

### 0.2 TODO詳細化
- [ ] Master_TODO.md の作成
  - [ ] マスター管理機能一覧
  - [ ] データ検証ルール
- [ ] Integration_TODO.md の作成
  - [ ] デプロイ手順
  - [ ] 環境設定

---

## Phase 1: 基盤構築（サーバー側ライブラリ）

### 1.1 プロジェクト初期設定
- [ ] TypeScript開発環境の構築
  - [ ] tsconfig.jsonの設定（target: ES5, module: none）
  - [ ] @types/google-apps-scriptのインストール
  - [ ] ビルドスクリプトの作成
- [ ] サーバー側GASプロジェクトの環境設定
  - [ ] claspプロジェクトの作成
  - [ ] ライブラリID の生成
  - [ ] バージョン管理の設定
  - [ ] アクセス権限の設定
- [ ] マスタースプレッドシートとの接続確認
  - [ ] スプレッドシートIDの設定
  - [ ] 読み取り権限の確認
  - [ ] テスト接続の実施
- [ ] CI/CD環境の構築
  - [ ] GitHub Actionsの設定
  - [ ] clasp認証情報の設定（Secrets）
  - [ ] 自動ビルド・デプロイの確認

### 1.2 共通モジュール開発

#### 1.2.1 Constants.ts（定数定義）
- [ ] スプレッドシートID定義
  - [ ] マスターシートID
  - [ ] シート名定義
- [ ] カラム定義
  - [ ] 各マスターのカラムインデックス
  - [ ] データ型定義
- [ ] エラーメッセージ定義
  - [ ] エラーコード体系
  - [ ] メッセージテンプレート

#### 1.2.2 Utils.ts（ユーティリティ）
- [ ] 文字列処理関数
  - [ ] `convertToHalfWidth()`: 全角→半角変換
  - [ ] `trim()`: 空白除去
  - [ ] `removeControlChars()`: 制御文字除去
- [ ] 日付処理関数
  - [ ] `parseDate()`: 多様な形式の日付パース
  - [ ] `formatDate()`: 日付フォーマット
  - [ ] `convertWareki()`: 和暦変換
- [ ] 検証関数
  - [ ] `isValidUrl()`: URL検証
  - [ ] `isEmpty()`: 空値チェック

#### 1.2.3 Logger.ts（ログ管理）
- [ ] ログ出力関数
  - [ ] `log()`: 通常ログ
  - [ ] `logError()`: エラーログ
  - [ ] `logDebug()`: デバッグログ
- [ ] ログフォーマット定義
  - [ ] タイムスタンプ形式
  - [ ] ログレベル定義
- [ ] ログ保存先設定
  - [ ] ログシートへの書き込み
  - [ ] ローテーション設定

### 1.3 データアクセス層

#### 1.3.1 MasterDataAccess.ts
- [ ] 企業マスター関連
  - [ ] `getCompanyById()`: 企業情報取得
  - [ ] `getCompanyFlags()`: 欠格フラグ取得
  - [ ] `checkDisqualification()`: 欠格チェック
- [ ] 拠点マスター関連
  - [ ] `getOfficesByCompany()`: 企業の拠点一覧
  - [ ] `getOfficeLicenses()`: 拠点の許可情報
  - [ ] `getOfficeExperiences()`: 拠点の実績情報
- [ ] 従業員マスター関連
  - [ ] `getEmployeesByOffice()`: 拠点の従業員一覧
  - [ ] `getEmployeeQualifications()`: 従業員の資格情報
- [ ] キャッシュ機能
  - [ ] `cacheManager()`: キャッシュ管理
  - [ ] `invalidateCache()`: キャッシュクリア

### 1.4 判定エンジン

#### 1.4.1 JudgmentEngine.ts（メインエンジン）
- [ ] 判定制御
  - [ ] `executeJudgment()`: 判定実行メイン
  - [ ] `getJudgmentOrder()`: 判定順序制御
  - [ ] `aggregateResults()`: 結果集約
- [ ] 結果管理
  - [ ] `saveJudgmentResult()`: 判定結果保存
  - [ ] `updateEvaluationMaster()`: 評価マスター更新
- [ ] 中間テーブル管理
  - [ ] `createOptimizedIntermediateTable()`: 最適化された中間テーブル作成
  - [ ] `filterJudgedRequirements()`: 判定済み要件のフィルタリング

#### 1.4.2 DisqualificationJudge.ts（欠格要件判定）
- [ ] フラグチェック関数
  - [ ] `checkBankruptcy()`: 破産フラグ
  - [ ] `checkViolentRelation()`: 暴力団関係
  - [ ] `checkRehabilitation()`: 更生手続き
  - [ ] `checkSuspension()`: 指名停止
  - [ ] `checkInfoSecurity()`: 情報保全体制
  - [ ] `checkSocialInsurance()`: 社会保険滞納
  - [ ] `checkTerrorist()`: 破壊的団体
  - [ ] `checkForeign()`: 外国法的制約
  - [ ] `checkBOJ()`: 日銀取引停止
- [ ] 統合判定
  - [ ] `judgeDisqualification()`: 欠格要件統合判定
  - [ ] `createDisqualificationMessage()`: NGメッセージ生成
- [ ] 一括処理
  - [ ] `batchDisqualificationJudgment()`: 欠格要件一括判定
  - [ ] `prepareBatchData()`: 判定結果を配列に格納
  - [ ] `writeBatchResults()`: バッチ結果書き込み（100件単位）
  - [ ] `saveWriteProgress()`: 書き込み進捗保存
  - [ ] `resumeBatchWrite()`: 中断された書き込みの再開
  - [ ] `writeSufficiencyDetails()`: 充足要件リストへの一括登録
  - [ ] `writeShortageDetails()`: 不足要件リストへの一括登録
  - [ ] `markDisqualificationRequirementsAsJudged()`: 判定済みフラグ設定

#### 1.4.3 GradeJudge.ts（等級要件判定）
- [ ] 等級チェック
  - [ ] `checkGradeRequirement()`: 等級要件チェック
  - [ ] `compareGrades()`: 等級比較ロジック
  - [ ] `checkConstructionType()`: 工事種別チェック
- [ ] スコアチェック
  - [ ] `checkScoreRequirement()`: スコア要件チェック
  - [ ] `validateLicenseScore()`: ライセンススコア検証

#### 1.4.4 RegionJudge.ts（地域要件判定）
- [ ] 地域チェック
  - [ ] `checkRegionRequirement()`: 地域要件チェック
  - [ ] `isInTargetArea()`: 対象地域判定
  - [ ] `checkOfficeLocation()`: 拠点所在地確認

#### 1.4.5 AchievementJudge.ts（実績要件判定）
- [ ] 実績チェック
  - [ ] `checkAchievementRequirement()`: 実績要件チェック
  - [ ] `checkConstructionScore()`: 工事成績チェック
  - [ ] `checkConstructionScale()`: 工事規模チェック
  - [ ] `checkJVRatio()`: JV出資比率チェック
- [ ] 期間チェック
  - [ ] `isWithinPeriod()`: 対象期間内判定
  - [ ] `calculateAverageScore()`: 平均点計算

#### 1.4.6 EngineerJudge.ts（技術者要件判定）
- [ ] 資格チェック
  - [ ] `checkEngineerRequirement()`: 技術者要件チェック
  - [ ] `hasRequiredQualification()`: 必要資格保有確認
  - [ ] `checkEquivalentQualification()`: 同等資格判定
- [ ] 配置可能性
  - [ ] `checkAvailability()`: 技術者の配置可能性

### 1.5 外部連携

#### 1.5.1 OCRConnector.ts
- [ ] API連携
  - [ ] `sendToOCR()`: OCR処理要求
  - [ ] `checkOCRStatus()`: 処理状態確認
  - [ ] `getOCRResult()`: 結果取得
- [ ] 結果処理
  - [ ] `parseOCRResult()`: OCR結果パース
  - [ ] `extractRequirements()`: 要件抽出
  - [ ] `classifyRequirements()`: 要件分類
- [ ] エラーハンドリング
  - [ ] `handleOCRError()`: OCRエラー処理
  - [ ] `retryOCR()`: リトライ処理

---

## Phase 2: クライアント側実装

### 2.1 UI実装

#### 2.1.1 UIManager.ts
- [ ] メニュー作成
  - [ ] `onOpen()`: スプレッドシート開始時処理
  - [ ] `createCustomMenu()`: カスタムメニュー作成
  - [ ] `showSettingsDialog()`: 設定ダイアログ
- [ ] 進捗表示
  - [ ] `showProgressBar()`: プログレスバー表示
  - [ ] `updateProgress()`: 進捗更新
  - [ ] `showCompletionDialog()`: 完了ダイアログ

### 2.2 データ管理

#### 2.2.1 DataManager.ts
- [ ] データ読み込み
  - [ ] `loadPreAnnouncements()`: 判定前公告読み込み
  - [ ] `loadBidAnnouncements()`: 入札公告読み込み
- [ ] データ書き込み
  - [ ] `saveAnnouncement()`: 公告情報保存
  - [ ] `saveRequirements()`: 要件情報保存
  - [ ] `saveJudgmentResults()`: 判定結果保存
- [ ] データ変換
  - [ ] `convertToMasterFormat()`: マスター形式変換
  - [ ] `createReportData()`: レポートデータ作成

#### 2.2.2 DataValidator.ts
- [ ] 入力検証
  - [ ] `validateRequiredFields()`: 必須項目チェック
  - [ ] `validateDataFormat()`: データ形式チェック
  - [ ] `validateDateFields()`: 日付項目検証
- [ ] データクレンジング
  - [ ] `cleanInputData()`: 入力データクリーニング
  - [ ] `normalizeData()`: データ正規化

### 2.3 ワークフロー制御

#### 2.3.1 MainController.ts
- [ ] メイン処理
  - [ ] `executeWorkflow()`: ワークフロー実行
  - [ ] `initializeWorkflow()`: 初期化処理
  - [ ] `finalizeWorkflow()`: 終了処理
- [ ] バッチ処理
  - [ ] `processBatch()`: バッチ処理実行
  - [ ] `determineBatchSize()`: バッチサイズ決定
  - [ ] `splitIntoBatches()`: バッチ分割

#### 2.3.2 StateManager.ts
- [ ] 状態管理
  - [ ] `saveState()`: 処理状態保存
  - [ ] `loadState()`: 処理状態読み込み
  - [ ] `clearState()`: 状態クリア
- [ ] 進捗追跡
  - [ ] `trackProgress()`: 進捗追跡
  - [ ] `getProcessedCount()`: 処理済み件数取得
  - [ ] `getErrorCount()`: エラー件数取得

#### 2.3.3 TimeoutHandler.ts
- [ ] タイムアウト処理
  - [ ] `setTimeoutTrigger()`: トリガー設定（7分後）
  - [ ] `checkElapsedTime()`: 経過時間チェック（5分で中断）
  - [ ] `handleTimeout()`: タイムアウト処理
  - [ ] `deleteTrigger()`: トリガー削除
- [ ] 再開処理
  - [ ] `resumeFromCheckpoint()`: チェックポイントから再開
  - [ ] `validateCheckpoint()`: チェックポイント検証
  - [ ] `restoreWriteProgress()`: 書き込み進捗の復元

### 2.4 レポート生成

#### 2.4.1 ReportGenerator.ts
- [ ] レポート作成
  - [ ] `generateReport()`: レポート生成メイン
  - [ ] `createSummarySheet()`: サマリーシート作成
  - [ ] `createDetailSheet()`: 詳細シート作成
- [ ] フォーマット処理
  - [ ] `formatReportData()`: データフォーマット
  - [ ] `applyConditionalFormatting()`: 条件付き書式
  - [ ] `addCharts()`: グラフ追加

---

## Phase 3: マスター管理機能【詳細は Master_TODO.md 参照】

### 3.1 管理者向け機能
- [ ] 管理者メニュー構築
  - [ ] マスターデータ一覧表示
  - [ ] データインポート/エクスポート
  - [ ] データ検証機能
- [ ] データメンテナンス機能
  - [ ] マスターデータ編集画面
  - [ ] 一括更新機能
  - [ ] 履歴管理機能

### 3.2 データ整合性管理
- [ ] 整合性チェック機能
  - [ ] 参照整合性チェック
  - [ ] データ重複チェック
  - [ ] 必須項目チェック
- [ ] バックアップ/リストア
  - [ ] 定期バックアップ設定
  - [ ] リストア機能
  - [ ] バージョン管理

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

### 4.2 単体テスト【詳細は Unit_Test_Spec.md 参照】
- [ ] サーバー側テスト
  - [ ] 判定ロジックテスト
  - [ ] データアクセステスト
  - [ ] 外部連携テスト
- [ ] クライアント側テスト
  - [ ] UI動作テスト
  - [ ] データ検証テスト
  - [ ] ワークフローテスト

### 4.3 結合テスト【詳細は Integration_Test_Spec.md 参照】
- [ ] E2Eテスト
  - [ ] 正常系シナリオ
  - [ ] 異常系シナリオ
  - [ ] 大量データテスト
- [ ] パフォーマンステスト
  - [ ] 処理時間測定
  - [ ] メモリ使用量確認
  - [ ] 同時実行テスト

### 4.4 受入テスト【詳細は UAT_Test_Spec.md 参照】
- [ ] ユーザー受入テスト
  - [ ] 実データでのテスト
  - [ ] 操作性確認
  - [ ] 結果精度確認

---

## Phase 5: デプロイ・運用準備【詳細は Integration_TODO.md 参照】

### 5.1 デプロイ準備
- [ ] 本番環境設定
  - [ ] 本番用スプレッドシート作成
  - [ ] APIキー設定
  - [ ] アクセス権限設定
- [ ] データ移行
  - [ ] マスターデータ移行
  - [ ] テストデータクリア

### 5.2 運用ドキュメント整備
- [ ] Operation_Manual.md の作成
  - [ ] 日常運用手順
  - [ ] 定期メンテナンス手順
  - [ ] 障害対応手順
- [ ] Admin_Guide.md の作成
  - [ ] システム構成詳細
  - [ ] 管理者権限設定
  - [ ] マスターデータ管理
- [ ] User_Guide.md の作成
  - [ ] 基本操作手順
  - [ ] 画面説明
  - [ ] よくある質問
- [ ] Troubleshooting.md の作成
  - [ ] エラーメッセージ一覧
  - [ ] 問題解決手順
  - [ ] サポート連絡先

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

## 注意事項
1. 各モジュールは300行以内に収める
2. SOLID原則を遵守する
3. エラーハンドリングを確実に実装する
4. ログを適切に記録する
5. テストを並行して実施する
6. ドキュメントは実装と並行して作成する

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