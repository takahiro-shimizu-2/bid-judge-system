# 入札公告要件判定システム - 開発TODOリスト

## 関連ドキュメント
- **設計書**: `/docs/design/Project_design.md`
- **要件定義書**: `/docs/requirements/要件定義書.md`
- **テスト仕様**: `/docs/test/Project_Test.md`
- **テストケース表**: `/docs/test/Project_TestCase.md`
- **テスト実装TODO**: `/docs/test/Project_Test_TODO.md`

---

## 開発フェーズ概要
1. **Phase 1**: 基盤構築（サーバー側ライブラリ）
2. **Phase 2**: クライアント側実装
3. **Phase 3**: 統合・テスト
4. **Phase 4**: デプロイ・運用準備

---

## Phase 1: 基盤構築（サーバー側ライブラリ）

### 1.1 プロジェクト初期設定
- [ ] サーバー側GASプロジェクトの環境設定
  - [ ] ライブラリID の生成
  - [ ] バージョン管理の設定
  - [ ] アクセス権限の設定
- [ ] マスタースプレッドシートとの接続確認
  - [ ] スプレッドシートIDの設定
  - [ ] 読み取り権限の確認
  - [ ] テスト接続の実施

### 1.2 共通モジュール開発

#### 1.2.1 Constants.gs（定数定義）
- [ ] スプレッドシートID定義
  - [ ] マスターシートID
  - [ ] シート名定義
- [ ] カラム定義
  - [ ] 各マスターのカラムインデックス
  - [ ] データ型定義
- [ ] エラーメッセージ定義
  - [ ] エラーコード体系
  - [ ] メッセージテンプレート

#### 1.2.2 Utils.gs（ユーティリティ）
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

#### 1.2.3 Logger.gs（ログ管理）
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

#### 1.3.1 MasterDataAccess.gs
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

#### 1.4.1 JudgmentEngine.gs（メインエンジン）
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

#### 1.4.2 DisqualificationJudge.gs（欠格要件判定）
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

#### 1.4.3 GradeJudge.gs（等級要件判定）
- [ ] 等級チェック
  - [ ] `checkGradeRequirement()`: 等級要件チェック
  - [ ] `compareGrades()`: 等級比較ロジック
  - [ ] `checkConstructionType()`: 工事種別チェック
- [ ] スコアチェック
  - [ ] `checkScoreRequirement()`: スコア要件チェック
  - [ ] `validateLicenseScore()`: ライセンススコア検証

#### 1.4.4 RegionJudge.gs（地域要件判定）
- [ ] 地域チェック
  - [ ] `checkRegionRequirement()`: 地域要件チェック
  - [ ] `isInTargetArea()`: 対象地域判定
  - [ ] `checkOfficeLocation()`: 拠点所在地確認

#### 1.4.5 AchievementJudge.gs（実績要件判定）
- [ ] 実績チェック
  - [ ] `checkAchievementRequirement()`: 実績要件チェック
  - [ ] `checkConstructionScore()`: 工事成績チェック
  - [ ] `checkConstructionScale()`: 工事規模チェック
  - [ ] `checkJVRatio()`: JV出資比率チェック
- [ ] 期間チェック
  - [ ] `isWithinPeriod()`: 対象期間内判定
  - [ ] `calculateAverageScore()`: 平均点計算

#### 1.4.6 EngineerJudge.gs（技術者要件判定）
- [ ] 資格チェック
  - [ ] `checkEngineerRequirement()`: 技術者要件チェック
  - [ ] `hasRequiredQualification()`: 必要資格保有確認
  - [ ] `checkEquivalentQualification()`: 同等資格判定
- [ ] 配置可能性
  - [ ] `checkAvailability()`: 技術者の配置可能性

### 1.5 外部連携

#### 1.5.1 OCRConnector.gs
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

#### 2.1.1 UIManager.gs
- [ ] メニュー作成
  - [ ] `onOpen()`: スプレッドシート開始時処理
  - [ ] `createCustomMenu()`: カスタムメニュー作成
  - [ ] `showSettingsDialog()`: 設定ダイアログ
- [ ] 進捗表示
  - [ ] `showProgressBar()`: プログレスバー表示
  - [ ] `updateProgress()`: 進捗更新
  - [ ] `showCompletionDialog()`: 完了ダイアログ

### 2.2 データ管理

