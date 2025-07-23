# 入札公告要件判定システム - 開発仕様書

## 関連ドキュメント
- **システムアーキテクチャ**: `/docs/design/System_Architecture.md`
- **要件定義書**: `/docs/requirements/要件定義書.md`
- **マスター項目一覧**: `/docs/requirements/appendix/別紙1：マスター項目一覧.md`
- **判定詳細**: `/docs/requirements/appendix/別紙2：入札公告要件判定詳細.md`
- **補助シート**: `/docs/requirements/appendix/別紙3：補助シート.md`
- **TODO リスト**: `/docs/todo/Project_TODO.md`
- **テスト仕様**: `/docs/test/Project_Test.md`

---

## 1. システム概要

### 1.1 目的
公共工事の入札公告における複雑な要件を自動判定し、企業の入札参加可否を迅速に判断するシステムを構築する。

### 1.2 主要機能
1. **入札公告管理**: PDF URLの登録と管理
2. **OCR処理**: 外部OCRツールによる公告文書の解析
3. **要件判定**: ルールベースによる自動判定
4. **結果レポート**: 判定結果の可視化と改善提案

### 1.3 成功指標
- 100件以上の公告を一括処理可能
- 判定精度 95%以上
- 処理時間 1公告あたり平均30秒以内

### 1.4 ユーザーペルソナ
1. **営業担当者**: 入札案件を探し、自社が参加可能か迅速に判断したい
2. **営業責任者**: 複数案件の判定結果を俯瞰し、戦略的に入札を決定したい
3. **システム管理者**: マスターデータの更新や判定ルールの調整を行いたい

### 1.5 利用シナリオ
```
┌─────────────────┐
│ 1. ログイン      │  ← Googleアカウント認証
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 2. 公告登録      │  ← CSVまたは手動入力
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 3. 一括判定実行  │  ← 100件を自動処理
└────────┬────────┘
         │ (非同期処理)
         ▼
┌─────────────────┐
│ 4. 結果確認      │  ← レポート生成・ダウンロード
└─────────────────┘
```

---

## 2. システムアーキテクチャ

### 2.1 全体構成図
```
┌─────────────────────────────────────────────────────────────────┐
│                           ユーザー環境                            │
├─────────────────────────────────────────────────────────────────┤
│  ┌────────────────────┐    ┌────────────────────┐              │
│  │ クライアント側      │    │ マスター管理用     │              │
│  │ スプレッドシート    │    │ スプレッドシート   │              │
│  │ (Client)           │    │ (Master)          │              │
│  └──────────┬─────────┘    └──────────┬─────────┘              │
│             │                         │                          │
│             └───────────┬─────────────┘                          │
│                         ↓                                        │
│            ┌────────────────────────┐                           │
│            │ サーバー側ライブラリ   │                           │
│            │ (Server Library)      │                           │
│            └────────────────────────┘                           │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                         外部サービス                              │
├─────────────────────────────────────────────────────────────────┤
│  ┌────────────┐    ┌────────────┐    ┌────────────┐          │
│  │ OCR API    │    │ LLM API    │    │ Google     │          │
│  │            │    │            │    │ Services   │          │
│  └────────────┘    └────────────┘    └────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 3層構造の詳細

#### 2.2.1 クライアント層（利用者スプレッドシート）
**役割**: エンドユーザーが直接操作するインターフェース

**主要機能**:
- カスタムメニューによる操作
- 判定前公告の入力・管理
- 判定結果の表示
- レポート生成

**構成モジュール**:
- UI管理（UIManager.gs）
- ワークフロー制御（WorkflowEngine.gs）
- データ管理（DataManager.gs）
- エラーハンドリング（ErrorHandler.gs）

#### 2.2.2 サーバー層（共有ライブラリ）
**役割**: 判定ロジックとマスターデータアクセスを提供する共有ライブラリ

**主要機能**:
- 判定エンジン（6種類の要件判定）
- マスターデータへのアクセス
- 外部API連携（OCR/LLM）
- 共通ユーティリティ

**公開インターフェース**:
```javascript
// ライブラリとして公開される関数
function executeJudgment(companyId, requirements) {}
function getCompanyData(companyId) {}
function getJudgmentResult(evaluationId) {}
```

#### 2.2.3 マスター層（基幹データ管理）
**役割**: マスターデータの管理と更新を行う管理者用インターフェース

**管理データ**:
- 企業マスター（M_COMPANY）
- 拠点マスター（M_OFFICE）
- 従業員マスター（M_EMPLOYEE）
- 各種実績・資格データ

### 2.3 データフロー

#### 2.3.1 判定処理フロー
```
[クライアント]
    ↓ 1. 判定要求
