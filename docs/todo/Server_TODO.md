# 入札公告要件判定システム - サーバー側TODO

## 関連ドキュメント
- **全体TODO**: `/Project_TODO.md`
- **設計書**: `/docs/design/Judgment_design.md`
- **ワークフロー設計**: `/docs/design/Workflow_design.md`

---

## 概要
サーバー側（GASライブラリ）の開発タスク一覧。チェックボックスで進捗管理。

---

## 1. 環境設定 & 初期構築

### 1.1 プロジェクト設定
[ ] GASプロジェクト（サーバー側）の作成
[ ] ライブラリIDの生成と記録
[ ] バージョン管理の初期設定
[ ] アクセス権限の設定（読み取り専用）

### 1.2 マスタースプレッドシート接続
[ ] スプレッドシートIDの環境変数設定
[ ] 各マスターシートの接続確認
[ ] テストデータでの読み取りテスト
[ ] エラーハンドリングの実装

### 1.3 開発環境構築
[ ] claspのセットアップ
[ ] TypeScript設定（tsconfig.json）
[ ] ESLint設定
[ ] prettier設定

---

## 2. 共通モジュール開発

### 2.1 Constants.ts（定数管理）
```javascript
[ ] // スプレッドシートID定義
    const MASTER_SPREADSHEET_ID = 'xxx';
    const SHEET_NAMES = {
      COMPANY: 'M_COMPANY',
      OFFICE: 'M_OFFICE',
      // ...
    };

[ ] // カラムインデックス定義
    const COLUMNS = {
      COMPANY: {
        ID: 0,
        NAME: 1,
        BANKRUPTCY_FLG: 8,
        // ...
      }
    };

[ ] // エラーメッセージ定義
    const ERROR_MESSAGES = {
      INVALID_INPUT: '入力データが不正です',
      MASTER_NOT_FOUND: 'マスターデータが見つかりません',
      // ...
    };

[ ] // API設定
    const API_CONFIG = {
      OCR_ENDPOINT: PropertiesService.getScriptProperties().getProperty('OCR_API_URL'),
      OCR_TIMEOUT: 30000,
      // ...
    };
```

### 2.2 Utils.ts（ユーティリティ）
```javascript
[ ] // 文字列処理関数
    function convertToHalfWidth(str) {
      // 全角→半角変換ロジック
    }

[ ] // 日付処理関数
    function parseDate(dateStr) {
      // 多様な日付形式の解析
      // - 2025/01/15
      // - ２０２５年１月１５日
      // - 令和7年1月15日
      // - 1月15日（今年とみなす）
    }

[ ] // 和暦変換
    function convertWareki(date) {
      // 西暦→和暦変換
    }

[ ] // バリデーション関数
    function isValidUrl(url) {
      // URL形式チェック
    }

[ ] // 配列処理ユーティリティ
    function chunk(array, size) {
      // 配列を指定サイズで分割
    }
```

### 2.3 Logger.ts（ログ管理）
```javascript
[ ] // ログレベル定義
    const LOG_LEVELS = {
      DEBUG: 0,
      INFO: 1,
      WARN: 2,
      ERROR: 3
    };

[ ] // ログ出力関数
    function log(message, level = 'INFO', details = {}) {
      // ログシートへの書き込み
      // タイムスタンプ、レベル、メッセージ、詳細
    }

[ ] // エラーログ専用
    function logError(error, context = {}) {
      // エラー内容の詳細記録
      // スタックトレース含む
    }

[ ] // パフォーマンスログ
    function logPerformance(operation, duration, details = {}) {
      // 処理時間の記録
    }

[ ] // ログローテーション
    function rotateLogSheet() {
      // 古いログの削除/アーカイブ
    }
```

---

## 3. データアクセス層

### 3.1 MasterDataAccess.ts（基本機能）
```javascript
[ ] // マスターシート取得
    function getMasterSheet(sheetName) {
      // スプレッドシート接続
      // シート取得
      // エラーハンドリング
    }

[ ] // キャッシュ管理
    class CacheManager {
      constructor() {
        this.cache = CacheService.getScriptCache();
      }
      
      get(key) { /* 実装 */ }
      set(key, value, ttl) { /* 実装 */ }
      invalidate(pattern) { /* 実装 */ }
    }

[ ] // データ読み取り基本関数
    function readMasterData(sheetName, filters = {}) {
      // フィルタ条件適用
      // データ取得
      // オブジェクト変換
    }
```

