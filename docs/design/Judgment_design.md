# 入札公告要件判定システム - 判定エンジン詳細設計書

## 関連ドキュメント
- **全体設計書**: `/Project_design.md`
- **要件定義書**: `/要件定義書.md`
- **判定詳細**: `/別紙2：入札公告要件判定詳細.md`
- **TODO リスト**: `/docs/todo/Server_TODO.md`

---

## 1. 判定エンジン概要

### 1.1 目的
入札公告の要件と企業情報を照合し、6種類の要件（欠格要件、等級要件、地域要件、実績要件、技術者要件、JV要件）に対して高精度な自動判定を実現する。

### 1.2 設計方針
- **モジュール化**: 要件種別ごとに独立した判定モジュール
- **拡張性**: 新しい要件種別の追加が容易
- **トレーサビリティ**: 判定根拠の明確化

### 1.3 判定フロー概要
```
┌─────────────────────────────────────────────────────────────┐
│                      判定エンジン全体フロー                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [公告要件 + 企業情報]                                      │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────┐                                       │
│  │ 判定前検証     │ → データ不整合時はエラー               │
│  └────────┬────────┘                                       │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────┐                                       │
│  │ 欠格要件判定   │ → NG時は即終了                        │
│  └────────┬────────┘                                       │
│           │ OK                                             │
│           ▼                                                 │
│  ┌─────────────────────────────────────┐                   │
│  │    並列判定処理                    │                   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ │                   │
│  │  │等級要件 │ │地域要件 │ │実績要件 │ │                   │
│  │  └─────────┘ └─────────┘ └─────────┘ │                   │
│  │  ┌─────────┐ ┌─────────┐             │                   │
│  │  │技術者  │ │JV要件  │             │                   │
│  │  └─────────┘ └─────────┘             │                   │
│  └────────┬────────────────────────────┘                   │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────┐                                       │
│  │ 総合判定       │ → 全要件の結果を統合                  │
│  └────────┬────────┘                                       │
│           │                                                 │
│           ▼                                                 │
│  [判定結果オブジェクト]                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. JudgmentEngine（メインエンジン）

### 2.1 クラス構成
```javascript
/**
 * 判定エンジンメインクラス
 */
class JudgmentEngine {
  constructor() {
    // 判定モジュールの初期化
    this.judges = {
      disqualification: new DisqualificationJudge(),
      grade: new GradeJudge(),
      region: new RegionJudge(),
      achievement: new AchievementJudge(),
      engineer: new EngineerJudge(),
      jv: new JVJudge()
    };
    
    // 判定順序の定義
    this.judgmentOrder = [
      'disqualification',  // 欠格要件（必須・即終了）
      'grade',            // 等級要件
      'region',           // 地域要件
      'achievement',      // 実績要件
      'engineer',         // 技術者要件
      'jv'               // JV要件
    ];
  }

  /**
   * 総合判定実行
   * @param {Object} announcement - 公告情報
   * @param {Object} company - 企業情報
   * @returns {JudgmentResult} 判定結果
   */
  executeJudgment(announcement, company) {
    const startTime = new Date();
    const result = new JudgmentResult();
    
    try {
      // 1. 入力検証
      this.validateInput(announcement, company);
      
      // 2. 判定コンテキストの準備
      const context = this.prepareContext(announcement, company);
      
      // 3. 欠格要件判定（最優先）
      const disqualificationResult = this.judges.disqualification
        .judge(context);
      
      if (!disqualificationResult.qualified) {
        return this.createDisqualifiedResult(
          disqualificationResult, 
          startTime
        );
      }
      
      // 4. その他要件の並列判定
      const parallelResults = this.executeParallelJudgments(context);
      
      // 5. 結果統合
      return this.aggregateResults(
        disqualificationResult, 
        parallelResults, 
        context, 
        startTime
      );
      
    } catch (error) {
      return this.createErrorResult(error, startTime);
    }
  }