[サーバーライブラリ]
    ↓ 2. マスターデータ取得
[マスター管理]
    ↓ 3. データ返却
[サーバーライブラリ]
    ↓ 4. 判定処理実行
    ↓ 5. 結果返却
[クライアント]
    ↓ 6. 結果表示
```

#### 2.3.2 マスターデータ更新フロー
```
[マスター管理]
    ↓ 1. データ更新
    ↓ 2. 整合性チェック
    ↓ 3. バージョン更新
[サーバーライブラリ]
    ↓ 4. キャッシュクリア
```

### 2.4 技術スタック
- **プラットフォーム**: Google Apps Script (GAS)
- **データストア**: Google Sheets
- **外部連携**: OCR API, LLM API
- **開発ツール**: clasp, Git
- **アーキテクチャパターン**: 3層構造、サーバー/クライアント分離

### 2.5 非機能要件

#### パフォーマンス
- 1件あたりの判定時間: 30秒以内
- 同時処理可能数: 5ユーザー
- キャッシュ有効期間: 1時間

#### 可用性
- 稼働時間: 24/7（メンテナンス時除く）
- タイムアウト自動復旧機能
- エラー時の部分的処理継続

#### 拡張性
- モジュール単位での機能追加
- 判定ルールの柔軟な変更
- 外部APIの追加対応

---

## 3. モジュール設計

### 3.1 クライアント側モジュール (src/client)

#### 3.1.1 MainController.gs (src/client/Controller/)
- **責務**: ワークフロー全体の制御
- **主要関数**:
  ```javascript
  /**
   * ワークフロー実行のメインエントリーポイント
   * @param {Object} options - 実行オプション
   * @param {boolean} options.isResume - 再開フラグ
   * @param {number} options.batchSize - バッチサイズ
   * @returns {Object} 実行結果
   */
  function executeWorkflow(options = {}) {
    // 1. 初期化処理
    // 2. 進捗状態の読み込み
    // 3. タイムアウトトリガー設定
    // 4. バッチ処理実行
    // 5. 完了処理
  }
  
  /**
   * タイムアウト時の再開処理
   * @param {Event} e - トリガーイベント
   */
  function handleTimeout(e) {
    // 1. 保存された状態を読み込み
    // 2. 再開ポイントから処理継続
    // 3. 新しいトリガー設定
  }
  ```

#### 3.1.2 UIManager.gs (src/client/UI/)
- **責務**: カスタムメニューとUI制御
- **主要関数**:
  - `createCustomMenu()`: メニュー作成
  - `showProgressDialog()`: 進捗表示

#### 3.1.3 DataValidator.gs (src/client/Data/)
- **責務**: 入力データの検証
- **主要関数**:
  - `validateInput()`: 必須項目チェック
  - `parseDates()`: 日付パース処理

### 3.2 サーバー側モジュール（ライブラリ）(src/server)

#### 3.2.1 JudgmentEngine.gs (src/server/Judgment/)
- **責務**: 要件判定の中核処理
- **主要関数**:
  - `executeJudgment()`: 統合判定実行
  - `aggregateResults()`: 判定結果集約

#### 3.2.2 OCRConnector.gs (src/server/External/)
- **責務**: 外部OCR APIとの連携
- **主要関数**:
  - `sendToOCR()`: OCR処理要求
  - `parseOCRResult()`: 結果パース

### 3.3 マスター管理モジュール (src/master)

#### 3.3.1 AdminMenu.gs (src/master/Admin/)
- **責務**: 管理者向け機能の提供
- **主要関数**:
  - `createAdminMenu()`: 管理メニュー作成
  - `importMasterData()`: マスターデータインポート

---

## 4. データ設計

### 4.1 スプレッドシート構成

#### 4.1.1 クライアント側スプレッドシート
| シート名 | 用途 | 主要カラム |
|----------|------|------------|
| 判定前公告一覧 | 判定対象の公告情報 | 公告ID, 公告名, PDF URL, ステータス |
| 判定後公告一覧 | 判定結果サマリー | 公告ID, 判定日時, 適格企業数, 処理時間 |
| 入札公告詳細 | 公告の詳細情報 | 公告ID, 発注機関, 工事概要, 要件一覧 |
| レポート | 判定結果レポート | 企業ID, 企業名, 判定結果, NG理由 |

#### 4.1.2 マスタースプレッドシート
| シート名 | 用途 | 主要カラム |
|----------|------|------------|
| 企業マスター | 企業基本情報 | 企業ID, 企業名, 各種欠格フラグ |
| 拠点マスター | 企業の拠点情報 | 拠点ID, 企業ID, 所在地, 拠点種別 |
| 従業員マスター | 技術者情報 | 従業員ID, 拠点ID, 資格情報 |
| ライセンス・実績マスター | 許可・実績情報 | 参照ID, カテゴリ, 詳細情報 |

### 4.2 データモデル

```javascript
// 公告情報
const Announcement = {
  id: String,              // 公告ID
  name: String,            // 公告名
  pdfUrl: String,          // PDF URL
  issuer: String,          // 発注機関
  publishDate: Date,       // 公告日
  deadline: Date,          // 入札締切日
  requirements: Object,    // 要件オブジェクト
  status: String          // 処理ステータス
};