### 3.2 企業マスターアクセス
```javascript
[ ] // 企業情報取得（ID指定）
    function getCompanyById(companyId) {
      // キャッシュチェック
      // マスター読み取り
      // オブジェクト変換
      return {
        id: companyId,
        name: row[1],
        corporateNumber: row[2],
        // ... 各種フラグ
        bankruptcy_flg: row[8] === 'TRUE',
        // ...
      };
    }

[ ] // 企業の欠格フラグ取得
    function getCompanyFlags(companyId) {
      // 欠格関連フラグのみ取得
      return {
        bankruptcy_flg: false,
        violent_relation_flg: false,
        // ...
      };
    }

[ ] // 複数企業一括取得
    function getCompaniesByIds(companyIds) {
      // バッチ読み取り最適化
    }

[ ] // 欠格チェック統合関数
    function checkDisqualification(companyData) {
      // 全欠格フラグのチェック
      // 結果オブジェクト返却
    }
```

### 3.3 拠点マスターアクセス
```javascript
[ ] // 企業の拠点一覧取得
    function getOfficesByCompany(companyId) {
      // 企業IDでフィルタ
      // 拠点情報の配列返却
    }

[ ] // 拠点の登録許可取得
    function getOfficeLicenses(officeId) {
      // T_OFFICE_LICENSEから取得
      // 営業品目と等級のリスト
    }

[ ] // 拠点の実績取得
    function getOfficeExperiences(officeId) {
      // T_OFFICE_EXPERIENCEから取得
      // 工事実績のリスト
    }

[ ] // 地域での拠点検索
    function findOfficesInRegion(companyId, region) {
      // 地域条件での絞り込み
    }
```

### 3.4 従業員・技術者アクセス
```javascript
[ ] // 拠点の従業員一覧
    function getEmployeesByOffice(officeId) {
      // 拠点IDでフィルタ
      // 従業員情報の配列
    }

[ ] // 従業員の資格情報
    function getEmployeeQualifications(employeeId) {
      // T_EMPLOYEE_QUALから取得
      // 保有資格のリスト
    }

[ ] // 資格による技術者検索
    function findEngineersByQualification(companyId, qualifications) {
      // 必要資格を持つ技術者の検索
    }

[ ] // 配置可能技術者の確認
    function getAvailableEngineers(companyId, projectStartDate) {
      // 指定日に配置可能な技術者
    }
```

---

## 4. 判定エンジン実装

### 4.1 JudgmentEngine.ts（メインエンジン）
```javascript
[ ] // エンジン初期化
    class JudgmentEngine {
      constructor() {
        this.initializeJudges();
      }
    }

[ ] // 総合判定実行
    function executeJudgment(announcement, company) {
      // 入力検証
      // コンテキスト準備
      // 各種判定実行
      // 結果統合
    }

[ ] // 判定順序制御
    function getJudgmentOrder() {
      // 欠格要件を最優先
      // その他は並列可能
    }

[ ] // 結果集約
    function aggregateResults(partialResults) {
      // 各判定結果の統合
      // 総合判定の決定
    }

[ ] // 判定結果保存
    function saveJudgmentResult(result) {
      // T_BID_EVALUATIONへ保存
      // 詳細結果も記録
    }
```

### 4.2 DisqualificationJudge.ts（欠格要件）
```javascript
[ ] // 欠格要件判定メイン
    function judgeDisqualification(context) {
      // 企業データ取得
      // 各フラグチェック
      // 結果オブジェクト生成
    }

[ ] // 個別フラグチェック関数
    function checkBankruptcy(company) { }
    function checkViolentRelation(company) { }
    function checkRehabilitation(company) { }
    function checkSuspension(company) { }
    function checkInfoSecurity(company) { }
    function checkSocialInsurance(company) { }
    function checkTerrorist(company) { }
    function checkForeign(company) { }
    function checkBOJ(company) { }

[ ] // 欠格理由メッセージ生成
    function createDisqualificationMessage(reasons) {
      // カテゴリ別に整理
      // 読みやすいメッセージ生成
    }

[ ] // 一括判定最適化
    function judgeDisqualificationBatch(companies, requirements) {
      // 全企業の判定結果を配列に格納
      // 各企業について：
      //   - 引っかかった要件 → 不足要件配列へ
      //   - 引っかからなかった要件 → 充足要件配列へ
      // バッチ書き込み準備
    }

[ ] // 判定結果の一括書き込み
    function writeBatchJudgmentResults(results) {
      // 100件単位でバッチ分割
      // 書き込み進捗の保存
      // 中断時の再開ポイント管理
    }

[ ] // 判定済みフラグの一括更新
    function markRequirementsAsJudged(requirementIds) {
      // T_BID_REQUIREMENTのevaluation_executed_flgを更新
    }
```