  /**
   * 並列判定実行
   * @private
   */
  executeParallelJudgments(context) {
    const results = {};
    
    // 欠格要件以外を並列実行
    const judgmentTypes = this.judgmentOrder.filter(
      type => type !== 'disqualification'
    );
    
    for (const type of judgmentTypes) {
      try {
        results[type] = this.judges[type].judge(context);
      } catch (error) {
        results[type] = {
          qualified: false,
          error: error.message,
          details: []
        };
      }
    }
    
    return results;
  }
}
```

### 2.2 判定結果スキーマ
```javascript
/**
 * 判定結果クラス
 */
class JudgmentResult {
  constructor() {
    this.announcementId = null;      // 公告ID
    this.companyId = null;           // 企業ID
    this.overallResult = null;       // 総合判定結果
    this.executionTime = null;       // 実行時間
    this.timestamp = new Date();     // 判定実施日時
    
    // 要件別結果
    this.requirementResults = {
      disqualification: null,
      grade: null,
      region: null,
      achievement: null,
      engineer: null,
      jv: null
    };
    
    // 詳細情報
    this.qualifiedRequirements = [];   // 適格要件リスト
    this.disqualifiedRequirements = []; // 不適格要件リスト
    this.insufficientItems = [];       // 不足項目詳細
    this.recommendations = [];         // 改善提案
  }

  /**
   * 総合判定結果の計算
   */
  calculateOverallResult() {
    // 全要件がOKの場合のみ適格
    this.overallResult = Object.values(this.requirementResults)
      .every(result => result && result.qualified);
    
    // 適格/不適格要件の分類
    for (const [type, result] of Object.entries(this.requirementResults)) {
      if (result) {
        if (result.qualified) {
          this.qualifiedRequirements.push(type);
        } else {
          this.disqualifiedRequirements.push(type);
          this.insufficientItems.push(...(result.insufficientItems || []));
        }
      }
    }
  }
}
```

---

## 3. DisqualificationJudge（欠格要件判定）

### 3.1 判定ロジック詳細
```javascript
class DisqualificationJudge {
  constructor() {
    // 欠格フラグと判定理由のマッピング
    this.flagMappings = {
      bankruptcy_flg: {
        message: '破産手続き開始の決定を受けて復権を得ない者',
        category: '経営状態'
      },
      violent_relation_flg: {
        message: '暴力団員又は暴力団員でなくなった日から5年を経過しない者',
        category: '反社会的勢力'
      },
      rehabilitation_flg: {
        message: '民事再生法又は会社更生法の適用申請中',
        category: '経営状態'
      },
      suspension_flg: {
        message: '営業停止又は指名停止期間中',
        category: '行政処分'
      },
      info_security_insufficient_flg: {
        message: '情報保全体制が不十分',
        category: 'セキュリティ'
      },
      social_insurance_delinquent_flg: {
        message: '社会保険料等の滞納あり',
        category: '納税・保険'
      },
      terrorist_flg: {
        message: '破壊活動防止法の規定する暴力主義的破壊活動を行う団体',
        category: '反社会的勢力'
      },
      foreign_restriction_flg: {
        message: '外国の法令により日本における営業を禁止されている者',
        category: '法的制約'
      },
      boj_suspended_flg: {
        message: '日本銀行取引停止処分中',
        category: '金融取引'
      }
    };
  }

  /**
   * 欠格要件判定実行
   * @param {Object} context - 判定コンテキスト
   * @returns {Object} 判定結果
   */
  judge(context) {
    const company = context.company;
    const result = {
      qualified: true,
      disqualificationReasons: [],
      checkedFlags: [],
      details: []
    };

    // 各フラグをチェック
    for (const [flag, config] of Object.entries(this.flagMappings)) {
      const flagValue = company[flag];
      
      result.checkedFlags.push({
        flag: flag,
        value: flagValue,
        timestamp: new Date()
      });

      if (flagValue === true) {
        result.qualified = false;
        result.disqualificationReasons.push({
          flag: flag,
          reason: config.message,
          category: config.category
        });
      }
    }

    // 詳細メッセージの生成
    if (!result.qualified) {
      result.message = this.generateDisqualificationMessage(
        result.disqualificationReasons
      );
    }

    return result;
  }