// 判定結果
const JudgmentResult = {
  announcementId: String,  // 公告ID
  companyId: String,       // 企業ID
  overallResult: String,   // 総合判定結果
  details: {
    disqualification: Object,  // 欠格要件判定
    grade: Object,            // 等級要件判定
    region: Object,           // 地域要件判定
    achievement: Object,      // 実績要件判定
    engineer: Object          // 技術者要件判定
  },
  timestamp: Date         // 判定日時
};
```

---

## 5. 処理フロー設計

### 5.1 メイン処理フロー
```
┌─────────────────┐
│ 1. 初期化処理   │
└────────┬────────┘
         ↓
┌─────────────────┐
│ 2. データ読込   │ ← 判定前公告一覧から
└────────┬────────┘
         ↓
┌─────────────────┐
│ 3. バッチ分割   │ ← 100件単位
└────────┬────────┘
         ↓
┌─────────────────┐
│ 4. OCR処理      │ ← 並列実行
└────────┬────────┘
         ↓
┌─────────────────┐
│ 5. 要件判定     │ ← 6種類の判定を実行
└────────┬────────┘
         ↓
┌─────────────────┐
│ 6. 結果保存     │ → 判定後公告一覧へ
└────────┬────────┘
         ↓
┌─────────────────┐
│ 7. レポート生成 │
└─────────────────┘
```


### 5.2 タイムアウト処理
- 実行時間が5分を超えた場合、自動的に処理を中断
- 処理状態を保存し、トリガーを設定して1分後に再開
- 再開時は中断したポイントから処理を継続

### 5.3 エラーハンドリング
- 各処理ステップでtry-catchによるエラー捕捉
- エラー発生時も他の公告の処理は継続
- エラー情報はログシートに記録

---

## 6. API設計

### 6.1 サーバー側ライブラリ公開API

```javascript
/**
 * 企業の判定を実行
 * @param {string} companyId - 企業ID
 * @param {Object} requirements - 要件オブジェクト
 * @returns {Object} 判定結果
 */
function executeJudgment(companyId, requirements) {
  // 実装
}

/**
 * 企業情報を取得
 * @param {string} companyId - 企業ID
 * @returns {Object} 企業情報
 */
function getCompanyData(companyId) {
  // 実装
}

/**
 * 判定結果を取得
 * @param {string} evaluationId - 評価ID
 * @returns {Object} 判定結果詳細
 */
