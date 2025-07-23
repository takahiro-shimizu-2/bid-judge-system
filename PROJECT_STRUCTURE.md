# プロジェクトディレクトリ構造

## 全体構成

```
bid-judge-system/
├── server/                          # サーバー側（共有ライブラリ）
│   ├── src/
│   │   ├── api/                    # 外部API連携
│   │   │   ├── OCRConnector.gs
│   │   │   └── LLMConnector.gs
│   │   ├── data/                   # データアクセス層
│   │   │   ├── MasterDataAccess.gs
│   │   │   └── DataValidator.gs
│   │   ├── judgment/               # 判定エンジン
│   │   │   ├── JudgmentEngine.gs
│   │   │   ├── DisqualificationJudge.gs
│   │   │   ├── GradeJudge.gs
│   │   │   ├── RegionJudge.gs
│   │   │   ├── AchievementJudge.gs
│   │   │   ├── EngineerJudge.gs
│   │   │   └── OtherJudge.gs
│   │   └── utils/                  # 共通ユーティリティ
│   │       ├── Utils.gs
│   │       └── Constants.gs
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
│   │   ├── ui/                    # UI関連
│   │   │   ├── UIManager.gs
│   │   │   ├── MenuBuilder.gs
│   │   │   └── DialogManager.gs
│   │   ├── workflow/              # ワークフロー制御
│   │   │   ├── MainController.gs
│   │   │   ├── WorkflowEngine.gs
│   │   │   ├── StateManager.gs
│   │   │   └── TimeoutHandler.gs
│   │   ├── stages/                # 処理ステージ
│   │   │   ├── TransferStage.gs
│   │   │   ├── OCRStage.gs
│   │   │   ├── ClassificationStage.gs
│   │   │   ├── JudgmentStage.gs
│   │   │   └── ReportStage.gs
│   │   ├── data/                  # データ管理
│   │   │   ├── DataManager.gs
│   │   │   ├── DataValidator.gs
│   │   │   └── SheetManager.gs
│   │   └── utils/                 # ユーティリティ
│   │       ├── Logger.gs
│   │       └── ErrorHandler.gs
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
│   ├── deploy-server.sh
│   ├── deploy-client.sh
│   └── setup-environment.sh
│
├── .github/                       # GitHub Actions
│   └── workflows/
│       ├── test.yml
│       └── deploy.yml
│
├── README.md
├── LICENSE
└── .gitignore
```

## 各ディレクトリの詳細

### server/ - サーバー側実装
共有ライブラリとして実装。複数のクライアントから利用される。

- **api/**: 外部サービス（OCR、LLM）との連携
- **data/**: マスターデータへのアクセス層
- **judgment/**: 要件判定ロジックの実装
- **utils/**: 共通処理（日付変換、文字列処理など）

### client/ - クライアント側実装
利用者のスプレッドシートに紐づくGASプロジェクト。

- **ui/**: カスタムメニュー、ダイアログ、プログレスバー
- **workflow/**: 処理フロー制御、状態管理、タイムアウト処理
- **stages/**: 各処理ステージ（転記、OCR、判定など）の実装
- **data/**: シートへの読み書き、データ検証

### tests/ - テストコード
- **unit/**: 個々の関数のテスト
- **integration/**: 複数モジュールの連携テスト
- **e2e/**: 全体フローのテスト

## ファイル命名規則

- GASファイル: PascalCase + .gs（例: JudgmentEngine.gs）
- HTMLファイル: kebab-case + .html（例: progress-dialog.html）
- 設定ファイル: PascalCase + Config.gs（例: ServerConfig.gs）

## スプレッドシート構成

### サーバー側マスターシート
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

### クライアント側シート
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

## デプロイ構成

1. **サーバー側**
   - GASライブラリとして公開
   - バージョン管理
   - 権限は読み取り専用

2. **クライアント側**
   - 各利用者のスプレッドシートにバインド
   - サーバーライブラリを参照
   - 実行権限は各利用者

## 開発フロー

1. ローカルでコード開発（clasp使用）
2. テスト環境へデプロイ
3. 単体テスト・結合テスト実施
4. 本番環境へデプロイ
5. ライブラリバージョン更新（サーバー側）