  /**
   * 欠格メッセージ生成
   * @private
   */
  generateDisqualificationMessage(reasons) {
    if (reasons.length === 0) return '';
    
    const categories = {};
    reasons.forEach(reason => {
      if (!categories[reason.category]) {
        categories[reason.category] = [];
      }
      categories[reason.category].push(reason.reason);
    });

    let message = '以下の欠格要件に該当します：\n';
    for (const [category, items] of Object.entries(categories)) {
      message += `\n【${category}】\n`;
      items.forEach(item => {
        message += `  ・${item}\n`;
      });
    }

    return message;
  }
}
```

---

## 4. GradeJudge（等級要件判定）

### 4.1 等級判定ロジック
```javascript
class GradeJudge {
  constructor() {
    // 等級の階層定義（上位ほど値が大きい）
    this.gradeHierarchy = {
      'S': 5,
      'A': 4,
      'B': 3,
      'C': 2,
      'D': 1,
      '未登録': 0
    };
    
    // 工事種別の同等性マッピング
    this.constructionTypeEquivalence = {
      '土木一式': ['土木', '土木工事'],
      '建築一式': ['建築', '建築工事'],
      '電気': ['電気工事', '電気設備'],
      '管': ['管工事', '給排水'],
      '舗装': ['舗装工事', 'アスファルト']
    };
  }

  /**
   * 等級要件判定
   * @param {Object} context - 判定コンテキスト
   * @returns {Object} 判定結果
   */
  judge(context) {
    const requirements = context.requirements.grade || [];
    const companyLicenses = context.company.licenses || [];
    
    const result = {
      qualified: true,
      details: [],
      insufficientItems: []
    };

    for (const requirement of requirements) {
      const checkResult = this.checkGradeRequirement(
        requirement, 
        companyLicenses
      );
      
      result.details.push(checkResult);
      
      if (!checkResult.satisfied) {
        result.qualified = false;
        result.insufficientItems.push({
          requirement: requirement,
          actual: checkResult.actualGrade,
          message: checkResult.message
        });
      }
    }

    return result;
  }

  /**
   * 個別等級要件チェック
   * @private
   */
  checkGradeRequirement(requirement, licenses) {
    const requiredType = this.normalizeConstructionType(
      requirement.constructionType
    );
    const requiredGrade = requirement.grade;
    const condition = requirement.condition || 'exact'; // exact, min

    // 該当する営業品目を検索
    const matchingLicense = licenses.find(license => 
      this.isConstructionTypeMatch(license.type, requiredType)
    );

    if (!matchingLicense) {
      return {
        satisfied: false,
        message: `${requiredType}の登録がありません`,
        actualGrade: null
      };
    }

    // 等級比較
    const comparison = this.compareGrades(
      matchingLicense.grade,
      requiredGrade,
      condition
    );

    return {
      satisfied: comparison.satisfied,
      message: comparison.message,
      actualGrade: matchingLicense.grade,
      requiredGrade: requiredGrade
    };
  }

  /**
   * 等級比較
   * @private
   */
  compareGrades(actualGrade, requiredGrade, condition) {
    const actualValue = this.gradeHierarchy[actualGrade] || 0;
    const requiredValue = this.gradeHierarchy[requiredGrade] || 0;

    switch (condition) {
      case 'exact':
        return {
          satisfied: actualValue === requiredValue,
          message: actualValue === requiredValue ? 
            '等級要件を満たしています' : 
            `等級が一致しません（要求:${requiredGrade}, 実際:${actualGrade}）`
        };
        
      case 'min':
        return {
          satisfied: actualValue >= requiredValue,
          message: actualValue >= requiredValue ? 
            '等級要件を満たしています' : 
            `等級が不足しています（要求:${requiredGrade}以上, 実際:${actualGrade}）`
        };
        
      default:
        return {
          satisfied: false,
          message: `不明な条件: ${condition}`
        };
    }
  }
}
```

---

## 5. RegionJudge（地域要件判定）

### 5.1 地域判定ロジック
```javascript
class RegionJudge {
  constructor() {
    // 地域階層定義
    this.regionHierarchy = {
      '全国': 0,
      '地方': 1,    // 関東地方、近畿地方など
      '都道府県': 2,
      '市区町村': 3,
      '地区': 4     // ○○地区など
    };
    
    // 地方と都道府県のマッピング
    this.regionMapping = {
      '北海道地方': ['北海道'],
      '東北地方': ['青森県', '岩手県', '宮城県', '秋田県', '山形県', '福島県'],
      '関東地方': ['茨城県', '栃木県', '群馬県', '埼玉県', '千葉県', '東京都', '神奈川県'],
      '中部地方': ['新潟県', '富山県', '石川県', '福井県', '山梨県', '長野県', '岐阜県', '静岡県', '愛知県'],
      '近畿地方': ['三重県', '滋賀県', '京都府', '大阪府', '兵庫県', '奈良県', '和歌山県'],
      '中国地方': ['鳥取県', '島根県', '岡山県', '広島県', '山口県'],
      '四国地方': ['徳島県', '香川県', '愛媛県', '高知県'],
      '九州地方': ['福岡県', '佐賀県', '長崎県', '熊本県', '大分県', '宮崎県', '鹿児島県'],
      '沖縄地方': ['沖縄県']
    };
  }

