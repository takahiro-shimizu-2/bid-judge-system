# 入札公告要件判定システム - API設計書

## 関連ドキュメント
- **全体設計書**: `/docs/design/Project_design.md`
- **判定エンジン設計**: `/docs/design/Judgment_design.md`
- **要件定義書**: `/docs/requirements/要件定義書.md`
- **サーバー側TODO**: `/docs/todo/Server_TODO.md`

---

## 1. 概要

### 1.1 目的
外部OCR/LLMサービスとの連携、およびGASライブラリとして公開するAPIの仕様を定義し、安定した拡張可能なインターフェースを提供する。

### 1.2 API構成
1. **外部API連携**: OCRサービス、LLMサービスとの通信
2. **ライブラリ公開API**: クライアント側から利用可能な判定機能
3. **内部API**: モジュール間の通信インターフェース

### 1.3 設計方針
- **RESTful原則**: 標準的なHTTPメソッドとステータスコード
- **型安全性**: TypeScriptによる厳密な型定義
- **エラーハンドリング**: 一貫したエラーレスポンス形式
- **拡張性**: バージョニングとの後方互換性

---

## 2. 外部API連携

### 2.1 OCR API

#### 2.1.1 エンドポイント仕様
```typescript
namespace ExternalAPI {
  export interface OCRConfig {
    endpoint: string;        // OCRサービスのエンドポイント
    apiKey: string;         // 認証キー
    timeout: number;        // タイムアウト時間（ミリ秒）
    maxRetries: number;     // 最大リトライ回数
  }
  
  export interface OCRRequest {
    documentUrl: string;    // PDFのURL
    options?: {
      language?: 'ja' | 'en';
      outputFormat?: 'text' | 'json';
      pageRange?: string;   // "1-5" or "1,3,5"
    };
  }
  
  export interface OCRResponse {
    requestId: string;      // リクエストID
    status: 'pending' | 'processing' | 'completed' | 'failed';
    result?: {
      text: string;         // 抽出されたテキスト
      pages: number;        // 総ページ数
      confidence: number;   // 信頼度スコア（0-1）
      metadata?: {
        processingTime: number;
        characterCount: number;
      };
    };
    error?: {
      code: string;
      message: string;
    };
  }
}
```

#### 2.1.2 OCR処理フロー
```typescript
namespace ExternalAPI {
  export class OCRConnector {
    private config: OCRConfig;
    
    constructor(config: OCRConfig) {
      this.config = config;
    }
    
    /**
     * OCR処理を開始
     */
    async processDocument(request: OCRRequest): Promise<string> {
      // 1. 処理開始リクエスト
      const response = await this.submitOCRRequest(request);
      const requestId = response.requestId;
      
      // 2. ポーリングで結果待機
      const result = await this.pollForResult(requestId);
      
      // 3. 結果の後処理
      return this.postProcessResult(result);
    }
    
    /**
     * OCRリクエスト送信
     */
    private async submitOCRRequest(request: OCRRequest): Promise<any> {
      const options: GoogleAppsScript.URL_Fetch.URLFetchRequestOptions = {
        method: 'post',
        headers: {
          'Authorization': `Bearer ${this.config.apiKey}`,
          'Content-Type': 'application/json'
        },
        payload: JSON.stringify({
          url: request.documentUrl,
          ...request.options
        }),
        muteHttpExceptions: true
      };
      
      const response = UrlFetchApp.fetch(this.config.endpoint, options);
      return this.handleResponse(response);
    }
    
    /**
     * 結果のポーリング
     */
    private async pollForResult(requestId: string): Promise<OCRResponse> {
      const maxAttempts = 30;  // 最大30回（5分）
      const interval = 10000;  // 10秒間隔
      
      for (let i = 0; i < maxAttempts; i++) {
        const result = await this.checkStatus(requestId);
        
        if (result.status === 'completed' || result.status === 'failed') {
          return result;
        }
        
        Utilities.sleep(interval);
      }
      
      throw new Error('OCR processing timeout');
    }
  }
}
```

### 2.2 LLM API

#### 2.2.1 エンドポイント仕様
```typescript
namespace ExternalAPI {
  export interface LLMConfig {
    endpoint: string;
    apiKey: string;
    model: string;          // 使用するモデル名
    temperature: number;    // 0.0-1.0
    maxTokens: number;
  }
  
  export interface LLMRequest {
    prompt: string;         // プロンプト
    context?: string;       // 追加コンテキスト
    examples?: Array<{      // Few-shot examples
      input: string;
      output: string;
    }>;
  }
  
  export interface LLMResponse {
    choices: Array<{
      text: string;
      finishReason: string;
    }>;
    usage: {
      promptTokens: number;
      completionTokens: number;
      totalTokens: number;
    };
  }
}
```

