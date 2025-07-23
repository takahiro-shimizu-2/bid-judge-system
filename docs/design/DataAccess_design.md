# 入札公告要件判定システム - データアクセス層設計書

## 関連ドキュメント
- **全体設計書**: `/docs/design/Project_design.md`
- **要件定義書**: `/docs/requirements/要件定義書.md`
- **マスター項目一覧**: `/docs/requirements/appendix/別紙1：マスター項目一覧.md`
- **サーバー側TODO**: `/docs/todo/Server_TODO.md`

---

## 1. 概要

### 1.1 目的
マスターデータへの効率的かつ安全なアクセスを提供し、データの整合性を保ちながら高速な判定処理を実現する。

### 1.2 設計方針
- **キャッシュファースト**: 頻繁にアクセスされるデータはキャッシュを活用
- **読み取り専用**: マスターデータは読み取り専用として扱う
- **エラーハンドリング**: データアクセス失敗時の適切な処理
- **型安全性**: TypeScriptの型システムを活用したデータ構造の保証

---

## 2. アーキテクチャ

### 2.1 レイヤー構成
```
┌─────────────────────────────────────────────────────────────┐
│                    ビジネスロジック層                         │
│              （判定エンジン、ワークフロー制御）               │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                   データアクセス層（本層）                    │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │ Repository  │  │   Cache     │  │   Query     │      │
│  │  Pattern    │  │  Manager    │  │  Builder    │      │
│  └─────────────┘  └─────────────┘  └─────────────┘      │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                     Google Sheets API                        │
│                  （マスタースプレッドシート）                 │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 主要コンポーネント

#### 2.2.1 Repository Pattern
各マスターデータに対応するリポジトリクラスを実装
- CompanyRepository: 企業マスターアクセス
- OfficeRepository: 拠点マスターアクセス
- EmployeeRepository: 従業員マスターアクセス
- ExperienceRepository: 実績マスターアクセス

#### 2.2.2 Cache Manager
Google Apps ScriptのCacheServiceを活用した効率的なキャッシュ管理
- スクリプトキャッシュ: 6時間（最大）
- ドキュメントキャッシュ: 必要に応じて使用

#### 2.2.3 Query Builder
複雑な検索条件を構築するためのクエリビルダー

---

## 3. データアクセスパターン

### 3.1 基本的なアクセスパターン

#### 3.1.1 単一レコード取得
```typescript
namespace DataAccess {
  export class CompanyRepository {
    private cache: CacheManager;
    private sheet: GoogleAppsScript.Spreadsheet.Sheet;
    
    constructor() {
      this.cache = new CacheManager();
      this.sheet = this.getMasterSheet();
    }
    
    /**
     * 企業IDで企業情報を取得
     * @param companyId 企業ID
     * @returns 企業情報オブジェクト
     */
    public getById(companyId: string): Company | null {
      // 1. キャッシュチェック
      const cacheKey = `company_${companyId}`;
      const cached = this.cache.get<Company>(cacheKey);
      if (cached) return cached;
      
      // 2. データ取得
      const data = this.findByColumn('A', companyId);
      if (!data) return null;
      
      // 3. オブジェクト変換
      const company = this.mapToEntity(data);
      
      // 4. キャッシュ保存
      this.cache.set(cacheKey, company, 3600); // 1時間
      
      return company;
    }
  }
}
```

#### 3.1.2 複数レコード取得
```typescript
namespace DataAccess {
  export class OfficeRepository {
    /**
     * 企業IDで拠点一覧を取得
     * @param companyId 企業ID
     * @returns 拠点情報配列
     */
    public getByCompanyId(companyId: string): Office[] {
      const cacheKey = `offices_company_${companyId}`;
      const cached = this.cache.get<Office[]>(cacheKey);
      if (cached) return cached;
      
      // フィルタリング処理
      const allData = this.sheet.getDataRange().getValues();
      const offices = allData
        .filter(row => row[1] === companyId) // B列が企業ID
        .map(row => this.mapToEntity(row));
      
      this.cache.set(cacheKey, offices, 3600);
      return offices;
    }
  }
}
```

### 3.2 高度なアクセスパターン

#### 3.2.1 条件付き検索
```typescript
namespace DataAccess {
  export interface SearchCriteria {
    region?: string;
    constructionType?: string;
    minScore?: number;
  }
  