  /**
   * 地域要件判定
   * @param {Object} context - 判定コンテキスト
   * @returns {Object} 判定結果
   */
  judge(context) {
    const requirements = context.requirements.region || [];
    const companyOffices = context.company.offices || [];
    
    const result = {
      qualified: true,
      details: [],
      qualifiedOffices: [],
      insufficientItems: []
    };

    // 地域要件がない場合は適格
    if (requirements.length === 0) {
      result.message = '地域要件の指定はありません';
      return result;
    }

    for (const requirement of requirements) {
      const checkResult = this.checkRegionRequirement(
        requirement,
        companyOffices
      );
      
      result.details.push(checkResult);
      
      if (checkResult.satisfied) {
        result.qualifiedOffices.push(...checkResult.matchingOffices);
      } else {
        result.qualified = false;
        result.insufficientItems.push({
          requirement: requirement,
          message: checkResult.message
        });
      }
    }

    // 重複を除去
    result.qualifiedOffices = [...new Set(result.qualifiedOffices)];

    return result;
  }

  /**
   * 個別地域要件チェック
   * @private
   */
  checkRegionRequirement(requirement, offices) {
    const targetRegion = requirement.region;
    const regionType = this.identifyRegionType(targetRegion);
    
    const matchingOffices = offices.filter(office => 
      this.isInTargetRegion(office.address, targetRegion, regionType)
    );

    return {
      satisfied: matchingOffices.length > 0,
      matchingOffices: matchingOffices.map(o => o.id),
      message: matchingOffices.length > 0 ?
        `${targetRegion}に${matchingOffices.length}拠点あります` :
        `${targetRegion}に拠点がありません`
    };
  }

  /**
   * 地域包含チェック
   * @private
   */
  isInTargetRegion(address, targetRegion, regionType) {
    switch (regionType) {
      case '全国':
        return true;
        
      case '地方':
        const prefectures = this.regionMapping[targetRegion] || [];
        return prefectures.some(pref => address.includes(pref));
        
      case '都道府県':
        return address.includes(targetRegion);
        
      case '市区町村':
        // より詳細な住所解析が必要
        return this.parseAddress(address).city === targetRegion;
        
      default:
        return address.includes(targetRegion);
    }
  }

  /**
   * 住所解析
   * @private
   */
  parseAddress(address) {
    // 簡易的な住所解析
    const prefectureMatch = address.match(
      /(北海道|京都府|大阪府|.{2,3}県)/
    );
    const cityMatch = address.match(
      /(.{1,6}市|.{1,6}区|.{1,6}町|.{1,6}村)/
    );
    
    return {
      prefecture: prefectureMatch ? prefectureMatch[1] : null,
      city: cityMatch ? cityMatch[1] : null
    };
  }
}
```

---

## 6. AchievementJudge（実績要件判定）

### 6.1 実績判定ロジック
```javascript
class AchievementJudge {
  constructor() {
    // 実績要件パターン
    this.patterns = {
      amount: /(\d+(?:,\d{3})*(?:\.\d+)?)\s*(?:億|千万|百万|万)?円/,
      score: /(?:平均)?(\d+(?:\.\d+)?)\s*点/,
      count: /(\d+)\s*件/,
      period: /過去(\d+)年/,
      jvRatio: /JV.*?(\d+(?:\.\d+)?)\s*[%％]/
    };
  }