#### 2.2.2 要件抽出プロンプト
```typescript
namespace ExternalAPI {
  export class RequirementExtractor {
    private static readonly EXTRACTION_PROMPT = `
以下の入札公告文から、判定に必要な要件を抽出してください。

【抽出する要件】
1. 欠格要件: 破産、暴力団関係、指名停止など
2. 等級要件: 必要な等級、工事種別
3. 地域要件: 本店・支店の所在地要件
4. 実績要件: 必要な工事実績、金額、期間
5. 技術者要件: 必要な資格、人数
6. JV要件: 共同企業体の条件

【出力形式】
以下のJSON形式で出力してください：
{
  "disqualification": {
    "requirements": ["要件1", "要件2"]
  },
  "grade": {
    "constructionType": "工事種別",
    "requiredGrade": "等級",
    "condition": "以上" | "同等"
  },
  "region": {
    "requirement": "地域名",
    "type": "本店" | "支店" | "営業所"
  },
  "achievement": {
    "amount": "金額",
    "period": "期間",
    "constructionType": "工事種別"
  },
  "engineer": {
    "qualification": ["資格1", "資格2"],
    "count": 人数
  },
  "jv": {
    "allowed": true | false,
    "conditions": ["条件1", "条件2"]
  }
}

【入札公告文】
`;
    
    /**
     * 要件抽出実行
     */
    async extractRequirements(ocrText: string): Promise<RequirementSet> {
      const request: LLMRequest = {
        prompt: RequirementExtractor.EXTRACTION_PROMPT + ocrText,
        temperature: 0.1,  // 低温度で一貫性を重視
        examples: this.getFewShotExamples()
      };
      
      const response = await this.llmConnector.complete(request);
      return this.parseResponse(response);
    }
    
    /**
     * Few-shot examples
     */
    private getFewShotExamples(): Array<{input: string; output: string}> {
      return [
        {
          input: "入札参加資格：令和5・6年度において、土木一式工事でA等級に格付けされている者",
          output: JSON.stringify({
            grade: {
              constructionType: "土木一式工事",
              requiredGrade: "A",
              condition: "同等"
            }
          })
        }
      ];
    }
  }
}
```

---

## 3. ライブラリ公開API

### 3.1 公開関数一覧

#### 3.1.1 判定実行API
```typescript
/**
 * BidJudgmentLibrary - 入札判定ライブラリ
 * @namespace BidJudgmentLibrary
 */

/**
 * 企業の入札参加可否を判定
 * @param {string} companyId - 企業ID
 * @param {Object} requirements - 要件オブジェクト
 * @returns {JudgmentResult} 判定結果
 */
function executeJudgment(companyId: string, requirements: RequirementSet): JudgmentResult {
  try {
    // 入力検証
    validateInput(companyId, requirements);
    
    // 判定エンジン実行
    const engine = new JudgmentEngine();
    const result = engine.execute(companyId, requirements);
    
    // 結果を返却
    return {
      success: true,
      data: result,
      error: null
    };
  } catch (error) {
    return {
      success: false,
      data: null,
      error: {
        code: 'JUDGMENT_ERROR',
        message: error.message
      }
    };
  }
}

/**
 * 複数企業の一括判定
 * @param {string[]} companyIds - 企業ID配列
 * @param {Object} requirements - 要件オブジェクト
 * @returns {BatchJudgmentResult} 一括判定結果
 */
function executeBatchJudgment(
  companyIds: string[], 
  requirements: RequirementSet
): BatchJudgmentResult {
  const results = new Map<string, JudgmentResult>();
  
  // バッチ処理の最適化
  const batchProcessor = new BatchProcessor();
  
  companyIds.forEach(companyId => {
    try {
      const result = executeJudgment(companyId, requirements);
      results.set(companyId, result);
    } catch (error) {
      results.set(companyId, {
        success: false,
        error: error.message
      });
    }
  });
  
  return {
    total: companyIds.length,
    success: Array.from(results.values()).filter(r => r.success).length,
    failed: Array.from(results.values()).filter(r => !r.success).length,
    results: Object.fromEntries(results)
  };
}
```

#### 3.1.2 データ取得API
```typescript
/**
 * 企業情報を取得
 * @param {string} companyId - 企業ID
 * @returns {Company} 企業情報
 */
function getCompanyData(companyId: string): Company {
  const repository = new CompanyRepository();
  const company = repository.getById(companyId);
  
  if (!company) {
    throw new Error(`Company not found: ${companyId}`);
  }
  
  // 機密情報のマスキング
  return maskSensitiveData(company);
}

/**
 * 判定結果の詳細を取得
 * @param {string} judgmentId - 判定ID
 * @returns {JudgmentDetail} 判定詳細
 */
function getJudgmentDetail(judgmentId: string): JudgmentDetail {
  const repository = new JudgmentRepository();
  return repository.getDetail(judgmentId);
}

/**
 * 利用可能な工事種別一覧を取得
 * @returns {ConstructionType[]} 工事種別一覧
 */
function getConstructionTypes(): ConstructionType[] {
  const repository = new MasterDataRepository();
  return repository.getConstructionTypes();
}
```

