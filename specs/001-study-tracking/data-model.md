# Data Model: 讀書時間與成果紀錄系統

**Feature**: 001-study-tracking  
**Date**: 2025-11-10

---

## Entity Relationships

```
Subject (科目)
    ├── isDefault: boolean (預設科目)
    ├── outcomeUnit: string (成果測量單位)
    └── 1 → N relationship with StudySession

StudySession (學習記錄)
    ├── subjectId: FK → Subject.id
    ├── timeSpent: number (分鐘)
    ├── effortLevel: number (1-5)
    ├── outcome: number | string (依科目單位)
    └── Timestamps: createdAt, updatedAt
```

---

## Core Entities

### 1. StudySession (學習記錄)

**存儲位置**: IndexedDB (`studySessions` store)

**索引**:
- Primary: `id` (UUID)
- Secondary: `date` (日期查詢)
- Secondary: `subjectId` (按科目篩選)
- Secondary: `createdAt` (時間排序)

**字段定義**:

| 欄位 | 類型 | 必填 | 驗證規則 | 備註 |
|------|------|------|--------|------|
| `id` | string (UUID) | ✓ | 唯一 | 系統自動生成 |
| `createdAt` | number (ms) | ✓ | > 0 | 建立時間戳 |
| `updatedAt` | number (ms) | ✓ | >= createdAt | 最後編輯時間戳 |
| `date` | string (YYYY-MM-DD) | ✓ | 有效日期 | 記錄日期 |
| `subjectId` | string | ✓ | FK exists | 科目 ID |
| `timeSpent` | number | ✓ | 1 ~ 480 (> 480 警告) | 分鐘數 |
| `effortLevel` | number | ✓ | 1 ~ 5 (integer) | 職業付出度 |
| `outcome` | number \| string | ✓ | > 0, 依科目單位 | 成果數值 |
| `notes` | string | ✗ | max 500 chars | 可選備註 |

**驗證規則 (雙層)**:

```javascript
// 前端驗證 (即時)
validateStudySession(data) {
  return {
    id: isUUID(data.id),
    date: isValidDate(data.date) && isFutureOrToday(data.date),
    subjectId: isExistingSubject(data.subjectId),
    timeSpent: isNumber(data.timeSpent) && data.timeSpent > 0,
    effortLevel: isInteger(data.effortLevel) && data.effortLevel >= 1 && data.effortLevel <= 5,
    outcome: isNumber(data.outcome) || isString(data.outcome),
    notes: data.notes === undefined || (isString(data.notes) && data.notes.length <= 500)
  }
}

// 保存前驗證 (持久化)
validateBeforeSave(data) {
  const errors = [];
  if (!data.timeSpent || data.timeSpent <= 0) errors.push('投入時間必須 > 0');
  if (data.timeSpent > 480) console.warn('投入時間 > 480 分鐘, 顯示警告');
  if (data.effortLevel < 1 || data.effortLevel > 5) errors.push('職業付出度必須在 1-5 之間');
  if (data.outcome === null || data.outcome === '') errors.push('成果不能為空');
  return errors.length === 0;
}
```

**示例**:

```javascript
{
  id: "550e8400-e29b-41d4-a716-446655440000",
  createdAt: 1699608000000,
  updatedAt: 1699608000000,
  date: "2025-11-10",
  subjectId: "s01",
  timeSpent: 120,           // 2 小時
  effortLevel: 4,           // 好
  outcome: 25,              // 25 頁 (英文科目)
  notes: "複習第 5 章語法"
}
```

---

### 2. Subject (科目)

**存儲位置**: 
- 預設科目: 代碼寫死 (config.js)
- 自訂科目: IndexedDB (`subjects` store)

**索引**:
- Primary: `id` (UUID)
- Unique: `name` (科目名稱唯一)

**字段定義**:

