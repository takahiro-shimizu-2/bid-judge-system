# ドキュメント構造

## ディレクトリ構成

```
docs/
├── design/                     # 設計関連ドキュメント
│   ├── Project_design.md       # システム全体設計書
│   ├── Judgment_design.md      # 判定エンジン詳細設計
│   ├── Workflow_design.md      # ワークフロー設計
│   ├── System_Architecture.md  # システムアーキテクチャ
│   └── Project_design_old.md   # 旧バージョン（参考用）
│
├── requirements/               # 要件定義関連
│   ├── 要件定義書.md            # メイン要件定義書
│   ├── 開発要望.md             # 開発要望まとめ
│   ├── 要件定義書_backup.md     # バックアップ
│   └── appendix/              # 別紙・詳細仕様
│       ├── 別紙1：マスター項目一覧.md      # データベース定義
│       ├── 別紙2：入札公告要件判定詳細.md   # 判定ロジック詳細
│       └── 別紙3：補助シート.md           # 補助資料
│
├── test/                      # テスト関連
│   ├── Project_Test.md        # テスト仕様書
│   ├── Project_TestCase.md    # テストケース一覧
│   └── Project_Test_TODO.md   # テスト実装TODO
│
├── todo/                      # TODO管理
│   ├── Project_TODO.md        # プロジェクト全体TODO
│   ├── Server_TODO.md         # サーバー側実装TODO
│   └── Client_TODO.md         # クライアント側実装TODO
│
├── updates_summary.md         # 更新履歴サマリー
└── STRUCTURE.md              # 本ファイル
```

## 各ディレクトリの役割

### design/ - 設計ドキュメント
システムの技術的な設計を記載。実装者向けの詳細な仕様書。

### requirements/ - 要件定義
ビジネス要件と機能要件を記載。何を作るかを定義。

### test/ - テスト関連
テスト計画、テストケース、テスト実装のTODOを管理。

### todo/ - タスク管理
実装タスクをサーバー側/クライアント側に分けて管理。

## ドキュメント参照順序

1. **新規参加者向け**
   - 要件定義書.md → Project_design.md → Project_TODO.md

2. **実装者向け**
   - Project_TODO.md → 各設計書（必要に応じて）

3. **テスター向け**
   - Project_Test.md → Project_TestCase.md → Project_Test_TODO.md

## 更新ルール

- 要件変更時は要件定義書.mdを更新
- 設計変更時は対応する設計書を更新
- 更新内容はupdates_summary.mdに記録
- TODOの進捗は各TODOファイルで管理