  /**
   * 実績要件判定
   * @param {Object} context - 判定コンテキスト
   * @returns {Object} 判定結果
   */
  judge(context) {
    const requirements = context.requirements.achievement || [];
    const companyAchievements = this.collectAchievements(context);
    
    const result = {
      qualified: true,
      details: [],
      qualifiedAchievements: [],
      insufficientItems: []
    };

    for (const requirement of requirements) {
      const checkResult = this.checkAchievementRequirement(
        requirement,
        companyAchievements
      );
      
      result.details.push(checkResult);
      
      if (checkResult.satisfied) {
        result.qualifiedAchievements.push(...checkResult.matchingAchievements);
      } else {
        result.qualified = false;
        result.insufficientItems.push({
          requirement: requirement,
          actualValue: checkResult.actualValue,
          message: checkResult.message
        });
      }
    }

    return result;
  }

  /**
   * 実績収集
   * @private
   */
  collectAchievements(context) {
    const achievements = [];
    
    // 企業実績
    if (context.company.achievements) {
      achievements.push(...context.company.achievements.map(a => ({
        ...a,
        source: 'company'
      })));
    }
    
    // 拠点実績
    if (context.company.offices) {
      context.company.offices.forEach(office => {
        if (office.achievements) {
          achievements.push(...office.achievements.map(a => ({
            ...a,
            source: 'office',
            officeId: office.id
          })));
        }
      });
    }
    
    // 技術者実績
    if (context.company.engineers) {
      context.company.engineers.forEach(engineer => {
        if (engineer.achievements) {
          achievements.push(...engineer.achievements.map(a => ({
            ...a,
            source: 'engineer',
            engineerId: engineer.id
          })));
        }
      });
    }
    
    return achievements;
  }

  /**
   * 個別実績要件チェック
   * @private
   */
  checkAchievementRequirement(requirement, achievements) {
    const requirementType = this.identifyRequirementType(requirement);
    
    switch (requirementType) {
      case 'AMOUNT':
        return this.checkAmountRequirement(requirement, achievements);
        
      case 'SCORE':
        return this.checkScoreRequirement(requirement, achievements);
        
      case 'COUNT':
        return this.checkCountRequirement(requirement, achievements);
        
      case 'JV':
        return this.checkJVRequirement(requirement, achievements);
        
      default:
        return {
          satisfied: false,
          message: `不明な実績要件タイプ: ${requirement.description}`
        };
    }
  }

  /**
   * 金額要件チェック
   * @private
   */
  checkAmountRequirement(requirement, achievements) {
    const requiredAmount = this.parseAmount(requirement.description);
    const period = this.parsePeriod(requirement.description) || 999;
    const cutoffDate = new Date();
    cutoffDate.setFullYear(cutoffDate.getFullYear() - period);
    
    const eligibleAchievements = achievements.filter(a => {
      if (!a.completionDate || new Date(a.completionDate) < cutoffDate) {
        return false;
      }
      return a.contractAmount >= requiredAmount;
    });
    
    return {
      satisfied: eligibleAchievements.length > 0,
      matchingAchievements: eligibleAchievements.map(a => a.id),
      actualValue: eligibleAchievements.length > 0 ?
        Math.max(...eligibleAchievements.map(a => a.contractAmount)) : 0,
      message: eligibleAchievements.length > 0 ?
        `要求金額${this.formatAmount(requiredAmount)}以上の実績があります` :
        `要求金額${this.formatAmount(requiredAmount)}以上の実績がありません`
    };
  }

