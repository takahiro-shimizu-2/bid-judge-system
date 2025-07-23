# 入札公告要件判定システム - セキュリティ設計書

## 関連ドキュメント
- **全体設計書**: `/docs/design/Project_design.md`
- **API設計書**: `/docs/design/API_design.md`
- **データアクセス設計**: `/docs/design/DataAccess_design.md`
- **要件定義書**: `/docs/requirements/要件定義書.md`

---

## 1. 概要

### 1.1 目的
入札公告要件判定システムにおける情報セキュリティを確保し、機密情報の保護、不正アクセスの防止、システムの完全性を維持する。

### 1.2 セキュリティ目標
1. **機密性（Confidentiality）**: 企業情報・判定結果の適切な保護
2. **完全性（Integrity）**: データの改ざん防止
3. **可用性（Availability）**: 正当な利用者への安定したサービス提供
4. **追跡可能性（Traceability）**: 全操作の監査証跡

### 1.3 対象範囲
- Google Apps Script実行環境
- Google Sheetsデータストア
- 外部API連携
- ユーザーアクセス管理

---

## 2. 脅威分析とリスク評価

### 2.1 想定される脅威

#### 2.1.1 外部からの脅威
| 脅威 | リスクレベル | 対策 |
|------|-------------|------|
| 不正アクセス | 高 | Google認証、アクセス制御リスト |
| データ漏洩 | 高 | 暗号化、アクセス権限管理 |
| DoS攻撃 | 中 | レート制限、リソース管理 |
| インジェクション攻撃 | 中 | 入力検証、パラメータ化 |

#### 2.1.2 内部からの脅威
| 脅威 | リスクレベル | 対策 |
|------|-------------|------|
| 権限の悪用 | 中 | 最小権限の原則、監査ログ |
| 誤操作 | 中 | 操作確認、ロールバック機能 |
| 情報の持ち出し | 高 | DLP対策、エクスポート制限 |

### 2.2 保護すべき資産
1. **企業マスターデータ**: 企業情報、財務状況、欠格フラグ
2. **判定結果**: 入札参加可否、不適格理由
3. **APIキー**: 外部サービス認証情報
4. **監査ログ**: システム利用履歴

---

## 3. 認証・認可設計

### 3.1 認証（Authentication）

#### 3.1.1 ユーザー認証
```typescript
namespace Security {
  export class Authentication {
    /**
     * Google OAuth 2.0による認証
     */
    static authenticate(): AuthResult {
      const user = Session.getActiveUser();
      const email = user.getEmail();
      
      if (!email) {
        throw new AuthenticationError('User not authenticated');
      }
      
      // ドメイン検証
      const allowedDomains = this.getAllowedDomains();
      const userDomain = email.split('@')[1];
      
      if (!allowedDomains.includes(userDomain)) {
        throw new AuthenticationError('Domain not authorized');
      }
      
      return {
        authenticated: true,
        userId: email,
        domain: userDomain,
        timestamp: new Date()
      };
    }
    
    /**
     * 許可ドメインリスト取得
     */
    private static getAllowedDomains(): string[] {
      const scriptProperties = PropertiesService.getScriptProperties();
      const domains = scriptProperties.getProperty('ALLOWED_DOMAINS');
      return domains ? JSON.parse(domains) : [];
    }
  }
}
```

#### 3.1.2 APIキー認証
```typescript
namespace Security {
  export class APIAuthentication {
    /**
     * APIキーの検証
     */
    static validateAPIKey(providedKey: string, service: string): boolean {
      const storedKey = this.getStoredAPIKey(service);
      
      // タイミング攻撃対策
      return this.secureCompare(providedKey, storedKey);
    }
    
    /**
     * セキュアな文字列比較
     */
    private static secureCompare(a: string, b: string): boolean {
      if (a.length !== b.length) return false;
      
      let result = 0;
      for (let i = 0; i < a.length; i++) {
        result |= a.charCodeAt(i) ^ b.charCodeAt(i);
      }
      
      return result === 0;
    }
  }
}
```

### 3.2 認可（Authorization）