### 3.2 型定義

#### 3.2.1 リクエスト/レスポンス型
```typescript
/**
 * 要件セット
 */
interface RequirementSet {
  disqualification?: DisqualificationRequirement;
  grade?: GradeRequirement;
  region?: RegionRequirement;
  achievement?: AchievementRequirement;
  engineer?: EngineerRequirement;
  jv?: JVRequirement;
}

/**
 * 判定結果
 */
interface JudgmentResult {
  success: boolean;
  data?: {
    judgmentId: string;
    companyId: string;
    overallResult: 'qualified' | 'disqualified' | 'conditional';
    details: {
      [key: string]: {
        result: boolean;
        reason?: string;
        score?: number;
      };
    };
    suggestions?: string[];
    timestamp: Date;
  };
  error?: {
    code: string;
    message: string;
    details?: any;
  };
}

/**
 * 一括判定結果
 */
interface BatchJudgmentResult {
  total: number;
  success: number;
  failed: number;
  results: {
    [companyId: string]: JudgmentResult;
  };
  processingTime?: number;
}
```

### 3.3 エラーコード体系

#### 3.3.1 エラーコード一覧
| コード | 説明 | HTTPステータス相当 |
|--------|------|-------------------|
| INVALID_INPUT | 入力値が不正 | 400 |
| COMPANY_NOT_FOUND | 企業が見つからない | 404 |
| JUDGMENT_ERROR | 判定処理エラー | 500 |
| TIMEOUT_ERROR | タイムアウト | 504 |
| RATE_LIMIT_EXCEEDED | レート制限超過 | 429 |
| UNAUTHORIZED | 認証エラー | 401 |
| FORBIDDEN | 権限不足 | 403 |

#### 3.3.2 エラーレスポンス形式
```typescript
interface ErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
    details?: {
      field?: string;
      value?: any;
      suggestion?: string;
    };
    timestamp: Date;
    requestId?: string;
  };
}
```

---

## 4. 内部API

### 4.1 モジュール間インターフェース

#### 4.1.1 判定エンジンインターフェース
```typescript
interface IJudgmentEngine {
  execute(companyId: string, requirements: RequirementSet): JudgmentResult;
  validateRequirements(requirements: RequirementSet): ValidationResult;
}

interface IJudge {
  judge(context: JudgmentContext): JudgeResult;
  getName(): string;
  getPriority(): number;
}
```

#### 4.1.2 リポジトリインターフェース
```typescript
interface IRepository<T> {
  getById(id: string): T | null;
  getAll(): T[];
  find(criteria: any): T[];
  exists(id: string): boolean;
}

interface ICacheableRepository<T> extends IRepository<T> {
  invalidateCache(pattern?: string): void;
  preloadCache(ids: string[]): void;
}
```

---

## 5. 認証・認可

### 5.1 API認証

#### 5.1.1 ライブラリアクセス制御
```typescript
namespace Security {
  export class AccessControl {
    /**
     * ライブラリ実行権限チェック
     */
    static checkAccess(): void {
      const user = Session.getActiveUser().getEmail();
      const authorizedDomains = ['company.com'];
      
      const domain = user.split('@')[1];
      if (!authorizedDomains.includes(domain)) {
        throw new Error('Unauthorized access');
      }
    }
    
    /**
     * レート制限チェック
     */
    static checkRateLimit(userId: string): void {
      const cache = CacheService.getUserCache();
      const key = `rate_limit_${userId}`;
      const count = Number(cache.get(key) || 0);
      
      if (count >= 100) {  // 1時間あたり100回まで
        throw new Error('Rate limit exceeded');
      }
      
      cache.put(key, String(count + 1), 3600);
    }
  }
}
```

### 5.2 外部API認証

#### 5.2.1 APIキー管理
```typescript
namespace Security {
  export class APIKeyManager {
    /**
     * APIキーの安全な取得
     */
    static getAPIKey(service: string): string {
      const scriptProperties = PropertiesService.getScriptProperties();
      const key = scriptProperties.getProperty(`${service}_API_KEY`);
      
      if (!key) {
        throw new Error(`API key not found for service: ${service}`);
      }
      
      return key;
    }
    
    /**
     * APIキーのローテーション
     */
    static rotateAPIKey(service: string): void {
      // 実装: 新しいキーを生成し、古いキーを無効化
    }
  }
}
```