### 4.3 GradeJudge.ts（等級要件）
```javascript
[ ] // 等級要件判定メイン
    function judgeGrade(context) {
      // 要求等級の解析
      // 保有等級の取得
      // 比較判定
    }

[ ] // 等級比較ロジック
    function compareGrades(required, actual, condition) {
      // 完全一致
      // 以上条件
      // 等級階層の考慮
    }

[ ] // 工事種別マッチング
    function matchConstructionType(required, actual) {
      // 同等性チェック
      // 類似名称の判定
    }

[ ] // 複数等級要件の処理
    function checkMultipleGradeRequirements(requirements, licenses) {
      // AND/OR条件の処理
    }
```

### 4.4 RegionJudge.ts（地域要件）
```javascript
[ ] // 地域要件判定メイン
    function judgeRegion(context) {
      // 要求地域の解析
      // 拠点所在地の確認
      // 地域包含チェック
    }

[ ] // 地域階層判定
    function identifyRegionType(region) {
      // 全国/地方/都道府県/市区町村
    }

[ ] // 地域包含チェック
    function isInTargetRegion(address, targetRegion) {
      // 住所解析
      // 階層別判定
    }

[ ] // 住所正規化
    function normalizeAddress(address) {
      // 表記ゆれの統一
      // 都道府県/市区町村の抽出
    }

[ ] // 地方と都道府県のマッピング
    const REGION_MAPPING = {
      '関東地方': ['東京都', '神奈川県', ...],
      // ...
    };
```

### 4.5 AchievementJudge.ts（実績要件）
```javascript
[ ] // 実績要件判定メイン
    function judgeAchievement(context) {
      // 要件パターン解析
      // 実績データ収集
      // 条件マッチング
    }

[ ] // 金額要件チェック
    function checkAmountRequirement(requirement, achievements) {
      // 金額解析
      // 期間フィルタ
      // 条件判定
    }

[ ] // 工事成績要件チェック
    function checkScoreRequirement(requirement, achievements) {
      // 平均点計算
      // 最低点チェック
    }

[ ] // JV実績の処理
    function processJVAchievements(achievements) {
      // 出資比率の考慮
      // 代表/構成員の区別
    }

[ ] // 実績の期間フィルタ
    function filterAchievementsByPeriod(achievements, years) {
      // 完成日での絞り込み
    }
```

### 4.6 EngineerJudge.ts（技術者要件）
```javascript
[ ] // 技術者要件判定メイン
    function judgeEngineer(context) {
      // 必要資格の解析
      // 技術者データ収集
      // 配置可能性チェック
    }

[ ] // 資格保有チェック
    function hasRequiredQualification(engineer, required) {
      // 直接保有
      // 同等資格
    }

[ ] // 同等資格マッピング
    const QUALIFICATION_EQUIVALENCE = {
      '1級土木施工管理技士': ['技術士（建設部門）'],
      // ...
    };

[ ] // 配置可能性チェック
    function checkAvailability(engineer, projectPeriod) {
      // 現在の配置状況
      // 期間の重複チェック
    }

[ ] // 複数技術者の割当可能性
    function checkAssignability(requirements, engineers) {
      // 組み合わせ最適化
      // 重複配置の回避
    }
```

---

## 5. 外部連携