#### 3.2.1 ロールベースアクセス制御（RBAC）
```typescript
namespace Security {
  export enum Role {
    ADMIN = 'ADMIN',           // システム管理者
    MANAGER = 'MANAGER',       // 管理者
    USER = 'USER',            // 一般利用者
    VIEWER = 'VIEWER'         // 閲覧のみ
  }
  
  export class Authorization {
    private static rolePermissions = new Map<Role, Set<string>>([
      [Role.ADMIN, new Set(['*'])],  // 全権限
      [Role.MANAGER, new Set([
        'judgment.execute',
        'judgment.view',
        'report.generate',
        'data.import',
        'data.export'
      ])],
      [Role.USER, new Set([
        'judgment.execute',
        'judgment.view',
        'report.generate'
      ])],
      [Role.VIEWER, new Set([
        'judgment.view',
        'report.view'
      ])]
    ]);
    
    /**
     * 権限チェック
     */
    static checkPermission(userId: string, permission: string): boolean {
      const userRole = this.getUserRole(userId);
      const permissions = this.rolePermissions.get(userRole);
      
      return permissions?.has('*') || permissions?.has(permission) || false;
    }
    
    /**
     * ユーザーロール取得
     */
    private static getUserRole(userId: string): Role {
      // ユーザー設定シートから取得
      const userSheet = this.getUserSheet();
      const userData = userSheet.getDataRange().getValues();
      
      const userRow = userData.find(row => row[0] === userId);
      return userRow ? userRow[1] as Role : Role.VIEWER;
    }
  }
}
```

#### 3.2.2 データレベルアクセス制御
```typescript
namespace Security {
  export class DataAccessControl {
    /**
     * 企業データへのアクセス権限チェック
     */
    static canAccessCompanyData(userId: string, companyId: string): boolean {
      // 管理者は全企業にアクセス可能
      if (Authorization.getUserRole(userId) === Role.ADMIN) {
        return true;
      }
      
      // 一般ユーザーは自社のみ
      const userCompany = this.getUserCompany(userId);
      return userCompany === companyId;
    }
    
    /**
     * データマスキング
     */
    static maskSensitiveData<T>(data: T, userRole: Role): T {
      if (userRole === Role.ADMIN) return data;
      
      const masked = { ...data };
      const sensitiveFields = ['taxId', 'bankAccount', 'internalNotes'];
      
      sensitiveFields.forEach(field => {
        if (masked[field]) {
          masked[field] = '***MASKED***';
        }
      });
      
      return masked;
    }
  }
}
```

---

## 4. データ保護

### 4.1 保存時の暗号化

#### 4.1.1 機密データの暗号化
```typescript
namespace Security {
  export class Encryption {
    /**
     * AES暗号化（Google Apps Script制約内での実装）
     */
    static encrypt(plainText: string, key: string): string {
      // 簡易暗号化（実運用では適切なライブラリを使用）
      const cipher = Utilities.computeHmacSignature(
        Utilities.MacAlgorithm.HMAC_SHA_256,
        plainText,
        key
      );
      return Utilities.base64Encode(cipher);
    }
    
    /**
     * 復号化
     */
    static decrypt(encryptedText: string, key: string): string {
      // 実装注意: GASでは完全な暗号化ライブラリが使用できない
      // 重要データは外部の暗号化サービスを検討
      return plainText; // 仮実装
    }
    
    /**
     * APIキーの安全な保存
     */
    static storeAPIKey(service: string, apiKey: string): void {
      const scriptProperties = PropertiesService.getScriptProperties();
      const encryptedKey = this.encrypt(apiKey, this.getMasterKey());
      
      scriptProperties.setProperty(`${service}_API_KEY`, encryptedKey);
    }
  }
}
```

### 4.2 通信時の暗号化

#### 4.2.1 HTTPS通信の強制
```typescript
namespace Security {
  export class SecureConnection {
    /**
     * セキュアな外部API呼び出し
     */
    static fetch(url: string, options: any = {}): HTTPResponse {
      // HTTPSでない場合は拒否
      if (!url.startsWith('https://')) {
        throw new SecurityError('HTTPS required for external connections');
      }
      
      // セキュリティヘッダーの追加
      const secureOptions = {
        ...options,
        headers: {
          ...options.headers,
          'X-Content-Type-Options': 'nosniff',
          'X-Frame-Options': 'DENY',
          'Strict-Transport-Security': 'max-age=31536000'
        }
      };
      
      return UrlFetchApp.fetch(url, secureOptions);
    }
  }
}
```