function getJudgmentResult(evaluationId) {
  // 実装
}
```

### 6.2 外部API連携

#### OCR API
- エンドポイント: 設定により指定
- メソッド: POST
- リクエスト: PDF URL
- レスポンス: テキスト抽出結果

#### LLM API
- エンドポイント: 設定により指定
- メソッド: POST
- リクエスト: 抽出テキスト + プロンプト
- レスポンス: 構造化された要件情報

---

## 7. セキュリティ設計

### 7.1 アクセス制御
- Google アカウントによる認証
- スプレッドシートの共有設定による権限管理
- ライブラリは読み取り専用アクセス

### 7.2 データ保護
- APIキーは PropertiesService で管理
- 個人情報を含むログは最小限に
- HTTPSによる通信の暗号化

### 7.3 監査
- 全操作をログシートに記録
- 実行者、実行時間、処理内容を保存

### 7.4 監視項目
- API使用量
- エラー発生率
- 処理時間統計

### 7.5 定期メンテナンス
- マスターデータの整合性チェック（週次）
- ログファイルのローテーション（月次）
- パフォーマンスレビュー（四半期）

---

## 8. パフォーマンス設計

### 8.1 処理の最適化
- バッチ処理による効率化（50-100件単位）
- 並列処理可能な部分は UrlFetchApp.fetchAll() を使用
- キャッシュサービスによるマスターデータのキャッシング

### 8.2 制限事項への対応
- スクリプト実行時間: 6分制限 → 5分でタイムアウト処理を実行
- API呼び出し回数: 日次制限内に収まるよう設計
- スプレッドシートセル数: 1000万セル制限を考慮

---

## 9. 開発ガイドライン

### 9.1 コーディング規約
- 関数名: キャメルケース（例: executeWorkflow）
- 変数名: キャメルケース（例: companyData）
- 定数名: アッパースネークケース（例: MAX_BATCH_SIZE）
- コメント: JSDoc形式で記述

### 9.2 TypeScript → JavaScript → GAS 変換規約

#### 9.2.1 ファイル構成
- **各ファイルは単一目的に特化**（1ファイル = 1モジュール）
- **import/export文は使用しない**（GASはES6モジュールをサポートしない）
- **名前空間パターンを使用**してモジュール化を実現

#### 9.2.2 TypeScript記述時の注意点
```typescript
// ❌ 悪い例：ES6モジュール
export class JudgmentEngine {
  // ...
}

// ✅ 良い例：名前空間パターン
namespace JudgmentModule {
  export class JudgmentEngine {
    // ...
  }
}
```

#### 9.2.3 GAS固有の制約対応
1. **グローバルスコープでの関数定義**
   - GASのエントリーポイントはグローバル関数である必要
   - カスタムメニューから呼び出される関数はグローバルに配置

2. **非同期処理の制限**
   - async/awaitは使用可能だが、並列処理に制限
   - UrlFetchApp.fetchAll()を使用して並列リクエスト

3. **型定義ファイル**
   - @types/google-apps-scriptを使用
   - カスタム型定義は.d.tsファイルに集約

4. **ビルド設定**
   ```json
   // tsconfig.json
   {
     "compilerOptions": {
       "target": "ES5",
       "module": "none",
       "lib": ["ES2015"],
       "outDir": "./dist/js"
     }
   }
   ```

### 9.3 モジュール設計原則
- 単一責任の原則（SRP）を遵守
- 各モジュールは300行以内
- 依存関係は最小限に
- 名前空間による論理的分離

### 9.4 テスト方針
- 単体テスト: 各関数の動作確認
- 結合テスト: モジュール間連携の確認
- E2Eテスト: 実際のシナリオでの動作確認

---

## 10. デプロイメント

### 10.1 開発環境
- ローカル開発: TypeScript + clasp
- バージョン管理: Git
- コード共有: GitHub
- ビルドツール: npm scripts + tsc

### 10.2 CI/CDパイプライン

#### 10.2.1 開発フロー
```
1. TypeScript開発（src/ts/）
   ↓
2. TypeScript → JavaScript変換（npm run build）
   ↓
3. JavaScript → GAS変換（clasp push）
   ↓
4. テスト実行（GAS上で実行）
   ↓
5. 本番デプロイ（clasp deploy）
```

#### 10.2.2 GitHub Actions設定
```yaml
# .github/workflows/deploy.yml
name: Deploy to GAS
on:
  push:
    branches: [main, develop]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build TypeScript
        run: npm run build
      
      - name: Run tests
        run: npm test
      
      - name: Deploy to GAS (develop)
        if: github.ref == 'refs/heads/develop'
        run: npm run deploy:dev
        env:
          CLASP_CREDENTIALS: ${{ secrets.CLASP_CREDENTIALS_DEV }}
      
      - name: Deploy to GAS (production)
        if: github.ref == 'refs/heads/main'
        run: npm run deploy:prod
        env:
          CLASP_CREDENTIALS: ${{ secrets.CLASP_CREDENTIALS_PROD }}
