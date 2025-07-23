# 入札公告要件判定システム - 統合・デプロイTODO

## 関連ドキュメント
- **全体TODO**: `/docs/todo/Project_TODO.md`
- **全体設計書**: `/docs/design/Project_design.md`
- **CI/CD設計**: `/docs/design/Project_design.md#10.2`
- **セキュリティ設計**: `/docs/design/Security_design.md`

---

## 概要
システム統合、デプロイメント、運用環境構築に関するタスク一覧。開発環境から本番環境への移行と運用開始準備を管理する。

---

## 優先度凡例
- 🔴 Critical: システム稼働に必須のタスク
- 🟡 High: 品質保証に重要なタスク
- 🟢 Medium: 運用効率化のタスク
- ⚪ Low: 改善・最適化タスク

---

## 1. 開発環境整備 【優先度: 🔴】

### 1.1 ローカル開発環境
```bash
- [ ] プロジェクト構造の作成
    mkdir -p bid-judge-system/{server,client}/{src/{ts,js,gas},tests,config}
    mkdir -p bid-judge-system/{docs,scripts,.github/workflows}

- [ ] package.json の作成
    {
      "name": "bid-judge-system",
      "version": "1.0.0",
      "scripts": {
        "build": "npm run build:server && npm run build:client",
        "build:server": "tsc -p server/tsconfig.json",
        "build:client": "tsc -p client/tsconfig.json",
        "push": "npm run push:server && npm run push:client",
        "push:server": "cd server && clasp push",
        "push:client": "cd client && clasp push",
        "deploy": "npm run build && npm run push",
        "test": "jest",
        "lint": "eslint src/**/*.ts",
        "format": "prettier --write src/**/*.ts"
      }
    }

- [ ] TypeScript設定（tsconfig.json）
    {
      "compilerOptions": {
        "target": "ES5",
        "module": "none",
        "lib": ["ES2015"],
        "outDir": "./src/js",
        "rootDir": "./src/ts",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "types": ["google-apps-script"]
      }
    }

- [ ] ESLint設定（.eslintrc.json）
- [ ] Prettier設定（.prettierrc）
- [ ] Git設定（.gitignore）
```

### 1.2 clasp設定
```bash
- [ ] claspインストールとログイン
    npm install -g @google/clasp
    clasp login

- [ ] サーバー側プロジェクト作成
    cd server
    clasp create --type library --title "BidJudgmentLibrary"
    # .clasp.json にスクリプトIDを記録

- [ ] クライアント側プロジェクト作成
    cd client
    clasp create --type sheets --title "BidJudgmentClient"
    # .clasp.json にスクリプトIDを記録

- [ ] appsscript.json の設定
    {
      "timeZone": "Asia/Tokyo",
      "dependencies": {
        "libraries": [{
          "userSymbol": "BidJudgmentLib",
          "libraryId": "サーバー側のスクリプトID",
          "version": "1"
        }]
      },
      "exceptionLogging": "STACKDRIVER",
      "runtimeVersion": "V8"
    }
```

---

## 2. CI/CDパイプライン構築 【優先度: 🔴】

### 2.1 GitHub Actions設定

#### 2.1.1 テストワークフロー（.github/workflows/test.yml）
```yaml
- [ ] 自動テスト設定
    name: Test
    on:
      push:
        branches: [develop, main]
      pull_request:
        branches: [develop, main]
    
    jobs:
      test:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v3
          - uses: actions/setup-node@v3
            with:
              node-version: '18'
          - run: npm ci
          - run: npm run lint
          - run: npm run test
          - run: npm run build
```

#### 2.1.2 ビルドワークフロー（.github/workflows/build.yml）
```yaml
- [ ] ビルド検証設定
    name: Build
    on:
      push:
        branches: [develop]
    
    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v3
          - uses: actions/setup-node@v3
          - run: npm ci
          - run: npm run build
          - uses: actions/upload-artifact@v3
            with:
              name: build-artifacts
              path: |
                server/src/js/
                client/src/js/
```