  /**
   * 工事成績要件チェック
   * @private
   */
  checkScoreRequirement(requirement, achievements) {
    const requiredScore = this.parseScore(requirement.description);
    const period = this.parsePeriod(requirement.description) || 999;
    const cutoffDate = new Date();
    cutoffDate.setFullYear(cutoffDate.getFullYear() - period);
    
    const eligibleAchievements = achievements.filter(a => {
      return a.completionDate && 
             new Date(a.completionDate) >= cutoffDate &&
             a.evaluationScore !== null;
    });
    
    if (eligibleAchievements.length === 0) {
      return {
        satisfied: false,
        message: '対象期間内の評価実績がありません'
      };
    }
    
    const averageScore = eligibleAchievements.reduce(
      (sum, a) => sum + a.evaluationScore, 0
    ) / eligibleAchievements.length;
    
    return {
      satisfied: averageScore >= requiredScore,
      matchingAchievements: eligibleAchievements.map(a => a.id),
      actualValue: averageScore,
      message: averageScore >= requiredScore ?
        `平均工事成績${averageScore.toFixed(1)}点（要求:${requiredScore}点以上）` :
        `平均工事成績${averageScore.toFixed(1)}点が要求${requiredScore}点に不足`
    };
  }

  /**
   * 金額解析
   * @private
   */
  parseAmount(text) {
    const match = text.match(this.patterns.amount);
    if (!match) return 0;
    
    let amount = parseFloat(match[1].replace(/,/g, ''));
    const unit = match[2];
    
    switch (unit) {
      case '億': amount *= 100000000; break;
      case '千万': amount *= 10000000; break;
      case '百万': amount *= 1000000; break;
      case '万': amount *= 10000; break;
    }
    
    return amount;
  }

  /**
   * 金額フォーマット
   * @private
   */
  formatAmount(amount) {
    if (amount >= 100000000) {
      return `${(amount / 100000000).toFixed(1)}億円`;
    } else if (amount >= 10000000) {
      return `${(amount / 10000000).toFixed(0)}千万円`;
    } else if (amount >= 1000000) {
      return `${(amount / 1000000).toFixed(0)}百万円`;
    } else if (amount >= 10000) {
      return `${(amount / 10000).toFixed(0)}万円`;
    } else {
      return `${amount.toFixed(0)}円`;
    }
  }
}
```

---

## 7. EngineerJudge（技術者要件判定）

### 7.1 技術者判定ロジック
```javascript
class EngineerJudge {
  constructor() {
    // 資格の同等性マッピング
    this.qualificationEquivalence = {
      '1級土木施工管理技士': [
        '技術士（建設部門）',
        '技術士（総合技術監理部門-建設）'
      ],
      '1級建築施工管理技士': [
        '一級建築士',
        '技術士（建設部門-建築）'
      ],
      '1級電気工事施工管理技士': [
        '第一種電気工事士',
        '技術士（電気電子部門）'
      ],
      '1級管工事施工管理技士': [
        '技術士（上下水道部門）',
        '技術士（衛生工学部門）'
      ]
    };
    
    // 監理技術者資格
    this.supervisorQualifications = [
      '監理技術者',
      '監理技術者講習修了者',
      '主任技術者'
    ];
  }

  /**
   * 技術者要件判定
   * @param {Object} context - 判定コンテキスト
   * @returns {Object} 判定結果
   */
  judge(context) {
    const requirements = context.requirements.engineer || [];
    const engineers = this.collectEngineers(context);
    
    const result = {
      qualified: true,
      details: [],
      assignableEngineers: [],
      insufficientItems: []
    };

    for (const requirement of requirements) {
      const checkResult = this.checkEngineerRequirement(
        requirement,
        engineers
      );
      
      result.details.push(checkResult);
      
      if (checkResult.satisfied) {
        result.assignableEngineers.push(...checkResult.qualifiedEngineers);
      } else {
        result.qualified = false;
        result.insufficientItems.push({
          requirement: requirement,
          availableCount: checkResult.availableCount,
          message: checkResult.message
        });
      }
    }

    // 配置可能性チェック
    if (result.qualified) {
      const assignmentCheck = this.checkAssignability(
        result.assignableEngineers,
        requirements
      );
      
      if (!assignmentCheck.possible) {
        result.qualified = false;
        result.insufficientItems.push({
          message: assignmentCheck.message
        });
      }
    }

    return result;
  }