```

#### 10.2.3 環境別設定
- **開発環境**: developブランチから自動デプロイ
- **ステージング環境**: 手動デプロイ + 統合テスト
- **本番環境**: mainブランチから承認後デプロイ

### 10.3 リリース手順
1. TypeScriptソースのビルド
2. 単体テストの実行
3. clasp pushでGASへアップロード
4. GAS上での統合テスト
5. ライブラリのバージョニング（サーバー側）
6. クライアント側のライブラリ参照更新
7. 本番環境での動作確認

### 10.4 ロールバック手順
1. GASのバージョン管理から前バージョンを選択
2. ライブラリのバージョンを前バージョンに戻す
3. クライアント側の参照を更新
4. 動作確認

---

## 11. プロジェクトディレクトリ構造

### 11.1 全体構成
```
bid-judge-system/
├── server/                          # サーバー側（共有ライブラリ）
│   ├── src/
│   │   ├── ts/                     # TypeScriptソース
│   │   │   ├── api/                # 外部API連携
│   │   │   │   ├── OCRConnector.ts
│   │   │   │   └── LLMConnector.ts
│   │   │   ├── data/               # データアクセス層
│   │   │   │   ├── MasterDataAccess.ts
│   │   │   │   └── DataValidator.ts
│   │   │   ├── judgment/           # 判定エンジン
│   │   │   │   ├── JudgmentEngine.ts
│   │   │   │   ├── DisqualificationJudge.ts
│   │   │   │   ├── GradeJudge.ts
│   │   │   │   ├── RegionJudge.ts
│   │   │   │   ├── AchievementJudge.ts
│   │   │   │   ├── EngineerJudge.ts
│   │   │   │   └── OtherJudge.ts
│   │   │   ├── utils/              # 共通ユーティリティ
│   │   │   │   ├── Utils.ts
│   │   │   │   └── Constants.ts
│   │   │   └── types/              # 型定義
│   │   │       └── index.d.ts
│   │   ├── js/                     # コンパイル済みJavaScript
│   │   │   └── (TypeScriptからの出力)
│   │   └── gas/                    # GAS用ファイル（clasp push先）
│   │       └── (JavaScriptからの変換)
│   ├── masters/                    # マスターデータ（スプレッドシート）
│   │   ├── M_COMPANY/
│   │   ├── M_OFFICE/
│   │   ├── M_EMPLOYEE/
│   │   └── ...
│   └── config/
│       └── ServerConfig.gs
│
├── client/                         # クライアント側（利用者スプレッドシート）
│   ├── src/
│   │   ├── ts/                    # TypeScriptソース
│   │   │   ├── ui/               # UI関連
│   │   │   │   ├── UIManager.ts
│   │   │   │   ├── MenuBuilder.ts
│   │   │   │   └── DialogManager.ts
│   │   │   ├── workflow/         # ワークフロー制御
│   │   │   │   ├── MainController.ts
│   │   │   │   ├── WorkflowEngine.ts
│   │   │   │   ├── StateManager.ts
│   │   │   │   └── TimeoutHandler.ts
│   │   │   ├── stages/           # 処理ステージ
│   │   │   │   ├── TransferStage.ts
│   │   │   │   ├── OCRStage.ts
│   │   │   │   ├── ClassificationStage.ts
│   │   │   │   ├── JudgmentStage.ts
│   │   │   │   └── ReportStage.ts
│   │   │   ├── data/             # データ管理
│   │   │   │   ├── DataManager.ts
│   │   │   │   ├── DataValidator.ts
│   │   │   │   └── SheetManager.ts
│   │   │   ├── utils/            # ユーティリティ
│   │   │   │   ├── Logger.ts
│   │   │   │   └── ErrorHandler.ts
│   │   │   └── types/            # 型定義
│   │   │       └── index.d.ts
│   │   ├── js/                   # コンパイル済みJavaScript
│   │   └── gas/                  # GAS用ファイル
│   ├── templates/                  # HTMLテンプレート
│   │   ├── dialogs/
│   │   │   ├── settings.html
│   │   │   ├── progress.html
│   │   │   └── help.html
│   │   └── styles/
│   │       └── style.css
│   └── config/
│       └── ClientConfig.gs
│
├── tests/                          # テストコード
│   ├── unit/                      # 単体テスト
│   │   ├── server/
│   │   └── client/
│   ├── integration/               # 結合テスト
│   └── e2e/                      # E2Eテスト
│
├── docs/                          # ドキュメント（現在整備済み）
│   ├── design/
│   ├── requirements/
│   ├── test/
│   └── todo/
│
├── scripts/                       # ビルド・デプロイスクリプト
│   ├── build.sh                   # TypeScript → JavaScript変換
│   ├── deploy-server.sh           # サーバー側デプロイ
│   ├── deploy-client.sh           # クライアント側デプロイ
│   └── setup-environment.sh       # 環境セットアップ
│
├── .github/                       # GitHub Actions
│   └── workflows/
│       ├── test.yml              # テスト自動実行
│       ├── build.yml             # ビルド検証
│       └── deploy.yml            # 自動デプロイ
│
├── package.json                   # npm設定
├── tsconfig.json                  # TypeScript設定
├── .clasprc.json                  # clasp設定
├── .gitignore
├── README.md
└── LICENSE
```

### 11.2 各ディレクトリの詳細

#### server/ - サーバー側実装
共有ライブラリとして実装。複数のクライアントから利用される。

- **api/**: 外部サービス（OCR、LLM）との連携
- **data/**: マスターデータへのアクセス層
- **judgment/**: 要件判定ロジックの実装
- **utils/**: 共通処理（日付変換、文字列処理など）

#### client/ - クライアント側実装
利用者のスプレッドシートに紐づくGASプロジェクト。

- **ui/**: カスタムメニュー、ダイアログ、プログレスバー
- **workflow/**: 処理フロー制御、状態管理、タイムアウト処理
- **stages/**: 各処理ステージ（転記、OCR、判定など）の実装
- **data/**: シートへの読み書き、データ検証

#### tests/ - テストコード
- **unit/**: 個々の関数のテスト
- **integration/**: 複数モジュールの連携テスト
- **e2e/**: 全体フローのテスト

### 11.3 ファイル命名規則

- TypeScriptファイル: PascalCase + .ts（例: JudgmentEngine.ts）
- JavaScriptファイル: PascalCase + .js（例: JudgmentEngine.js）
- GASファイル: PascalCase + .gs（例: JudgmentEngine.gs）
- HTMLファイル: kebab-case + .html（例: progress-dialog.html）
- 設定ファイル: PascalCase + Config.ts（例: ServerConfig.ts）

### 11.4 ビルドフロー

```
1. src/ts/*.ts → src/js/*.js (tsc)
   - TypeScriptコンパイル
   - 型チェック実行
   
2. src/js/*.js → src/gas/*.gs (custom script)
   - import/export文の除去
   - 名前空間への変換
   - GAS固有の調整
   
3. src/gas/*.gs → Google Apps Script (clasp push)
   - GASプロジェクトへアップロード
   - マニフェスト更新
```

### 11.4 ドキュメント構成

```
docs/
├── requirements/               # 要件定義関連
│   ├── 要件定義書.md           # システム要件定義書（主要）
│   ├── 要件定義書_backup.md    # バックアップ
│   ├── 開発要望.md            # 開発要望まとめ
│   └── appendix/              # 別紙・詳細仕様
│       ├── 別紙1：マスター項目一覧.md      # データベース定義
│       ├── 別紙2：入札公告要件判定詳細.md  # 判定ロジック詳細
│       └── 別紙3：補助シート.md           # 補助資料
│
├── design/                     # 設計書
│   ├── Project_design.md       # 全体設計書（本書）
│   ├── Project_design_old.md   # 旧バージョン（参考）
│   ├── Workflow_design.md      # ワークフロー詳細設計
│   ├── Judgment_design.md      # 判定エンジン詳細設計
│   ├── DataAccess_design.md    # データアクセス詳細設計【未作成】
│   ├── UI_design.md            # UI詳細設計【未作成】
│   ├── API_design.md           # API詳細設計【未作成】
│   └── Security_design.md      # セキュリティ詳細設計【未作成】
│
├── todo/                       # TODO管理
│   ├── Project_TODO.md         # 全体TODO（主要）
│   ├── Server_TODO.md          # サーバー側TODO
│   ├── Client_TODO.md          # クライアント側TODO
│   ├── Master_TODO.md          # マスター管理TODO【未作成】
│   └── Integration_TODO.md     # 統合・連携TODO【未作成】
│
├── test/                       # テスト仕様
│   ├── Project_Test.md         # テスト仕様書
│   ├── Project_TestCase.md     # テストケース表
│   ├── Project_Test_TODO.md    # テスト実装TODO
│   ├── Unit_Test_Spec.md       # 単体テスト仕様【未作成】
│   ├── Integration_Test_Spec.md # 結合テスト仕様【未作成】
│   └── UAT_Test_Spec.md        # 受入テスト仕様【未作成】
│
├── operation/                  # 運用関連【未作成】
│   ├── Operation_Manual.md     # 運用マニュアル
│   ├── Admin_Guide.md          # 管理者ガイド
│   ├── User_Guide.md           # 利用者ガイド
│   └── Troubleshooting.md      # トラブルシューティング
│
└── updates_summary.md          # 更新履歴サマリー
```

#### 各ドキュメントの役割

**要件定義関連**
- 要件定義書.md: ビジネス要件と機能要件の定義
- 別紙1-3: 詳細仕様とデータ定義

**設計書**
- Project_design.md: 全体アーキテクチャとモジュール構成
- Workflow_design.md: 処理フローとタイムアウト処理
- Judgment_design.md: 6種類の判定ロジック詳細
- DataAccess_design.md: マスターデータアクセス層の設計【要作成】
- UI_design.md: 画面設計とユーザビリティ【要作成】
- API_design.md: 外部API連携とライブラリ公開API【要作成】
- Security_design.md: セキュリティ実装詳細【要作成】

**TODO管理**
- Project_TODO.md: 全体のタスク管理と優先順位
- Server_TODO.md: サーバー側実装タスク
- Client_TODO.md: クライアント側実装タスク
- Master_TODO.md: マスター管理機能の実装【要作成】
- Integration_TODO.md: システム統合タスク【要作成】

**テスト仕様**
- Project_Test.md: 全体的なテスト計画
- Project_TestCase.md: 具体的なテストケース
- Project_Test_TODO.md: テスト実装タスク
- Unit_Test_Spec.md: 関数レベルのテスト仕様【要作成】
- Integration_Test_Spec.md: モジュール連携テスト【要作成】
- UAT_Test_Spec.md: ユーザー受入テスト【要作成】

**運用関連**【要作成】
- Operation_Manual.md: 日常運用手順
- Admin_Guide.md: システム管理者向けガイド
- User_Guide.md: エンドユーザー向けガイド
- Troubleshooting.md: 問題解決ガイド

### 11.5 スプレッドシート構成

#### サーバー側マスターシート
```
入札判定システム_マスター/
├── M_COMPANY（企業マスター）
├── M_OFFICE（拠点マスター）
├── M_EMPLOYEE（従業員マスター）
├── T_OFFICE_LICENSE（拠点登録許可）
├── T_OFFICE_EXPERIENCE（拠点工事実績）
├── T_EMPLOYEE_QUAL（従業員資格）
├── M_AGENCY（発注機関マスター）
├── M_CONSTRUCTION_TYPE（営業品目マスター）
└── M_QUALIFICATION（技術者資格マスター）
```

#### クライアント側シート
```
入札判定システム_利用者/
├── 判定前公告一覧
├── 入札公告マスター
├── 公告要件マスター
├── 企業公告判定マスター
├── 充足要件リスト
├── 不足要件リスト
├── 要件拠点中間テーブル
├── 報告用シート
├── LOG
└── 設定
```

---

## 改訂履歴
- 2024-01-XX: 初版作成
- 2024-01-XX: システムアーキテクチャの明確化（3層構造）
- 2024-01-XX: プロジェクトディレクトリ構造を追加
- 2024-01-XX: System_Architecture.mdの内容を統合
- 2024-01-XX: TypeScript→JavaScript→GAS変換フローを追加
- 2024-01-XX: CI/CDパイプライン設計を追加