#### 2.1.3 デプロイワークフロー（.github/workflows/deploy.yml）
```yaml
- [ ] 自動デプロイ設定
    name: Deploy
    on:
      push:
        branches: [main]
      workflow_dispatch:
    
    jobs:
      deploy-dev:
        if: github.ref == 'refs/heads/develop'
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v3
          - uses: actions/setup-node@v3
          - run: npm ci
          - run: npm run build
          - name: Deploy to Dev
            env:
              CLASP_CREDENTIALS: ${{ secrets.CLASP_CREDENTIALS_DEV }}
            run: |
              echo $CLASP_CREDENTIALS > ~/.clasprc.json
              npm run push
      
      deploy-prod:
        if: github.ref == 'refs/heads/main'
        runs-on: ubuntu-latest
        environment: production
        steps:
          - uses: actions/checkout@v3
          - uses: actions/setup-node@v3
          - run: npm ci
          - run: npm run build
          - name: Deploy to Production
            env:
              CLASP_CREDENTIALS: ${{ secrets.CLASP_CREDENTIALS_PROD }}
            run: |
              echo $CLASP_CREDENTIALS > ~/.clasprc.json
              npm run push
```

### 2.2 GitHub Secrets設定
- [ ] CLASP_CREDENTIALS_DEV の設定
  - [ ] 開発環境用の認証情報取得
  - [ ] GitHub Secretsに登録
- [ ] CLASP_CREDENTIALS_PROD の設定
  - [ ] 本番環境用の認証情報取得
  - [ ] GitHub Secretsに登録
- [ ] その他のシークレット
  - [ ] OCR_API_KEY
  - [ ] LLM_API_KEY
  - [ ] ALLOWED_DOMAINS

---

## 3. ビルドプロセス実装 【優先度: 🔴】

### 3.1 TypeScript → JavaScript変換
```javascript
- [ ] ビルドスクリプト（scripts/build.js）
    const fs = require('fs');
    const path = require('path');
    
    function convertToGAS(jsFile) {
      let content = fs.readFileSync(jsFile, 'utf8');
      
      // import/export文の除去
      content = content.replace(/^import .* from .*;$/gm, '');
      content = content.replace(/^export /gm, '');
      
      // 名前空間への変換
      content = wrapInNamespace(content);
      
      // GASファイルとして保存
      const gasFile = jsFile.replace('/js/', '/gas/').replace('.js', '.gs');
      fs.writeFileSync(gasFile, content);
    }

- [ ] 名前空間ラッパー
    function wrapInNamespace(content) {
      // ファイル名から名前空間を決定
      // namespace で囲む
    }

- [ ] ビルド実行スクリプト
    npm run build:ts    # TypeScript → JavaScript
    npm run build:gas   # JavaScript → GAS
    npm run build:all   # 全体ビルド
```

### 3.2 ファイル結合処理
```javascript
- [ ] 依存関係の解決
    function resolveDependencies(files) {
      // ファイル間の依存関係を解析
      // 適切な順序で結合
    }

- [ ] GAS用ファイル生成
    function generateGASBundle(files) {
      // 複数ファイルを1つに結合
      // グローバル関数の露出
    }
```

---

## 4. テスト環境構築 【優先度: 🟡】

### 4.1 テストフレームワーク設定
```javascript
- [ ] Jest設定（jest.config.js）
    module.exports = {
      preset: 'ts-jest',
      testEnvironment: 'node',
      roots: ['<rootDir>/src/ts', '<rootDir>/tests'],
      testMatch: ['**/__tests__/**/*.ts', '**/*.test.ts'],
      collectCoverageFrom: [
        'src/ts/**/*.ts',
        '!src/ts/**/*.d.ts'
      ],
      coverageThreshold: {
        global: {
          branches: 80,
          functions: 80,
          lines: 80,
          statements: 80
        }
      }
    };

- [ ] GASモック作成
    // __mocks__/google-apps-script.ts
    global.SpreadsheetApp = {
      getActiveSpreadsheet: jest.fn(),
      openById: jest.fn(),
      // ... その他のモック
    };

- [ ] テストユーティリティ
    function createMockSheet(data: any[][]) {
      return {
        getDataRange: () => ({
          getValues: () => data
        })
      };
    }
```