| 欄位 | 類型 | 必填 | 驗證規則 | 備註 |
|------|------|------|--------|------|
| `id` | string (UUID) | ✓ | 唯一 | 預設科目固定 ID, 自訂科目系統生成 |
| `name` | string | ✓ | 1-50 chars, 唯一 | 科目名稱 |
| `isDefault` | boolean | ✓ | - | 是否為預設科目 |
| `isHidden` | boolean | ✓ | - | 用戶隱藏標記 (默認 false) |
| `outcomeUnit` | string | ✓ | 預設單位或自訂 | 成果測量單位 |
| `createdAt` | number (ms) | ✓ | > 0 | 建立時間戳 |
| `updatedAt` | number (ms) | ✓ | >= createdAt | 編輯時間戳 |

**預設科目清單** (config.js 寫死):

```javascript
const DEFAULT_SUBJECTS = [
  { id: 's01', name: '英文', outcomeUnit: '頁數', isDefault: true, isHidden: false },
  { id: 's02', name: '數學', outcomeUnit: '習題數', isDefault: true, isHidden: false },
  { id: 's03', name: '程式設計', outcomeUnit: '分數', isDefault: true, isHidden: false },
  { id: 's04', name: '日語', outcomeUnit: '頁數', isDefault: true, isHidden: false },
  { id: 's05', name: '商業英文', outcomeUnit: '分數', isDefault: true, isHidden: false },
  { id: 's06', name: '歷史', outcomeUnit: '章節數', isDefault: true, isHidden: false },
  { id: 's07', name: '地理', outcomeUnit: '章節數', isDefault: true, isHidden: false },
  { id: 's08', name: '物理', outcomeUnit: '習題數', isDefault: true, isHidden: false },
  { id: 's09', name: '化學', outcomeUnit: '習題數', isDefault: true, isHidden: false },
  { id: 's10', name: '生物', outcomeUnit: '頁數', isDefault: true, isHidden: false }
];
```

**預設成果單位清單**:

```javascript
const OUTCOME_UNITS = [
  '分數',    // 0-100
  '習題數',  // 正整數
  '頁數',    // 正整數
  '章節數',  // 正整數
  '計時',    // 時間 (分鐘)
  '其他'     // 自訂單位
];
```

**驗證規則**:

```javascript
// 新增或編輯科目
validateSubject(data) {
  return {
    name: isString(data.name) && data.name.length > 0 && data.name.length <= 50,
    outcomeUnit: OUTCOME_UNITS.includes(data.outcomeUnit) || isCustomUnit(data.outcomeUnit),
    isHidden: isBoolean(data.isHidden),
    uniqueName: !subjectNameExists(data.name, data.id) // 編輯時排除自身
  }
}

// 科目唯一性檢查
subjectNameExists(name, excludeId = null) {
  const allSubjects = [...DEFAULT_SUBJECTS, ...customSubjects];
  return allSubjects.some(s => s.name === name && s.id !== excludeId);
}
```

**示例**:

```javascript
// 預設科目
{
  id: "s01",
  name: "英文",
  isDefault: true,
  isHidden: false,
  outcomeUnit: "頁數",
  createdAt: 1699608000000,
  updatedAt: 1699608000000
}

// 自訂科目
{
  id: "c-550e8400-e29b-41d4-a716-446655440000",
  name: "韓文",
  isDefault: false,
  isHidden: false,
  outcomeUnit: "頁數",
  createdAt: 1699608000000,
  updatedAt: 1699608000000
}
```

---

## Storage Layer Architecture

### LocalStorage (快取層)

**目的**: 快速讀寫, 同步操作, 減少 IndexedDB 查詢

**存儲內容**:
- 最近 100 筆記錄 (`recentSessions`)
- 科目清單 + 隱藏狀態 (`subjects`)
- 用戶偏好設置 (`userPreferences`): 暗色模式, 排序方式等

**示例**:

```javascript
localStorage.setItem('recentSessions', JSON.stringify([
  { id: '...', date: '2025-11-10', subjectId: 's01', ... },
  // ... 最多 100 筆
]));

localStorage.setItem('subjects', JSON.stringify([
  { id: 's01', name: '英文', isHidden: false, ... },
  // 包括預設科目 + 自訂科目
]));

localStorage.setItem('userPreferences', JSON.stringify({
  darkMode: false,
  sortBy: 'date-desc',
  itemsPerPage: 20
}));
```