---

## 6. パフォーマンス最適化

### 6.1 バッチ処理

#### 6.1.1 並列リクエスト
```typescript
namespace Performance {
  export class ParallelProcessor {
    /**
     * 複数のAPIリクエストを並列実行
     */
    static async fetchAll(requests: URLFetchRequest[]): Promise<HTTPResponse[]> {
      // UrlFetchApp.fetchAll を使用した並列処理
      const responses = UrlFetchApp.fetchAll(requests);
      
      return responses.map((response, index) => {
        if (response.getResponseCode() !== 200) {
          throw new Error(`Request ${index} failed: ${response.getContentText()}`);
        }
        return JSON.parse(response.getContentText());
      });
    }
  }
}
```

### 6.2 キャッシュ戦略

#### 6.2.1 APIレスポンスキャッシュ
```typescript
namespace Performance {
  export class APICache {
    /**
     * APIレスポンスのキャッシュ
     */
    static cacheResponse(key: string, response: any, ttl: number = 3600): void {
      const cache = CacheService.getScriptCache();
      const serialized = JSON.stringify({
        data: response,
        timestamp: new Date().getTime(),
        ttl: ttl
      });
      
      // 大きなデータは分割して保存
      if (serialized.length > 100000) {  // 100KB以上
        this.cacheInChunks(key, serialized, ttl);
      } else {
        cache.put(key, serialized, ttl);
      }
    }
  }
}
```

---

## 7. モニタリング・ログ

### 7.1 APIメトリクス

#### 7.1.1 メトリクス収集
```typescript
namespace Monitoring {
  export class APIMetrics {
    /**
     * API呼び出しの記録
     */
    static recordAPICall(
      endpoint: string,
      method: string,
      responseTime: number,
      statusCode: number
    ): void {
      const metrics = {
        timestamp: new Date(),
        endpoint,
        method,
        responseTime,
        statusCode,
        userId: Session.getActiveUser().getEmail()
      };
      
      // ログシートに記録
      Logger.log('API_CALL', metrics);
      
      // 異常値の検知
      if (responseTime > 5000 || statusCode >= 500) {
        this.alertAbnormalMetrics(metrics);
      }
    }
  }
}
```

---

## 8. テスト

### 8.1 モックサーバー

#### 8.1.1 外部APIモック
```typescript
namespace Testing {
  export class MockOCRService {
    static responses = new Map([
      ['success.pdf', { text: 'サンプルテキスト', confidence: 0.95 }],
      ['error.pdf', { error: 'Processing failed' }],
      ['timeout.pdf', { delay: 10000 }]
    ]);
    
    static async process(url: string): Promise<any> {
      const filename = url.split('/').pop();
      const response = this.responses.get(filename);
      
      if (response?.delay) {
        Utilities.sleep(response.delay);
      }
      
      return response || { text: 'Default response' };
    }
  }
}
```

### 8.2 統合テスト

#### 8.2.1 E2Eテストシナリオ
```typescript
namespace Testing {
  export class E2ETest {
    /**
     * 判定フロー全体のテスト
     */
    static async testCompleteFlow(): Promise<void> {
      // 1. OCR処理
      const ocrResult = await MockOCRService.process('test.pdf');
      
      // 2. 要件抽出
      const requirements = await MockLLMService.extract(ocrResult.text);
      
      // 3. 判定実行
      const judgment = executeJudgment('C00001', requirements);
      
      // 4. 結果検証
      assert(judgment.success === true);
      assert(judgment.data.overallResult === 'qualified');
    }
  }
}
```

---

## 9. バージョニング

### 9.1 APIバージョン管理

#### 9.1.1 バージョン戦略
- メジャーバージョン: 後方互換性のない変更
- マイナーバージョン: 後方互換性のある機能追加
- パッチバージョン: バグ修正

#### 9.1.2 バージョン指定方法
```typescript
// ライブラリのバージョン指定
// スクリプトID: XXXXX
// バージョン: 23 (v2.3.0相当)

// 使用例
const result = BidJudgmentLibrary.executeJudgment(companyId, requirements);
```

---

## 10. 実装チェックリスト

### 10.1 外部API連携
- [ ] OCRConnector実装
- [ ] LLMConnector実装
- [ ] エラーハンドリング
- [ ] リトライロジック
- [ ] タイムアウト処理

### 10.2 ライブラリ公開API
- [ ] 公開関数の実装
- [ ] 型定義ファイル
- [ ] ドキュメント生成
- [ ] サンプルコード

### 10.3 セキュリティ
- [ ] 認証機能
- [ ] レート制限
- [ ] APIキー管理
- [ ] アクセスログ

---

## 改訂履歴
- 2025-07-23: 初版作成