### 4.3 データ消去

#### 4.3.1 安全なデータ削除
```typescript
namespace Security {
  export class DataSanitization {
    /**
     * 機密データの完全削除
     */
    static secureDelete(range: Range): void {
      // 1. データを上書き（複数回）
      const rows = range.getNumRows();
      const cols = range.getNumColumns();
      const dummy = Array(rows).fill(Array(cols).fill('DELETED'));
      
      for (let i = 0; i < 3; i++) {
        range.setValues(dummy);
        SpreadsheetApp.flush();
      }
      
      // 2. 最終的にクリア
      range.clear();
      
      // 3. 削除ログの記録
      AuditLogger.log('DATA_DELETED', {
        range: range.getA1Notation(),
        timestamp: new Date()
      });
    }
  }
}
```

---

## 5. 入力検証とサニタイゼーション

### 5.1 入力検証

#### 5.1.1 バリデーションルール
```typescript
namespace Security {
  export class InputValidator {
    /**
     * 包括的な入力検証
     */
    static validate(input: any, rules: ValidationRule[]): ValidationResult {
      const errors: ValidationError[] = [];
      
      rules.forEach(rule => {
        if (!this.validateRule(input, rule)) {
          errors.push({
            field: rule.field,
            message: rule.message,
            value: input[rule.field]
          });
        }
      });
      
      return {
        valid: errors.length === 0,
        errors
      };
    }
    
    /**
     * SQLインジェクション対策
     */
    static sanitizeForQuery(input: string): string {
      // 危険な文字をエスケープ
      return input
        .replace(/'/g, "''")
        .replace(/;/g, '')
        .replace(/--/g, '')
        .replace(/\/\*/g, '')
        .replace(/\*\//g, '');
    }
    
    /**
     * XSS対策
     */
    static sanitizeForHTML(input: string): string {
      const htmlEntities = {
        '&': '&amp;',
        '<': '&lt;',
        '>': '&gt;',
        '"': '&quot;',
        "'": '&#x27;',
        '/': '&#x2F;'
      };
      
      return input.replace(/[&<>"'\/]/g, char => htmlEntities[char]);
    }
  }
}
```

#### 5.1.2 ファイルアップロード検証
```typescript
namespace Security {
  export class FileValidator {
    private static readonly ALLOWED_TYPES = ['application/pdf'];
    private static readonly MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
    
    /**
     * PDFファイルの検証
     */
    static validatePDF(fileBlob: Blob): ValidationResult {
      const errors = [];
      
      // ファイルタイプチェック
      if (!this.ALLOWED_TYPES.includes(fileBlob.getContentType())) {
        errors.push('Invalid file type. Only PDF allowed.');
      }
      
      // ファイルサイズチェック
      if (fileBlob.getBytes().length > this.MAX_FILE_SIZE) {
        errors.push('File size exceeds 10MB limit.');
      }
      
      // マジックナンバーチェック
      const bytes = fileBlob.getBytes();
      const pdfMagic = [0x25, 0x50, 0x44, 0x46]; // %PDF
      
      if (!this.checkMagicNumber(bytes, pdfMagic)) {
        errors.push('Invalid PDF file format.');
      }
      
      return {
        valid: errors.length === 0,
        errors
      };
    }
  }
}
```

---

## 6. 監査とログ管理

### 6.1 監査ログ