  /**
   * 技術者収集
   * @private
   */
  collectEngineers(context) {
    const engineers = [];
    
    // 企業所属技術者
    if (context.company.engineers) {
      engineers.push(...context.company.engineers.map(e => ({
        ...e,
        affiliation: 'company'
      })));
    }
    
    // 拠点所属技術者
    if (context.company.offices) {
      context.company.offices.forEach(office => {
        if (office.engineers) {
          engineers.push(...office.engineers.map(e => ({
            ...e,
            affiliation: 'office',
            officeId: office.id
          })));
        }
      });
    }
    
    return engineers;
  }

  /**
   * 個別技術者要件チェック
   * @private
   */
  checkEngineerRequirement(requirement, engineers) {
    const requiredQualifications = this.parseQualifications(
      requirement.description
    );
    const requiredCount = requirement.count || 1;
    
    const qualifiedEngineers = engineers.filter(engineer => 
      this.hasRequiredQualification(engineer, requiredQualifications)
    );
    
    // 配置可能な技術者のみカウント
    const availableEngineers = qualifiedEngineers.filter(engineer =>
      this.isAvailable(engineer)
    );
    
    return {
      satisfied: availableEngineers.length >= requiredCount,
      qualifiedEngineers: availableEngineers.map(e => e.id),
      availableCount: availableEngineers.length,
      message: availableEngineers.length >= requiredCount ?
        `必要な技術者が${availableEngineers.length}名確保できます` :
        `技術者が不足しています（必要:${requiredCount}名、配置可能:${availableEngineers.length}名）`
    };
  }

  /**
   * 資格保有チェック
   * @private
   */
  hasRequiredQualification(engineer, requiredQualifications) {
    const engineerQualifications = engineer.qualifications || [];
    
    for (const required of requiredQualifications) {
      // 直接保有
      if (engineerQualifications.includes(required)) {
        return true;
      }
      
      // 同等資格保有
      const equivalents = this.qualificationEquivalence[required] || [];
      if (equivalents.some(eq => engineerQualifications.includes(eq))) {
        return true;
      }
    }
    
    return false;
  }

  /**
   * 配置可能性チェック
   * @private
   */
  isAvailable(engineer) {
    // 現在の配置状況確認
    if (engineer.currentProject) {
      const projectEndDate = new Date(engineer.currentProject.endDate);
      const announcementStartDate = new Date(); // 本来は公告の工期開始日
      
      return projectEndDate < announcementStartDate;
    }
    
    return true;
  }

  /**
   * 複数要件の配置可能性チェック
   * @private
   */
  checkAssignability(engineerIds, requirements) {
    // 技術者の重複配置チェック
    const uniqueEngineers = new Set(engineerIds);
    const totalRequired = requirements.reduce(
      (sum, req) => sum + (req.count || 1), 0
    );
    
    if (uniqueEngineers.size < totalRequired) {
      return {
        possible: false,
        message: `技術者の重複配置はできません（必要:${totalRequired}名、利用可能:${uniqueEngineers.size}名）`
      };
    }
    
    return { possible: true };
  }
}
```

---

## 8. 判定結果の保存と活用

### 8.1 判定結果の永続化
```javascript
class JudgmentResultRepository {
  /**
   * 判定結果の保存
   */
  save(result) {
    const evaluationMaster = this.getEvaluationMaster();
    const newRow = this.createEvaluationRow(result);
    
    evaluationMaster.appendRow(newRow);
    
    // 詳細結果も保存
    if (result.qualifiedRequirements.length > 0) {
      this.saveSufficiencyDetails(result);
    }
    
    if (result.disqualifiedRequirements.length > 0) {
      this.saveShortageDetails(result);
    }
  }

  /**
   * 評価マスター行作成
   * @private
   */
  createEvaluationRow(result) {
    return [
      result.announcementId,
      result.companyId,
      result.timestamp,
      result.overallResult ? '適格' : '不適格',
      result.requirementResults.disqualification?.qualified ? 'OK' : 'NG',
      result.requirementResults.grade?.qualified ? 'OK' : 'NG',
      result.requirementResults.region?.qualified ? 'OK' : 'NG',
      result.requirementResults.achievement?.qualified ? 'OK' : 'NG',
      result.requirementResults.engineer?.qualified ? 'OK' : 'NG',
      result.requirementResults.jv?.qualified ? 'OK' : 'NG',
      result.executionTime,
      JSON.stringify(result.recommendations)
    ];
  }