### IndexedDB (持久化層)

**目的**: 大容量存儲, 支援 1000+ 記錄, 結構化查詢

**數據庫**: `StudyTrackingDB` (v1.0)

**Object Stores**:

```javascript
// studySessions store
{
  keyPath: 'id',
  indexes: [
    { name: 'date', unique: false },
    { name: 'subjectId', unique: false },
    { name: 'createdAt', unique: false }
  ]
}

// subjects store (自訂科目)
{
  keyPath: 'id',
  indexes: [
    { name: 'name', unique: true }
  ]
}

// backups store (用戶備份)
{
  keyPath: 'id',
  indexes: [
    { name: 'timestamp', unique: false }
  ]
}
```

---

## Query Patterns

### 按日期範圍查詢記錄

```javascript
// 查詢 2025-11-01 到 2025-11-10 的記錄
queryByDateRange(startDate, endDate) {
  const start = new Date(startDate).getTime();
  const end = new Date(endDate).getTime() + 86400000; // +1 天
  
  return db.studySessions
    .where('createdAt').between(start, end)
    .toArray();
}
```

### 按科目篩選

```javascript
queryBySubject(subjectId) {
  return db.studySessions
    .where('subjectId').equals(subjectId)
    .toArray();
}
```

### 最近 N 筆記錄

```javascript
getRecentSessions(limit = 20) {
  return db.studySessions
    .orderBy('createdAt')
    .reverse()
    .limit(limit)
    .toArray();
}
```

---

## Analysis Queries

### 計算投入時間 vs. 成果的相關性

```javascript
calculateCorrelation() {
  const sessions = db.studySessions.toArray();
  
  if (sessions.length < 2) {
    return { status: 'INSUFFICIENT_DATA' };
  }
  
  // 提取時間和成果序列
  const timeSpent = sessions.map(s => s.timeSpent);
  const outcomes = sessions.map(s => parseFloat(s.outcome));
  
  // 計算皮爾遜相關係數
  const correlation = pearsonCorrelation(timeSpent, outcomes);
  
  // 計算趨勢線 (線性迴歸)
  const trendLine = linearRegression(timeSpent, outcomes);
  
  // 等級評分
  const rating = getCorrelationRating(correlation);
  
  return {
    pearsonCoefficient: correlation,
    r2Value: trendLine.r2,
    trendLine: trendLine,
    rating: rating // 極弱/弱/中等/強/極強
  };
}
```

### 按科目計算相關性

```javascript
calculateCorrelationBySubject(subjectId) {
  const sessions = db.studySessions
    .where('subjectId').equals(subjectId)
    .toArray();
    
  // 同上述邏輯
}
```

---

## Data Integrity & Validation

### 遠端同步保留位置

雖然系統無後端，但保留以下結構便於未來擴展:

```javascript
// 備份與恢復
{
  id: 'backup-timestamp',
  timestamp: number,
  data: {
    sessions: [],
    subjects: [],
    preferences: {}
  },
  version: '1.0.0'
}
```

### 資料匯出格式

**JSON 格式**:
```json
{
  "version": "1.0.0",
  "exportDate": "2025-11-10T06:52:21.391Z",
  "subjects": [...],
  "sessions": [...]
}
```

**CSV 格式**:
```csv
日期,科目,投入時間(分鐘),職業付出度(1-5),成果,備註
2025-11-10,英文,120,4,25 頁,複習第 5 章語法
```

---

## Migration Strategy

### 版本升級路徑

```javascript
// v1.0 → v1.1 (Future)
// - 新增 tags 欄位
// - 保留現有記錄相容性
// - 自動迁移腳本
```

---

**版本**: 1.0.0  
**狀態**: Finalized  
**作成日期**: 2025-11-10