#### 6.1.1 ログ記録
```typescript
namespace Security {
  export class AuditLogger {
    private static readonly LOG_SHEET_NAME = 'AUDIT_LOG';
    
    /**
     * 監査ログの記録
     */
    static log(action: string, details: any): void {
      try {
        const logEntry = {
          timestamp: new Date(),
          userId: Session.getActiveUser().getEmail(),
          action: action,
          details: JSON.stringify(details),
          ipAddress: this.getClientIP(),
          userAgent: this.getUserAgent(),
          sessionId: Session.getTemporaryActiveUserKey()
        };
        
        this.writeToLogSheet(logEntry);
        
        // 重要なイベントは即座にアラート
        if (this.isCriticalAction(action)) {
          this.sendAlert(logEntry);
        }
      } catch (error) {
        console.error('Audit logging failed:', error);
      }
    }
    
    /**
     * ログシートへの書き込み
     */
    private static writeToLogSheet(entry: any): void {
      const sheet = this.getLogSheet();
      const row = [
        entry.timestamp,
        entry.userId,
        entry.action,
        entry.details,
        entry.ipAddress,
        entry.userAgent,
        entry.sessionId
      ];
      
      sheet.appendRow(row);
    }
    
    /**
     * 重要アクション判定
     */
    private static isCriticalAction(action: string): boolean {
      const criticalActions = [
        'AUTHENTICATION_FAILED',
        'UNAUTHORIZED_ACCESS',
        'DATA_EXPORT',
        'PERMISSION_CHANGED',
        'API_KEY_ACCESSED'
      ];
      
      return criticalActions.includes(action);
    }
  }
}
```

#### 6.1.2 ログ分析
```typescript
namespace Security {
  export class LogAnalyzer {
    /**
     * 異常パターンの検出
     */
    static detectAnomalies(): AnomalyReport {
      const logs = this.getRecentLogs(24); // 過去24時間
      const anomalies = [];
      
      // 1. 大量アクセスの検出
      const accessCounts = this.countByUser(logs);
      accessCounts.forEach((count, userId) => {
        if (count > 1000) {
          anomalies.push({
            type: 'EXCESSIVE_ACCESS',
            userId,
            count,
            severity: 'HIGH'
          });
        }
      });
      
      // 2. 認証失敗の検出
      const authFailures = logs.filter(log => 
        log.action === 'AUTHENTICATION_FAILED'
      );
      if (authFailures.length > 10) {
        anomalies.push({
          type: 'BRUTE_FORCE_ATTEMPT',
          count: authFailures.length,
          severity: 'CRITICAL'
        });
      }
      
      // 3. 異常時間帯のアクセス
      const oddHourAccess = logs.filter(log => {
        const hour = new Date(log.timestamp).getHours();
        return hour < 6 || hour > 22;
      });
      
      return { anomalies, analyzedLogs: logs.length };
    }
  }
}
```

### 6.2 ログ保管

#### 6.2.1 ログローテーション
```typescript
namespace Security {
  export class LogRotation {
    private static readonly MAX_LOG_ROWS = 100000;
    private static readonly ARCHIVE_FOLDER_ID = 'xxxxx';
    
    /**
     * ログのアーカイブ
     */
    static rotateLogSheet(): void {
      const logSheet = AuditLogger.getLogSheet();
      const rowCount = logSheet.getLastRow();
      
      if (rowCount > this.MAX_LOG_ROWS) {
        // アーカイブ用スプレッドシート作成
        const archive = this.createArchive();
        
        // データ移動
        const dataToArchive = logSheet.getRange(
          2, 1, 
          rowCount - this.MAX_LOG_ROWS / 2, 
          logSheet.getLastColumn()
        ).getValues();
        
        archive.getSheetByName('Sheet1')
          .getRange(1, 1, dataToArchive.length, dataToArchive[0].length)
          .setValues(dataToArchive);
        
        // 元データ削除
        logSheet.deleteRows(2, dataToArchive.length);
        
        AuditLogger.log('LOG_ROTATED', {
          archivedRows: dataToArchive.length,
          archiveId: archive.getId()
        });
      }
    }
  }
}
```

---

## 7. エラーハンドリングとセキュリティ

### 7.1 セキュアなエラーハンドリング