  /**
   * 充足要件詳細保存
   * @private
   */
  saveSufficiencyDetails(result) {
    const sufficiencySheet = this.getSufficiencySheet();
    
    result.qualifiedRequirements.forEach(reqType => {
      const reqResult = result.requirementResults[reqType];
      if (reqResult && reqResult.details) {
        reqResult.details.forEach(detail => {
          sufficiencySheet.appendRow([
            result.announcementId,
            result.companyId,
            reqType,
            detail.requirement,
            detail.actualValue,
            detail.message,
            result.timestamp
          ]);
        });
      }
    });
  }

  /**
   * 不足要件詳細保存
   * @private
   */
  saveShortageDetails(result) {
    const shortageSheet = this.getShortageSheet();
    
    result.insufficientItems.forEach(item => {
      shortageSheet.appendRow([
        result.announcementId,
        result.companyId,
        item.requirementType,
        item.requirement,
        item.actualValue,
        item.message,
        item.recommendation,
        result.timestamp
      ]);
    });
  }
}
```

---

## 9. パフォーマンス最適化

### 9.1 キャッシング戦略
```javascript
class JudgmentCache {
  constructor() {
    this.cache = CacheService.getScriptCache();
    this.TTL = 3600; // 1時間
  }

  /**
   * 判定結果のキャッシュ
   */
  cacheResult(announcementId, companyId, result) {
    const key = `judgment_${announcementId}_${companyId}`;
    const value = JSON.stringify({
      result: result,
      timestamp: new Date()
    });
    
    this.cache.put(key, value, this.TTL);
  }

  /**
   * キャッシュからの取得
   */
  getFromCache(announcementId, companyId) {
    const key = `judgment_${announcementId}_${companyId}`;
    const cached = this.cache.get(key);
    
    if (cached) {
      const data = JSON.parse(cached);
      // データが古すぎないかチェック
      const age = Date.now() - new Date(data.timestamp).getTime();
      if (age < this.TTL * 1000) {
        return data.result;
      }
    }
    
    return null;
  }
}
```

### 9.2 並列処理の最適化
```javascript
class ParallelJudgmentExecutor {
  /**
   * 複数企業の並列判定
   */
  executeParallel(announcement, companies) {
    const batchSize = 10; // 並列実行数
    const results = [];
    
    for (let i = 0; i < companies.length; i += batchSize) {
      const batch = companies.slice(i, i + batchSize);
      const batchResults = batch.map(company => {
        try {
          return {
            companyId: company.id,
            result: new JudgmentEngine().executeJudgment(
              announcement, 
              company
            )
          };
        } catch (error) {
          return {
            companyId: company.id,
            error: error.message
          };
        }
      });
      
      results.push(...batchResults);
      
      // 進捗更新
      if (i % 50 === 0) {
        Logger.log(`判定進捗: ${i}/${companies.length}`);
      }
    }
    
    return results;
  }
}
```

---

## 10. テスト戦略

### 10.1 単体テストケース
```javascript
// DisqualificationJudge のテスト例
function testDisqualificationJudge() {
  const judge = new DisqualificationJudge();
  
  // テストケース1: 全フラグFALSE
  const context1 = {
    company: {
      bankruptcy_flg: false,
      violent_relation_flg: false,
      // ... 他のフラグも全てfalse
    }
  };
  
  const result1 = judge.judge(context1);
  assert(result1.qualified === true, '全フラグFALSEは適格');
  
  // テストケース2: 単一フラグTRUE
  const context2 = {
    company: {
      bankruptcy_flg: true,
      violent_relation_flg: false,
      // ... 他のフラグはfalse
    }
  };
  
  const result2 = judge.judge(context2);
  assert(result2.qualified === false, '破産フラグTRUEは不適格');
  assert(result2.disqualificationReasons.length === 1, '理由は1つ');
}
```

### 10.2 統合テストシナリオ
1. 全要件適格パターン
2. 欠格要件NGパターン
3. 複数要件NGパターン
4. 境界値テスト
5. エラーハンドリングテスト

---

## 改訂履歴
- 2024-01-XX: 初版作成