  export class ExperienceRepository {
    /**
     * 条件に合致する実績を検索
     * @param criteria 検索条件
     * @returns 実績情報配列
     */
    public search(criteria: SearchCriteria): Experience[] {
      const query = new QueryBuilder()
        .where('region', '=', criteria.region)
        .where('constructionType', '=', criteria.constructionType)
        .where('score', '>=', criteria.minScore)
        .build();
      
      return this.executeQuery(query);
    }
  }
}
```

#### 3.2.2 バッチ取得
```typescript
namespace DataAccess {
  export class BatchDataAccess {
    /**
     * 複数IDで一括取得（効率化）
     * @param ids ID配列
     * @returns エンティティのMap
     */
    public getBatch<T>(ids: string[]): Map<string, T> {
      const result = new Map<string, T>();
      
      // キャッシュから取得可能なものを先に取得
      const uncachedIds: string[] = [];
      ids.forEach(id => {
        const cached = this.cache.get<T>(`entity_${id}`);
        if (cached) {
          result.set(id, cached);
        } else {
          uncachedIds.push(id);
        }
      });
      
      // 未キャッシュ分をバッチ取得
      if (uncachedIds.length > 0) {
        const batchData = this.fetchBatch(uncachedIds);
        batchData.forEach((value, key) => {
          result.set(key, value);
          this.cache.set(`entity_${key}`, value, 3600);
        });
      }
      
      return result;
    }
  }
}
```

---

## 4. キャッシュ戦略

### 4.1 キャッシュレベル

#### 4.1.1 L1キャッシュ（メモリ）
- 実行中のスクリプト内でのみ有効
- 最も高速だが容量制限あり
- 用途: 同一処理内での重複アクセス防止

#### 4.1.2 L2キャッシュ（CacheService）
- スクリプト実行間で共有
- 最大6時間保持
- 用途: 頻繁にアクセスされるマスターデータ

### 4.2 キャッシュ管理
```typescript
namespace DataAccess {
  export class CacheManager {
    private scriptCache: GoogleAppsScript.Cache.Cache;
    private memoryCache: Map<string, any>;
    
    constructor() {
      this.scriptCache = CacheService.getScriptCache();
      this.memoryCache = new Map();
    }
    
    /**
     * キャッシュ取得（L1→L2の順で確認）
     */
    public get<T>(key: string): T | null {
      // L1キャッシュチェック
      if (this.memoryCache.has(key)) {
        return this.memoryCache.get(key) as T;
      }
      
      // L2キャッシュチェック
      const cached = this.scriptCache.get(key);
      if (cached) {
        const parsed = JSON.parse(cached) as T;
        this.memoryCache.set(key, parsed); // L1に昇格
        return parsed;
      }
      
      return null;
    }
    
    /**
     * キャッシュ保存（L1とL2の両方に保存）
     */
    public set<T>(key: string, value: T, ttl: number): void {
      this.memoryCache.set(key, value);
      this.scriptCache.put(key, JSON.stringify(value), ttl);
    }
    
    /**
     * キャッシュ無効化
     */
    public invalidate(pattern?: string): void {
      if (pattern) {
        // パターンマッチでの削除
        const keys = Array.from(this.memoryCache.keys());
        keys.forEach(key => {
          if (key.includes(pattern)) {
            this.memoryCache.delete(key);
            this.scriptCache.remove(key);
          }
        });
      } else {
        // 全削除
        this.memoryCache.clear();
        // Script Cacheは個別削除のみ対応
      }
    }
  }
}
```

### 4.3 キャッシュ更新戦略
- **TTL（Time To Live）**: 1時間を基本とする
- **イベントベース更新**: マスター更新時にキャッシュクリア
- **サイズ制限対応**: 大きなデータは分割してキャッシュ

---

## 5. エラーハンドリング

### 5.1 エラー種別と対処

#### 5.1.1 接続エラー
```typescript
namespace DataAccess {
  export class DataAccessError extends Error {
    constructor(
      public code: string,
      message: string,
      public retriable: boolean = false
    ) {
      super(message);
    }
  }
  
  export class BaseRepository {
    protected async executeWithRetry<T>(
      operation: () => T,
      maxRetries: number = 3
    ): Promise<T> {
      let lastError: Error;
      
      for (let i = 0; i < maxRetries; i++) {
        try {
          return operation();
        } catch (error) {
          lastError = error;
          
          // リトライ可能なエラーかチェック
          if (this.isRetriableError(error) && i < maxRetries - 1) {
            // 指数バックオフ
            Utilities.sleep(Math.pow(2, i) * 1000);
            continue;
          }
          
          throw error;
        }
      }
      
      throw new DataAccessError(
        'CONNECTION_ERROR',
        `Failed after ${maxRetries} attempts: ${lastError.message}`,
        true
      );
    }
    