#### 2.2.1 DataManager.gs
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

#### 2.2.2 DataValidator.gs
- [ ] 入力検証
  - [ ] `validateRequiredFields()`: 必須項目チェック
  - [ ] `validateDataFormat()`: データ形式チェック
  - [ ] `validateDateFields()`: 日付項目検証
- [ ] データクレンジング
  - [ ] `cleanInputData()`: 入力データクリーニング
  - [ ] `normalizeData()`: データ正規化

### 2.3 ワークフロー制御

#### 2.3.1 MainController.gs
- [ ] メイン処理
  - [ ] `executeWorkflow()`: ワークフロー実行
  - [ ] `initializeWorkflow()`: 初期化処理
  - [ ] `finalizeWorkflow()`: 終了処理
- [ ] バッチ処理
  - [ ] `processBatch()`: バッチ処理実行
  - [ ] `determineBatchSize()`: バッチサイズ決定
  - [ ] `splitIntoBatches()`: バッチ分割

#### 2.3.2 StateManager.gs
- [ ] 状態管理
  - [ ] `saveState()`: 処理状態保存
  - [ ] `loadState()`: 処理状態読み込み
  - [ ] `clearState()`: 状態クリア
- [ ] 進捗追跡
  - [ ] `trackProgress()`: 進捗追跡
  - [ ] `getProcessedCount()`: 処理済み件数取得
  - [ ] `getErrorCount()`: エラー件数取得

#### 2.3.3 TimeoutHandler.gs
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

#### 2.4.1 ReportGenerator.gs
- [ ] レポート作成
  - [ ] `generateReport()`: レポート生成メイン
  - [ ] `createSummarySheet()`: サマリーシート作成
  - [ ] `createDetailSheet()`: 詳細シート作成
- [ ] フォーマット処理
  - [ ] `formatReportData()`: データフォーマット
  - [ ] `applyConditionalFormatting()`: 条件付き書式
  - [ ] `addCharts()`: グラフ追加

---

## Phase 3: 統合・テスト

### 3.1 統合作業
- [ ] ライブラリ公開設定
  - [ ] バージョン作成
  - [ ] アクセス権限設定
  - [ ] ドキュメント作成
- [ ] クライアント側統合
  - [ ] ライブラリ参照設定
  - [ ] 関数呼び出しテスト
  - [ ] エラーハンドリング確認

### 3.2 単体テスト
- [ ] サーバー側テスト
  - [ ] 判定ロジックテスト
  - [ ] データアクセステスト
  - [ ] 外部連携テスト
- [ ] クライアント側テスト
  - [ ] UI動作テスト
  - [ ] データ検証テスト
  - [ ] ワークフローテスト

### 3.3 結合テスト
- [ ] E2Eテスト
  - [ ] 正常系シナリオ
  - [ ] 異常系シナリオ
  - [ ] 大量データテスト
- [ ] パフォーマンステスト
  - [ ] 処理時間測定
  - [ ] メモリ使用量確認
  - [ ] 同時実行テスト

### 3.4 受入テスト
- [ ] ユーザー受入テスト
  - [ ] 実データでのテスト
  - [ ] 操作性確認
  - [ ] 結果精度確認

---

## Phase 4: デプロイ・運用準備

### 4.1 デプロイ準備
- [ ] 本番環境設定
  - [ ] 本番用スプレッドシート作成
  - [ ] APIキー設定
  - [ ] アクセス権限設定
- [ ] データ移行
  - [ ] マスターデータ移行
  - [ ] テストデータクリア

### 4.2 ドキュメント整備
- [ ] ユーザーマニュアル
  - [ ] 操作手順書
  - [ ] トラブルシューティング
  - [ ] FAQ
- [ ] 運用マニュアル
  - [ ] システム構成図
  - [ ] 障害対応手順
  - [ ] バックアップ手順

### 4.3 運用体制構築
- [ ] 監視設定
  - [ ] エラー通知設定
  - [ ] パフォーマンス監視
  - [ ] ログ監視
- [ ] サポート体制
  - [ ] 問い合わせ窓口設定
  - [ ] エスカレーション手順

### 4.4 リリース作業
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

---

## 完了基準
- [ ] 全機能の実装完了
- [ ] 全テストケースの合格
- [ ] ドキュメントの完成
- [ ] 本番環境での動作確認
- [ ] ユーザー教育の完了