### 5.1 OCRConnector.ts
```javascript
[ ] // OCR API設定
    const OCR_CONFIG = {
      endpoint: API_CONFIG.OCR_ENDPOINT,
      apiKey: API_CONFIG.OCR_API_KEY,
      timeout: 30000
    };

[ ] // OCR送信
    function sendToOCR(pdfUrl, options = {}) {
      // リクエスト生成
      // API呼び出し
      // レスポンス処理
      // エラーハンドリング
    }

[ ] // 処理状態確認
    function checkOCRStatus(processId) {
      // ポーリング処理
      // タイムアウト管理
    }

[ ] // 結果取得
    function getOCRResult(processId) {
      // 結果取得
      // フォーマット変換
    }

[ ] // OCR結果パース
    function parseOCRResult(ocrData) {
      // テキスト抽出
      // 構造化データ生成
    }

[ ] // リトライ処理
    function retryOCR(pdfUrl, maxRetries = 3) {
      // 指数バックオフ
      // エラー記録
    }
```

### 5.2 LLMConnector.ts
```javascript
[ ] // LLM API設定
    const LLM_CONFIG = {
      endpoint: API_CONFIG.LLM_ENDPOINT,
      model: 'gemini-pro',
      temperature: 0.1
    };

[ ] // 要件抽出
    function extractRequirements(text) {
      // プロンプト生成
      // API呼び出し
      // 結果パース
    }

[ ] // 要件分類
    function classifyRequirements(requirements) {
      // 6種類への分類
      // 信頼度スコア付与
    }

[ ] // プロンプトテンプレート
    const PROMPTS = {
      EXTRACT: `以下の入札公告文から要件を抽出してください...`,
      CLASSIFY: `以下の要件を6つのカテゴリに分類してください...`
    };

[ ] // レスポンス検証
    function validateLLMResponse(response) {
      // スキーマ検証
      // 必須項目チェック
    }
```

---

## 6. バッチ処理最適化

### 6.1 BatchOptimizer.ts
```javascript
[ ] // 動的バッチサイズ決定
    function determineBatchSize(itemCount, avgProcessingTime) {
      // 利用可能時間の計算
      // 最適サイズの決定
    }

[ ] // 並列処理管理
    function executeParallel(items, processor, maxConcurrent = 5) {
      // Promise管理
      // エラーハンドリング
    }

[ ] // 進捗追跡
    class ProgressTracker {
      constructor(total) { }
      update(processed) { }
      getEstimatedCompletion() { }
    }

[ ] // メモリ管理
    function monitorMemoryUsage() {
      // 使用量チェック
      // 必要に応じてGC
    }
```

---

## 7. エラーハンドリング

### 7.1 ErrorHandler.ts
```javascript
[ ] // エラー分類
    function classifyError(error) {
      // タイムアウト
      // API制限
      // データ不正
      // システムエラー
    }

[ ] // エラー対応戦略
    function determineErrorStrategy(errorType) {
      // リトライ
      // スキップ
      // 中断
      // 通知
    }

[ ] // エラー通知
    function notifyError(error, context) {
      // メール送信
      // ログ記録
      // Slack通知（オプション）
    }

[ ] // リカバリ処理
    function recoverFromError(error, state) {
      // 状態復元
      // 部分的再実行
    }
```

---

## 8. パフォーマンステスト

### 8.1 性能測定
[ ] 100件バッチ処理の時間測定
[ ] メモリ使用量のプロファイリング
[ ] API呼び出しのボトルネック特定
[ ] キャッシュヒット率の測定

### 8.2 最適化
[ ] データベースクエリの最適化
[ ] キャッシュ戦略の改善
[ ] 並列度の調整
[ ] 不要な処理の削除

---

## 9. セキュリティ実装

### 9.1 アクセス制御
[ ] ライブラリの読み取り専用設定
[ ] APIキーの暗号化保存
[ ] 実行権限の最小化
[ ] 監査ログの実装

### 9.2 データ保護
[ ] 機密情報のマスキング
[ ] 一時データの確実な削除
[ ] 通信の暗号化確認

---

## 10. ドキュメント作成

### 10.1 APIドキュメント
[ ] 各公開関数のJSDoc記載
[ ] 使用例の追加
[ ] パラメータ詳細説明
[ ] 戻り値の型定義

### 10.2 統合ガイド
[ ] ライブラリの使用方法
[ ] 初期設定手順
[ ] トラブルシューティング
[ ] FAQ作成

---

## 完了基準
- [ ] 全単体テストがパス
- [ ] コードカバレッジ80%以上
- [ ] パフォーマンス基準達成
- [ ] セキュリティレビュー完了
- [ ] ドキュメント完成

---

## 改訂履歴
- 2024-01-XX: 初版作成