    private isRetriableError(error: any): boolean {
      // ネットワークエラー、タイムアウトなどはリトライ可能
      return error.message?.includes('timeout') ||
             error.message?.includes('Service unavailable');
    }
  }
}
```

#### 5.1.2 データ不整合エラー
```typescript
namespace DataAccess {
  export class DataIntegrityValidator {
    /**
     * データ整合性チェック
     */
    public validate(data: any[]): ValidationResult {
      const errors: ValidationError[] = [];
      
      // 必須項目チェック
      if (!data[0]) {
        errors.push({
          field: 'id',
          message: 'ID is required',
          severity: 'error'
        });
      }
      
      // 参照整合性チェック
      if (data[1] && !this.isValidReference(data[1])) {
        errors.push({
          field: 'companyId',
          message: 'Invalid company reference',
          severity: 'warning'
        });
      }
      
      return {
        valid: errors.filter(e => e.severity === 'error').length === 0,
        errors
      };
    }
  }
}
```

### 5.2 エラー処理フロー
1. **即座にリトライ可能**: ネットワークエラー等
2. **遅延リトライ**: API制限等
3. **代替処理**: キャッシュからの取得
4. **エラー伝播**: ビジネスロジック層へ

---

## 6. パフォーマンス最適化

### 6.1 バッチ処理
- 複数レコードの一括取得
- getValues()の呼び出し回数最小化
- 範囲指定による部分読み込み

### 6.2 インデックス活用
```typescript
namespace DataAccess {
  export class IndexManager {
    private indices: Map<string, Map<string, number>>;
    
    /**
     * インデックス構築
     */
    public buildIndex(sheetData: any[][], keyColumn: number): void {
      const index = new Map<string, number>();
      
      sheetData.forEach((row, rowIndex) => {
        const key = row[keyColumn];
        if (key) {
          index.set(String(key), rowIndex);
        }
      });
      
      this.indices.set(`column_${keyColumn}`, index);
    }
    
    /**
     * インデックスを使用した高速検索
     */
    public findByIndex(columnIndex: number, value: string): number | null {
      const index = this.indices.get(`column_${columnIndex}`);
      return index?.get(value) ?? null;
    }
  }
}
```

### 6.3 遅延読み込み
- 必要なカラムのみ取得
- ページネーション対応
- プログレッシブレンダリング

---

## 7. セキュリティ考慮事項

### 7.1 アクセス制御
- マスターデータは読み取り専用
- スプレッドシートの共有設定で制御
- ライブラリレベルでの書き込み禁止

### 7.2 データマスキング
```typescript
namespace DataAccess {
  export class DataMasking {
    /**
     * 機密情報のマスキング
     */
    public mask(data: any, fields: string[]): any {
      const masked = { ...data };
      
      fields.forEach(field => {
        if (masked[field]) {
          masked[field] = this.maskValue(masked[field]);
        }
      });
      
      return masked;
    }
    
    private maskValue(value: string): string {
      if (value.length <= 4) return '****';
      return value.substring(0, 2) + '****' + value.substring(value.length - 2);
    }
  }
}
```

### 7.3 監査ログ
- アクセスログの記録
- 異常アクセスの検知
- パフォーマンスメトリクス

---

## 8. テスト戦略

### 8.1 単体テスト
- 各リポジトリメソッドのテスト
- キャッシュ動作のテスト
- エラーハンドリングのテスト

### 8.2 統合テスト
- 実際のスプレッドシートとの連携テスト
- 大量データでのパフォーマンステスト
- 同時アクセステスト

### 8.3 モックデータ
```typescript
namespace DataAccess {
  export class MockRepository implements IRepository {
    private mockData: Map<string, any>;
    
    constructor() {
      this.mockData = new Map([
        ['C00001', { id: 'C00001', name: 'テスト企業1', bankruptcy_flg: false }],
        ['C00002', { id: 'C00002', name: 'テスト企業2', bankruptcy_flg: true }]
      ]);
    }
    
    public getById(id: string): any {
      return this.mockData.get(id) || null;
    }
  }
}
```

---

## 9. 実装チェックリスト

### 9.1 必須実装項目
- [ ] BaseRepository抽象クラス
- [ ] 各エンティティのRepository実装
- [ ] CacheManager実装
- [ ] QueryBuilder実装
- [ ] エラーハンドリング機構
- [ ] データ検証機能
- [ ] ログ出力機能

### 9.2 オプション実装項目
- [ ] インデックス管理
- [ ] バッチ最適化
- [ ] 監査ログ
- [ ] パフォーマンスモニタリング

---

## 改訂履歴
- 2025-07-23: 初版作成