# 入札公告要件判定システム

## 概要
公共工事の入札公告における複雑な要件を自動判定し、企業の入札参加可否を迅速に判断するGoogle Apps Script (GAS) ベースのシステムです。

## 主な機能
- **入札公告管理**: PDF URLの一括登録と管理
- **自動要件抽出**: OCR/LLMを活用した公告文書の解析
- **要件判定**: 6種類の要件に対する自動判定
  - 欠格要件（破産、暴力団関係等）
  - 等級要件（工事種別・等級）
  - 地域要件（拠点所在地）
  - 実績要件（工事実績・成績）
  - 技術者要件（資格保有者）
  - JV要件（共同企業体）
- **バッチ処理**: 100件以上の公告を一括処理
- **レポート生成**: 判定結果の可視化と改善提案

## システム構成
```
┌─────────────────────────────────────────────────────────────┐
│                         クライアント側                         │
│  ・Google Sheets（動的データ管理）                            │
│  ・GAS（ワークフロー制御、UI管理）                           │
└─────────────────────────────────────────────────────────────┘
                                 ↓↑
┌─────────────────────────────────────────────────────────────┐
│                          サーバー側                           │
│  ・Master Sheets（基幹マスター：読み取り専用）                │
│  ・GAS ライブラリ（判定ロジック、OCR/LLM連携）               │
└─────────────────────────────────────────────────────────────┘
                                 ↓↑
┌─────────────────────────────────────────────────────────────┐
│                        外部サービス                           │
│  ・OCR API                                                    │
│  ・LLM API（要件抽出・分類）                                  │
└─────────────────────────────────────────────────────────────┘
```

## 技術スタック
- **プラットフォーム**: Google Apps Script (GAS)
- **開発言語**: TypeScript（JavaScriptにトランスパイル）
- **データストア**: Google Sheets
- **外部連携**: OCR API, LLM API
- **アーキテクチャ**: サーバー/クライアント分離型
- **ビルドツール**: TypeScript Compiler, clasp
- **CI/CD**: GitHub Actions

## ドキュメント構成

### ディレクトリ構造
```
docs/
├── design/                     # 設計関連ドキュメント
│   ├── Project_design.md       # システム全体設計書（アーキテクチャ、ディレクトリ構造含む）
│   ├── Judgment_design.md      # 判定エンジン詳細設計
│   └── Workflow_design.md      # ワークフロー設計
├── requirements/               # 要件定義関連
│   ├── 要件定義書.md            # メイン要件定義書
│   ├── 開発要望.md             # 開発要望まとめ
│   └── appendix/              # 別紙・詳細仕様
│       ├── 別紙1：マスター項目一覧.md      # データベース定義
│       ├── 別紙2：入札公告要件判定詳細.md   # 判定ロジック詳細
│       └── 別紙3：補助シート.md           # 補助資料
├── test/                      # テスト関連
│   ├── Project_Test.md        # テスト仕様書
│   ├── Project_TestCase.md    # テストケース一覧
│   └── Project_Test_TODO.md   # テスト実装TODO
└── todo/                      # TODO管理
    ├── Project_TODO.md        # プロジェクト全体TODO
    ├── Server_TODO.md         # サーバー側実装TODO
    └── Client_TODO.md         # クライアント側実装TODO
```

### ドキュメント参照順序
1. **新規参加者向け**: 要件定義書.md → Project_design.md → Project_TODO.md
2. **実装者向け**: Project_TODO.md → 各設計書（必要に応じて）
3. **テスター向け**: Project_Test.md → Project_TestCase.md → Project_Test_TODO.md

## 開発環境のセットアップ

### 前提条件
- Google アカウント
- Google Apps Script へのアクセス権限
- 関連スプレッドシートへのアクセス権限

### セットアップ手順
1. リポジトリのクローン
   ```bash
   git clone https://github.com/takahiro-shimizu-2/bid-judge-system.git
   cd bid-judge-system
   ```

2. 依存関係のインストールとビルド
   ```bash
   npm install
   npm run build  # TypeScriptをJavaScriptにコンパイル
   ```

3. Google Apps Script プロジェクトの作成
   - サーバー側プロジェクト（ライブラリ）
   - クライアント側プロジェクト
   - claspでプロジェクトと連携
   ```bash
   clasp login
   clasp create --type sheets --title "入札判定システム"
   ```

4. スプレッドシートの準備
   - マスターデータ用スプレッドシート
   - クライアント用スプレッドシート

5. 環境設定
   - スプレッドシートIDの設定
   - APIキーの設定（必要に応じて）
   - .clasprc.jsonの設定

6. デプロイ
   ```bash
   npm run deploy  # TypeScriptビルド → GASへアップロード
   ```

## 使用方法

### 基本的な使い方
1. クライアント用スプレッドシートを開く
2. カスタムメニューから「入札公告判定」を選択
3. 「判定前公告一覧」シートに公告情報を入力
4. 「一括判定実行」をクリック
5. 処理完了後、結果を確認

### 注意事項
- 一度に100件以上の処理を行う場合、タイムアウトにより自動的に再開されます
- OCR処理には時間がかかる場合があります
- マスターデータは定期的に更新が必要です

## 開発ガイドライン

### コーディング規約
- ファイルサイズ: 最大500行
- 関数: 単一責任の原則を遵守
- 命名規則: camelCaseを使用
- コメント: JSDoc形式で記載
- 開発言語: TypeScriptで記述し、JavaScriptにトランスパイル
- モジュール: import/export文は使用せず、名前空間パターンを使用
- ビルド: `npm run build`でJavaScriptファイルを生成

### ブランチ戦略
- `main`: 本番環境
- `develop`: 開発環境
- `feature/*`: 機能開発

### テスト
- 単体テスト: 各モジュールの個別機能
- 結合テスト: モジュール間連携
- システムテスト: エンドツーエンド
- 受入テスト: ユーザー視点での確認

## ライセンス
[ライセンス情報を記載]

## 貢献方法
1. Forkする
2. Feature branchを作成 (`git checkout -b feature/AmazingFeature`)
3. 変更をコミット (`git commit -m 'Add some AmazingFeature'`)
4. Branchにプッシュ (`git push origin feature/AmazingFeature`)
5. Pull Requestを作成

## お問い合わせ
[連絡先情報を記載]

## 更新履歴
- 2024-01-XX: 初版作成