#### 7.1.1 エラー情報の制御
```typescript
namespace Security {
  export class SecureErrorHandler {
    /**
     * エラー情報のサニタイズ
     */
    static handleError(error: Error, context: string): ErrorResponse {
      // 本番環境では詳細情報を隠蔽
      const isProduction = this.isProduction();
      
      const errorResponse: ErrorResponse = {
        success: false,
        error: {
          code: this.getErrorCode(error),
          message: isProduction 
            ? this.getGenericMessage(error)
            : error.message,
          timestamp: new Date()
        }
      };
      
      // 内部ログには詳細を記録
      AuditLogger.log('ERROR', {
        context,
        error: {
          message: error.message,
          stack: error.stack,
          ...errorResponse
        }
      });
      
      return errorResponse;
    }
    
    /**
     * 一般的なエラーメッセージ
     */
    private static getGenericMessage(error: Error): string {
      const errorMap = new Map([
        ['AUTHENTICATION_ERROR', 'ログインが必要です'],
        ['AUTHORIZATION_ERROR', 'アクセス権限がありません'],
        ['VALIDATION_ERROR', '入力内容に誤りがあります'],
        ['SYSTEM_ERROR', 'システムエラーが発生しました']
      ]);
      
      return errorMap.get(error.name) || 'エラーが発生しました';
    }
  }
}
```

---

## 8. セキュリティ運用

### 8.1 定期的なセキュリティレビュー

#### 8.1.1 チェックリスト
- [ ] アクセス権限の棚卸（月次）
- [ ] APIキーのローテーション（四半期）
- [ ] ログ分析レポート（週次）
- [ ] 脆弱性スキャン（月次）
- [ ] セキュリティパッチ適用（随時）

### 8.2 インシデント対応

#### 8.2.1 インシデント対応フロー
```typescript
namespace Security {
  export class IncidentResponse {
    /**
     * セキュリティインシデント検出時の対応
     */
    static handleIncident(incident: SecurityIncident): void {
      // 1. 影響範囲の特定
      const impact = this.assessImpact(incident);
      
      // 2. 即座の対応
      if (impact.severity === 'CRITICAL') {
        this.lockdownSystem();
      }
      
      // 3. 通知
      this.notifySecurityTeam(incident);
      
      // 4. 証跡保全
      this.preserveEvidence(incident);
      
      // 5. 対応記録
      AuditLogger.log('SECURITY_INCIDENT', {
        incident,
        impact,
        response: 'INITIATED'
      });
    }
    
    /**
     * システムロックダウン
     */
    private static lockdownSystem(): void {
      // 全ユーザーのアクセスを一時停止
      const lockdownMode = {
        enabled: true,
        timestamp: new Date(),
        allowedUsers: ['security@company.com']
      };
      
      PropertiesService.getScriptProperties()
        .setProperty('LOCKDOWN_MODE', JSON.stringify(lockdownMode));
    }
  }
}
```

---

## 9. コンプライアンス

### 9.1 個人情報保護

#### 9.1.1 個人情報の取り扱い
- 最小限の個人情報のみ収集
- 目的外使用の禁止
- 保存期間の設定（3年）
- 削除要求への対応

### 9.2 監査要件

#### 9.2.1 監査証跡の保持
- ログ保存期間: 3年
- 改ざん防止措置
- 定期的なバックアップ

---

## 10. セキュリティテスト

### 10.1 ペネトレーションテスト

#### 10.1.1 テスト項目
```typescript
namespace SecurityTest {
  export class PenetrationTest {
    /**
     * セキュリティテストスイート
     */
    static runSecurityTests(): TestResult[] {
      const results = [];
      
      // 1. 認証バイパステスト
      results.push(this.testAuthBypass());
      
      // 2. インジェクションテスト
      results.push(this.testInjection());
      
      // 3. 権限昇格テスト
      results.push(this.testPrivilegeEscalation());
      
      // 4. セッション管理テスト
      results.push(this.testSessionManagement());
      
      return results;
    }
  }
}
```

---

## 11. 実装チェックリスト

### 11.1 必須実装項目
- [ ] 認証機能の実装
- [ ] 認可機能の実装
- [ ] 入力検証の実装
- [ ] 監査ログの実装
- [ ] エラーハンドリングの実装
- [ ] データ暗号化の実装

### 11.2 運用準備項目
- [ ] セキュリティポリシーの策定
- [ ] インシデント対応手順書
- [ ] 定期監査スケジュール
- [ ] セキュリティ教育計画

---

## 改訂履歴
- 2025-07-23: 初版作成