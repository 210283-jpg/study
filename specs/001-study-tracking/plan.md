# Implementation Plan: 讀書時間與成果紀錄系統

**Branch**: `001-study-tracking` | **Date**: 2025-11-10 | **Spec**: `/specs/001-study-tracking/spec.md`

**Input**: Feature specification from `/specs/001-study-tracking/spec.md`

---

## Summary

建立一個互動式靜態網頁工具，幫助學習者追蹤學習投入與成果。系統採用多頁應用 (MPA) 架構，每個功能模塊為獨立 HTML 頁面。使用 LocalStorage 快取 + IndexedDB 持久化雙層架構存儲數據。前端採用 Vanilla JS + ECharts 圖表庫實現視覺化分析。

---

## Technical Context

**Language/Version**: HTML5 + CSS3 + Vanilla JavaScript (ES2020+)  
**Primary Dependencies**: 
- ECharts (圖表視覺化)
- IndexedDB (數據持久化)
- LocalStorage (快取層)

**Storage**: LocalStorage (快取) + IndexedDB (持久化)  
**Testing**: 瀏覽器控制台測試 + 集成測試  
**Target Platform**: 現代瀏覽器 (Chrome, Firefox, Safari, Edge)  
**Project Type**: 靜態網站 (GitHub Pages)  
**Performance Goals**: LCP < 2 秒, 支援 1000+ 筆記錄  
**Constraints**: 
- 首頁加載 < 3 秒
- 無後端依賴
- WCAG 2.1 AA 可訪問性標準

**Scale/Scope**: 單用戶應用，支援本地存儲 1000+ 記錄

---

## Constitution Check

✅ **以靜態前端為本**: 純靜態網站，GitHub Pages 可部署，無後端依賴  
✅ **可訪問性優先**: 遵循 WCAG 2.1 AA，語意化 HTML，ARIA 標籤，鍵盤導航  
✅ **響應式設計**: Flexbox/Grid，流動式佈局，所有螢幕尺寸適配  
✅ **效能為要**: LCP 目標 < 2 秒，支援大量記錄  
✅ **簡潔與可維護性**: KISS 原則，模組化代碼，完整文檔

---

## Project Structure

### Documentation (this feature)

```text
specs/001-study-tracking/
├── spec.md                      # ✅ 功能規格
├── plan.md                      # 📍 本文件 (技術計劃)
├── data-model.md                # 📍 數據模型 (本計劃生成)
├── checklists/
│   └── requirements.md          # ✅ 需求質量檢查
└── tasks.md                     # 📍 實作任務 (tasks 指令生成)
```

### Source Code (repository root)

```text
/workspaces/study/
├── index.html                   # 主頁 & 儀表板
├── record.html                  # 新增/編輯記錄
├── history.html                 # 歷史記錄檢視
├── analysis.html                # 分析 & 相關性
├── assets/
│   ├── css/
│   │   ├── global.css           # 全域樣式, 響應式設計
│   │   ├── dark-mode.css        # 暗色模式
│   │   └── components/          # 組件樣式 (buttons, forms, cards)
│   ├── js/
│   │   ├── config.js            # 全域配置 (科目清單, 驗證規則)
│   │   ├── db.js                # IndexedDB 管理
│   │   ├── storage.js           # LocalStorage 快取層
│   │   ├── models/
│   │   │   ├── StudySession.js  # 學習記錄模型
│   │   │   └── Subject.js       # 科目模型
│   │   ├── services/
│   │   │   ├── StudyService.js  # 記錄 CRUD 操作
│   │   │   ├── SubjectService.js # 科目管理
│   │   │   └── AnalysisService.js # 分析與相關性計算
│   │   ├── ui/
│   │   │   ├── RecordForm.js    # 記錄表單組件
│   │   │   ├── RecordList.js    # 記錄清單組件
│   │   │   ├── ChartComponent.js # 圖表組件
│   │   │   └── EmptyState.js    # 空狀態提示
│   │   ├── utils/
│   │   │   ├── validation.js    # 數據驗證 (雙層)
│   │   │   ├── correlation.js   # 皮爾遜相關係數計算
│   │   │   ├── export.js        # JSON/CSV 匯出
│   │   │   └── formatters.js    # 日期、數字格式化
│   │   └── pages/
│   │       ├── record-page.js   # record.html 邏輯
│   │       ├── history-page.js  # history.html 邏輯
│   │       ├── analysis-page.js # analysis.html 邏輯
│   │       └── index-page.js    # index.html 邏輯
│   ├── libs/
│   │   └── echarts.min.js       # ECharts 圖表庫
│   └── icons/
│       └── [SVG icons]          # 圖標資源
├── .gitignore
├── README.md
└── ACCESSIBILITY.md
```

**結構決定**: MPA (Multi-Page Application) 架構，每個功能模塊為獨立 HTML 頁面。使用模組化 JS 代碼共享邏輯，靜態資源按需加載。

---

## Data Model

### StudySession (學習記錄)
```javascript
{
  id: string,                     // UUID
  createdAt: number,              // 建立時間戳 (ms)
  updatedAt: number,              // 編輯時間戳 (ms)
  date: string,                   // YYYY-MM-DD
  subjectId: string,              // 科目 ID (ref: Subject)
  timeSpent: number,              // 分鐘數 (> 0)
  effortLevel: number,            // 職業付出度 (1-5)
  outcome: number | string,       // 成果 (依科目單位)
  notes: string                   // 可選備註
}
```

### Subject (科目)
```javascript
{
  id: string,                     // UUID
  name: string,                   // 科目名稱 (唯一)
  isDefault: boolean,             // 是否為預設科目
  isHidden: boolean,              // 用戶隱藏標記
  outcomeUnit: string,            // 成果測量單位 (分數、習題數、頁數、章節數、計時)
  createdAt: number,              // 建立時間戳
  updatedAt: number               // 編輯時間戳
}
```