### 4.2 E2Eテスト環境
```javascript
- [ ] テスト用スプレッドシート準備
    function setupTestEnvironment() {
      // テスト用マスターデータ作成
      // テスト用クライアントシート作成
      // 権限設定
    }

- [ ] テストシナリオ実装
    describe('E2E: 判定フロー', () => {
      test('100件の一括判定', async () => {
        // テストデータ投入
        // 判定実行
        // 結果検証
      });
    });

- [ ] パフォーマンステスト
    function performanceTest() {
      // 処理時間測定
      // メモリ使用量確認
      // API呼び出し回数
    }
```

---

## 5. 環境別設定管理 【優先度: 🟡】

### 5.1 環境設定ファイル
```javascript
- [ ] 開発環境設定（config/dev.json）
    {
      "environment": "development",
      "masterSpreadsheetId": "dev-master-sheet-id",
      "apiEndpoints": {
        "ocr": "https://dev-ocr-api.example.com",
        "llm": "https://dev-llm-api.example.com"
      },
      "logging": {
        "level": "debug",
        "destination": "console"
      }
    }

- [ ] ステージング環境設定（config/staging.json）
- [ ] 本番環境設定（config/prod.json）

- [ ] 環境変数管理
    function getConfig() {
      const env = PropertiesService.getScriptProperties()
        .getProperty('ENVIRONMENT') || 'development';
      return configs[env];
    }
```

### 5.2 シークレット管理
```javascript
- [ ] PropertiesService設定スクリプト
    function setSecrets(env: string) {
      const properties = PropertiesService.getScriptProperties();
      const secrets = getSecretsForEnv(env);
      
      Object.entries(secrets).forEach(([key, value]) => {
        properties.setProperty(key, value);
      });
    }

- [ ] シークレットローテーション
    function rotateAPIKeys() {
      // 新しいキーの生成
      // 段階的な切り替え
      // 古いキーの無効化
    }
```

---

## 6. デプロイメント手順 【優先度: 🔴】

### 6.1 サーバー側デプロイ
```bash
- [ ] ライブラリのデプロイ手順
    1. コードレビュー完了確認
    2. テスト全合格確認
    3. ビルド実行
       npm run build:server
    4. GASへプッシュ
       cd server && clasp push
    5. バージョン作成
       clasp deploy --description "v1.0.0: Initial release"
    6. バージョン番号記録

- [ ] ライブラリ公開設定
    // GAS エディタで実行
    1. ライブラリとして公開
    2. アクセス権限設定（全員が閲覧可能）
    3. スクリプトIDの記録
```

### 6.2 クライアント側デプロイ
```bash
- [ ] クライアントのデプロイ手順
    1. ライブラリ参照の更新
    2. ビルド実行
       npm run build:client
    3. GASへプッシュ
       cd client && clasp push
    4. 動作確認
    5. ユーザーへの通知

- [ ] スプレッドシート設定
    1. マスターシートIDの設定
    2. カスタムメニューの確認
    3. 権限設定の確認
```

---

## 7. データ移行 【優先度: 🟡】

### 7.1 マスターデータ移行
```javascript
- [ ] 移行スクリプト作成
    function migrateToProduction() {
      // 開発環境からのエクスポート
      // データ検証
      // 本番環境へのインポート
      // 整合性確認
    }

- [ ] データ検証
    function validateMigrationData() {
      // レコード数の確認
      // 必須項目の確認
      // 参照整合性の確認
    }

- [ ] ロールバック手順
    function rollbackMigration() {
      // バックアップからの復元
      // 状態確認
      // 通知
    }
```