### DefaultSubjects (預設科目清單, 寫死代碼)
```javascript
[
  { id: 's01', name: '英文', outcomeUnit: '頁數' },
  { id: 's02', name: '數學', outcomeUnit: '習題數' },
  { id: 's03', name: '程式設計', outcomeUnit: '分數' },
  { id: 's04', name: '日語', outcomeUnit: '頁數' },
  { id: 's05', name: '商業英文', outcomeUnit: '分數' },
  { id: 's06', name: '歷史', outcomeUnit: '章節數' },
  { id: 's07', name: '地理', outcomeUnit: '章節數' },
  { id: 's08', name: '物理', outcomeUnit: '習題數' },
  { id: 's09', name: '化學', outcomeUnit: '習題數' },
  { id: 's10', name: '生物', outcomeUnit: '頁數' }
]
```

---

## Technology Decisions

### 為何選擇 IndexedDB + LocalStorage?
- LocalStorage: 快速讀寫, 同步操作, 快取層
- IndexedDB: 大容量 (GB 級), 非同步, 支援 1000+ 記錄, 結構化查詢

### 為何選擇 ECharts?
- 功能豐富 (散點圖, 折線圖, 趨勢線, 相關性視覺化)
- 輕量級 (全量版 ~1.3MB)
- 響應式與暗色模式支援
- 豐富的互動功能 (tooltip, zoom, 數據點點擊)

### 為何選擇 MPA 而非 SPA?
- 符合「靜態網站」定位
- 每頁獨立加載, 性能更優
- SEO 友善
- 代碼模塊化，便於維護

### 雙層驗證策略
- **前端驗證**: 用戶提交時實時驗證 (快速反饋)
- **保存驗證**: 存儲前二次驗證 (數據完整性)

### 相關性分析三層指標
1. **皮爾遜相關係數** (-1 ~ +1): 線性相關程度
2. **趨勢線擬合**: R² 值, 視覺化趨勢
3. **等級評分**: 簡化版相關性描述 (極弱/弱/中等/強/極強)

---

## Deployment Strategy

### 開發環境
```bash
# 本地測試
python3 -m http.server 8000
# 訪問 http://localhost:8000
```

### 部署至 GitHub Pages
```bash
git add .
git commit -m "Feature: 讀書時間與成果紀錄系統"
git push origin 001-study-tracking
# 創建 PR 合併至 main
# main 分支自動部署至 GitHub Pages
```

---

## Performance Optimization

### 加載性能
- 圖片優化 (WebP 格式, SVG 圖標)
- CSS 關鍵路徑內聯, 延遲加載非關鍵 CSS
- JS 分離 (庫代碼獨立, 頁面代碼延遲加載)
- 壓縮資源 (gzip)

### 運行時性能
- 分頁顯示歷史記錄 (每頁 20 筆, 虛擬滾動)
- 圖表按需渲染 (避免隱藏圖表渲染)
- IndexedDB 異步查詢, 不阻塞 UI
- 計算相關性時顯示加載動畫

### 存儲優化
- LocalStorage 快取最近 100 筆記錄
- IndexedDB 存儲完整數據 (支援 1000+)
- 定期清理過期快取 (>30 天)

---

## Accessibility Standards (WCAG 2.1 AA)

### 語意化 HTML
- 使用 `<header>`, `<main>`, `<nav>`, `<footer>`
- 表單使用 `<label>` 關聯
- 列表使用正確的 `<ul>`, `<ol>`, `<li>`

### 鍵盤導航
- Tab 導航順序合理
- Enter/Space 觸發按鈕
- Esc 關閉對話框
- 焦點指示器清晰

### 顏色與對比度
- 文字對比度 >= 4.5:1 (常規文字)
- 互動元素對比度 >= 3:1
- 不僅用顏色傳達信息 (支援暗色模式)

### ARIA 標籤
- 表單驗證錯誤使用 `aria-invalid`
- 動態內容使用 `aria-live`
- 圖表使用 `aria-label` 描述

---

## Risk Mitigation

| 風險 | 可能性 | 影響 | 緩解策略 |
|------|--------|------|--------|
| IndexedDB 配額超出 | 低 | 高 | 實現數據分頁、定期清理機制 |
| 跨瀏覽器兼容性 | 中 | 中 | 測試現代瀏覽器, 提供向下相容方案 |
| 大數據集性能下降 | 中 | 中 | 分頁、虛擬滾動、非同步計算 |
| 暗色模式適配 | 低 | 低 | 完整的 CSS 變數支援 |

---

## Testing Strategy

### 功能測試
- 記錄創建、編輯、刪除
- 篩選、搜尋功能
- 圖表渲染、相關性計算
- 資料匯出

### 性能測試
- Lighthouse 審計 (目標 > 80)
- 加載時間 (< 2 秒)
- 記錄數量極限測試 (1000+)

### 可訪問性測試
- 鍵盤導航完整性
- 螢幕閱讀器測試
- 顏色對比度檢查

### 跨瀏覽器測試
- Chrome, Firefox, Safari, Edge (最新版)
- 手機瀏覽器 (iOS Safari, Android Chrome)

---

## Success Criteria

- ✅ 首次加載時間 < 2 秒 (Lighthouse LCP)
- ✅ 支援 1000+ 筆記錄無明顯延遲
- ✅ Lighthouse 評分 > 80
- ✅ WCAG 2.1 AA 完全合規
- ✅ 所有使用者故事 P1 優先級完成
- ✅ 資料同步無損

---

**版本**: 1.0.0  
**狀態**: Ready for Implementation  
**作成日期**: 2025-11-10