### 7.2 設定データ移行
- [ ] ユーザー設定の移行
- [ ] 権限設定の移行
- [ ] カスタム設定の移行

---

## 8. 監視・運用設定 【優先度: 🟢】

### 8.1 監視設定
```javascript
- [ ] エラー監視
    function setupErrorMonitoring() {
      // Stackdriverログ設定
      // エラー通知設定
      // アラート閾値設定
    }

- [ ] パフォーマンス監視
    function setupPerformanceMonitoring() {
      // 実行時間の記録
      // API使用量の追跡
      // リソース使用状況
    }

- [ ] 可用性監視
    function setupAvailabilityMonitoring() {
      // 定期的なヘルスチェック
      // 応答時間の監視
      // ダウンタイム通知
    }
```

### 8.2 ログ管理
```javascript
- [ ] ログ収集設定
    function setupLogging() {
      // ログレベル設定
      // ログローテーション
      // 長期保存設定
    }

- [ ] ログ分析
    function analyzeLogPatterns() {
      // エラーパターン分析
      // 使用状況分析
      // 異常検知
    }
```

---

## 9. 運用ドキュメント作成 【優先度: 🟢】

### 9.1 運用マニュアル
- [ ] システム構成図の作成
- [ ] 起動・停止手順書
- [ ] 定期メンテナンス手順書
- [ ] 障害対応手順書
- [ ] バックアップ・リストア手順書

### 9.2 ユーザーマニュアル
- [ ] インストール手順書
- [ ] 基本操作ガイド
- [ ] トラブルシューティング
- [ ] FAQ作成
- [ ] 動画マニュアル作成

### 9.3 開発者向けドキュメント
- [ ] アーキテクチャドキュメント
- [ ] API仕様書（JSDoc生成）
- [ ] デプロイメントガイド
- [ ] コントリビューションガイド

---

## 10. 本番リリース準備 【優先度: 🔴】

### 10.1 リリース前チェックリスト
- [ ] 全機能の動作確認
  - [ ] 判定処理の正常動作
  - [ ] タイムアウト処理の確認
  - [ ] エラーハンドリングの確認
- [ ] セキュリティ監査
  - [ ] アクセス権限の確認
  - [ ] APIキーの保護確認
  - [ ] ログの適切性確認
- [ ] パフォーマンステスト
  - [ ] 100件バッチ処理
  - [ ] 同時アクセステスト
  - [ ] リソース使用量確認
- [ ] ドキュメント完成度
  - [ ] 運用マニュアル
  - [ ] ユーザーガイド
  - [ ] API仕様書

### 10.2 リリース作業
```bash
- [ ] リリース手順
    1. リリースブランチ作成
       git checkout -b release/v1.0.0
    2. バージョン番号更新
    3. リリースノート作成
    4. 最終テスト実行
    5. mainブランチへマージ
    6. タグ作成
       git tag -a v1.0.0 -m "Release version 1.0.0"
    7. 本番デプロイ実行
    8. 動作確認
    9. ユーザー通知

- [ ] ロールバック計画
    1. 前バージョンの保持
    2. 切り戻し手順の文書化
    3. データバックアップ
```

### 10.3 リリース後対応
- [ ] 初期サポート体制
  - [ ] サポート窓口設置
  - [ ] 問い合わせ対応フロー
  - [ ] エスカレーション手順
- [ ] フィードバック収集
  - [ ] ユーザーアンケート
  - [ ] 使用状況分析
  - [ ] 改善要望収集
- [ ] 次期バージョン計画
  - [ ] 機能追加リスト
  - [ ] 改善項目リスト
  - [ ] ロードマップ作成

---

## 完了基準
- [ ] CI/CDパイプラインの完全自動化
- [ ] 全環境でのテスト合格
- [ ] セキュリティ監査の合格
- [ ] 運用ドキュメントの完成
- [ ] 本番環境での安定稼働（1週間）

---

## 改訂履歴
- 2025-07-23: 初版作成