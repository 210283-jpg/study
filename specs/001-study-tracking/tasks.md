---
description: "Task list for 讀書時間與成果紀錄系統 implementation"
---

# Tasks: 讀書時間與成果紀錄系統

**Input**: Design documents from `/specs/001-study-tracking/`  
**Prerequisites**: plan.md ✅, spec.md ✅, data-model.md ✅

**Organization**: Tasks are grouped by user story and phase to enable independent implementation and testing.

---

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3, US4, US5)

---

## Phase 1: Setup (Project Infrastructure)

**Purpose**: Initialize project structure and dependencies

- [x] **T001** Initialize project structure per plan.md
  - Create directories: `assets/css/`, `assets/js/`, `assets/libs/`, `assets/icons/` ✅
  - Create base HTML files: `index.html` ✅ (record.html, history.html, analysis.html pending)
  - Create `.gitignore` with standard patterns (pending)

- [x] **T002** [P] Setup global CSS framework
  - Create `assets/css/global.css` with responsive grid/flexbox baseline ✅
  - Add CSS variables for colors, spacing, typography ✅
  - Implement dark mode support (`@prefers-color-scheme: dark`) ✅
  - Ensure WCAG 2.1 AA contrast ratios (min 4.5:1 for normal text) ✅

- [x] **T003** [P] Setup component CSS
  - Create `assets/css/components/buttons.css` ✅
  - Create `assets/css/components/cards.css` ✅
  - forms.css and modals.css pending (needed for next phases)

- [x] **T004** [P] Setup JavaScript infrastructure
  - Create `assets/js/config.js` ✅
  - Create `assets/js/utils/formatters.js` ✅
  - Create `assets/js/utils/validation.js` ✅

- [ ] **T005** Download & integrate ECharts library
  - Download ECharts minified version (~1.3MB) - pending
  - Place at `assets/libs/echarts.min.js`
  - Verify library loads correctly in browser console

- [ ] **T006** Create README.md with project overview
  - Add feature description, technical stack, local development instructions
  - Include Accessibility guidelines link

- [ ] **T007** Create ACCESSIBILITY.md with a11y implementation details
  - Document WCAG 2.1 AA compliance strategy
  - Include keyboard navigation guide, ARIA labels, testing procedures

**Checkpoint**: ✅ Project structure ready, CSS framework functional, JS infrastructure in place. Phase 2 ready to begin.

---

## Phase 2: Core Data & Storage Infrastructure

**Purpose**: Implement data models and storage layers (prerequisite for all user stories)

- [x] **T008** Implement IndexedDB database layer (`assets/js/db.js`)
  - Create `StudyTrackingDB` database with version 1.0 ✅
  - Define Object Stores: `studySessions`, `subjects` (custom), `backups` ✅
  - Add indices: date, subjectId, createdAt, name (unique) ✅
  - Implement connection pooling and error handling ✅
  - Add database schema validation on open ✅

- [x] **T009** [P] Implement LocalStorage cache layer (`assets/js/storage.js`)
  - Create `LocalStorageCache` class for quick read/write ✅
  - Implement `recentSessions` (last 100 records) cache ✅
  - Implement `subjects` cache (predefined + custom) ✅
  - Implement `userPreferences` cache ✅
  - Add cache invalidation on IndexedDB updates ✅

- [x] **T010** [P] Create StudySession model (`assets/js/models/StudySession.js`)
  - Implement class with all fields ✅
  - Add UUID generation method ✅
  - Add timestamp management (createdAt/updatedAt) ✅
  - Add validation method ✅

- [x] **T011** [P] Create Subject model (`assets/js/models/Subject.js`)
  - Implement class with all fields ✅
  - Add method to check subject existence ✅
  - Add method to mark subject as hidden/visible ✅

- [x] **T012** Implement StudyService (`assets/js/services/StudyService.js`)
  - Implement CRUD operations ✅
  - Implement query methods (getByDateRange, getBySubject) ✅
  - All methods include error handling and validation ✅

- [x] **T013** [P] Implement SubjectService (`assets/js/services/SubjectService.js`)
  - Implement getDefaults(), getCustom(), create(), hide(), show(), delete() ✅
  - Implement getAll() with visibility filtering ✅

- [x] **T014** [P] Implement AnalysisService (`assets/js/services/AnalysisService.js`)
  - Implement calculateCorrelation() with Pearson coefficient ✅
  - Implement calculateCorrelationBySubject() ✅
  - Implement getTrendLine() and rating system ✅
  - Include getDashboardStats() for metrics ✅

- [x] **T015** [P] Create utility functions for analysis (`assets/js/utils/correlation.js`)
  - Implement pearsonCorrelation(x, y) ✅ (in AnalysisService)
  - Implement linearRegression(x, y) ✅ (in AnalysisService)
  - All functions handle edge cases ✅

**Checkpoint**: ✅ All data models, storage layers, and services functional and tested. Phase 3 ready to begin.

---

## Phase 3: User Story 1 - Record Single Study Session (Priority: P1) 🎯 MVP

**Goal**: Enable users to quickly record a single study session with validation

**Independent Test**: User can input all fields, see validation errors, and save a valid record

### Tests for User Story 1

- [ ] **T016** [P] [US1] Unit tests for StudySession validation in browser console
  - Test valid input passes validation
  - Test invalid timeSpent (0, negative, > 480 with warning)
  - Test invalid effortLevel (< 1, > 5, non-integer)
  - Test invalid outcome (empty, wrong type)
  - Test missing required fields

- [ ] **T017** [P] [US1] Integration test: Record creation flow
  - Test form submission → validation → IndexedDB save → cache update
  - Test localStorage cache reflects new record
  - Test repeated opens show persisted record

### Implementation for User Story 1

- [ ] **T018** [P] [US1] Create RecordForm component (`assets/js/ui/RecordForm.js`)
  - Build form HTML with fields: date, subject, timeSpent, effortLevel, outcome, notes
  - Add real-time validation on change (show error messages)
  - Implement timer button for outcome field (click to start/stop counting)
  - Add warning when timeSpent > 480 minutes (yellow border + message)
  - Add 「Save」 and 「Cancel」 buttons
  - All inputs must have proper `<label>` associations and ARIA attributes
  - Implement dark mode support via CSS variables

- [ ] **T019** [P] [US1] Create record.html page
  - Add semantic HTML structure: `<header>`, `<main>`, `<footer>`
  - Include page title and breadcrumb navigation
  - Embed RecordForm component
  - Add loading indicator (for async saves)
  - Include success/error message display area
  - Link to global CSS and JS files

- [ ] **T020** [US1] Implement record-page.js logic (`assets/js/pages/record-page.js`)
  - Initialize RecordForm on page load
  - Handle form submission:
    - Call dual validation (front-end + before save)
    - Show validation errors if any
    - If > 480 min: confirm before saving
    - Save via StudyService.create()
  - On success: show confirmation message + redirect to history.html after 1.5s
  - On error: show error message with retry option
  - Implement keyboard shortcuts (Ctrl+S to save, Esc to cancel)

- [ ] **T021** [US1] Add subject dropdown to RecordForm
  - Fetch all subjects (defaults + custom) from SubjectService
  - Sort alphabetically
  - Show outcome unit as helper text next to outcome field
  - Implement search/filter if > 10 subjects
  - Use `<select>` for accessibility, with keyboard navigation support

- [ ] **T022** [US1] Implement timer feature for outcome field
  - Add 「Start Timer」 button next to outcome input
  - Click button → timer starts, button changes to 「Stop」
  - Timer updates every second in MM:SS format
  - Click 「Stop」 → record elapsed time as outcome value (converted to appropriate unit)
  - Store elapsed time in minutes, format for display
  - Include pause feature (click 「Pause」 to pause, 「Resume」 to continue)

- [ ] **T023** [US1] Add save validation & error handling
  - Implement `validateBeforeSave()` with all rules from data-model.md
  - Test edge cases: timeSpent = 0, outcome = empty, subjectId = invalid
  - Display user-friendly error messages in form
  - Prevent save if validation fails

**Checkpoint**: Users can create and save a single study session; record persists across page reloads

---

## Phase 4: User Story 2 - View Learning History (Priority: P1)

**Goal**: Display all saved study sessions in a sortable, filterable list

**Independent Test**: User can see all records in a list, filter by subject/date range, and view record details

### Tests for User Story 2

- [ ] **T024** [P] [US2] Unit tests for StudyService queries
  - Test `getAll()` returns all records
  - Test `getByDateRange(start, end)` returns correct subset
  - Test `getBySubject(id)` returns correct subset
  - Test pagination with limit parameter

- [ ] **T025** [P] [US2] Integration test: History page loading & filtering
  - Test page loads and displays all records
  - Test subject filter updates list correctly
  - Test date range filter updates list correctly
  - Test combined filters work together

### Implementation for User Story 2

- [ ] **T026** [P] [US2] Create RecordList component (`assets/js/ui/RecordList.js`)
  - Display records in table format: Date, Subject, Time, Effort, Outcome, Notes
  - Add row click handler to show details
  - Implement pagination (20 records per page)
  - Add virtual scrolling for 1000+ records (lazy load as user scrolls)
  - Include delete button (with confirmation) on each row
  - Add edit button → navigate to record.html with ID pre-filled
  - Ensure table is responsive (horizontal scroll on mobile)

- [ ] **T027** [P] [US2] Create filter controls component (`assets/js/ui/FilterControls.js`)
  - Subject dropdown filter (fetch from SubjectService)
  - Date range picker (start date, end date)
  - 「Clear Filters」 button
  - Add keyboard navigation and ARIA labels
  - Emit 'filter-change' event when filters updated

- [ ] **T028** [US2] Create history.html page
  - Add semantic HTML: `<header>`, `<main>`, `<nav>`
  - Include page title and navigation links
  - Embed FilterControls component
  - Embed RecordList component
  - Add empty state message (if no records)
  - Include export button (CSV/JSON)

- [ ] **T029** [US2] Implement history-page.js logic (`assets/js/pages/history-page.js`)
  - Initialize RecordList and FilterControls on load
  - Fetch all records on page load via StudyService.getAll()
  - Listen to 'filter-change' events from FilterControls
  - On filter change:
    - Fetch filtered records (date range, subject)
    - Re-render RecordList with new data
  - Handle record delete:
    - Show confirmation dialog
    - Call StudyService.delete(id)
    - Update list
  - Handle record edit:
    - Save record ID to localStorage
    - Navigate to record.html
  - Implement sort options (by date, by time spent, by effort level)

- [ ] **T030** [US2] Add empty state UI
  - Show friendly message when no records exist
  - Include button linking to record.html (「記錄第一次學習」)
  - Add illustration/icon for visual appeal

**Checkpoint**: Users can view all records, filter by date/subject, and manage records (delete, edit)

---

## Phase 5: User Story 3 - Visualize Learning Efficiency (Priority: P1)

**Goal**: Generate scatter plot showing time spent vs. outcome with correlation analysis

**Independent Test**: System renders chart correctly, displays correlation metrics, updates on data changes

### Tests for User Story 3

- [ ] **T031** [P] [US3] Unit tests for AnalysisService
  - Test `calculateCorrelation()` with sample data
  - Test correlation rating mapping (coefficient → 極弱/弱/中等/強/極強)
  - Test linear regression R² calculation
  - Test < 2 records returns `{ status: 'INSUFFICIENT_DATA' }`

- [ ] **T032** [P] [US3] Integration test: Chart rendering
  - Test chart renders with valid data
  - Test chart updates when new records added
  - Test subject filter updates chart correctly

### Implementation for User Story 3

- [ ] **T033** [P] [US3] Create ChartComponent (`assets/js/ui/ChartComponent.js`)
  - Initialize ECharts instance with scatter plot
  - X-axis: Time Spent (minutes)
  - Y-axis: Outcome (value varies by subject)
  - Plot data points as scatter
  - Add trend line overlay (via linearRegression)
  - Show tooltip on hover: (time, outcome, date, subject)
  - Color-code points by subject
  - Add legend (clickable to toggle subject visibility)
  - Support dark mode (auto-detect from CSS variables)
  - Implement zoom + pan features
  - Handle resize events for responsive rendering

- [ ] **T034** [P] [US3] Create correlation analysis component (`assets/js/ui/CorrelationAnalysis.js`)
  - Display three analysis metrics in card layout:
    1. Pearson Coefficient: r value with interpretation
    2. Trend Line: R² value + visual representation
    3. Rating: 極弱/弱/中等/強/極強 with description
  - Include 「Insufficient Data」 message if < 2 records
  - Color-code results (red for weak, green for strong correlation)
  - Add explanation tooltips for each metric

- [ ] **T035** [US3] Create analysis.html page
  - Add semantic HTML structure
  - Include page title: 「學習效率分析」
  - Add subject filter (same as history)
  - Embed ChartComponent
  - Embed CorrelationAnalysis
  - Add empty state when no records
  - Include date range filter for analysis scope

- [ ] **T036** [US3] Implement analysis-page.js logic (`assets/js/pages/analysis-page.js`)
  - Initialize chart and analysis components on load
  - Fetch all sessions via StudyService.getAll()
  - Call AnalysisService.calculateCorrelation() → display results
  - Listen to filter changes:
    - Re-fetch filtered sessions
    - Update chart data
    - Recalculate correlation
  - Handle filter by subject: recalculate correlation for that subject
  - Implement loading animation while calculating (correlation computation might take time for 1000+ records)

- [ ] **T037** [US3] Add data export functionality (`assets/js/utils/export.js`)
  - Implement `exportToJSON(sessions, subjects)`: create JSON with all data
  - Implement `exportToCSV(sessions)`: create CSV with headers and rows
  - Create download link and trigger browser download
  - Add export button to analysis page
  - Test export with various data sizes

**Checkpoint**: Users see scatter plot with correlation analysis; data visualizations update dynamically

---

## Phase 6: User Story 4 - Edit & Delete Records (Priority: P2)

**Goal**: Allow users to modify or remove records with audit trail

**Independent Test**: Users can edit records (with timestamp updates), delete with confirmation, verify changes persist

### Tests for User Story 4

- [ ] **T038** [P] [US4] Unit tests for StudyService update/delete
  - Test `update()` changes record and updates `updatedAt` timestamp
  - Test `delete()` removes record from IndexedDB
  - Test validation still applies on update

- [ ] **T039** [P] [US4] Integration test: Edit & delete flows
  - Test clicking edit on history page loads record in form
  - Test submitting changes updates record
  - Test delete confirmation dialog
  - Test deleted record no longer in list/chart

### Implementation for User Story 4

- [ ] **T040** [US4] Enhance record.html for edit mode
  - Detect if record ID in URL or localStorage (from history page)
  - If editing: pre-fill form with existing data
  - Show 「Last edited: [timestamp]」 under form title
  - Change submit button text to 「Update」 instead of 「Save」
  - Add delete button (with confirmation)

- [ ] **T041** [US4] Implement edit flow in record-page.js
  - On page load: check for record ID
  - If ID found:
    - Fetch record via StudyService.read(id)
    - Populate form with data
    - Switch to edit mode (different UI cues)
  - On submit:
    - Call StudyService.update(id, data) instead of create
    - Verify `updatedAt` timestamp is newer than `createdAt`
  - On delete:
    - Show confirmation dialog (「確定要刪除此記錄嗎?」)
    - Call StudyService.delete(id)
    - Redirect to history.html with success message

- [ ] **T042** [US4] Add bulk delete with confirmation
  - Add checkboxes to RecordList component
  - Add bulk delete button (only visible if rows selected)
  - Show confirmation dialog (「確定要刪除 N 筆記錄嗎?」)
  - Delete all selected records
  - Update list and analysis chart

**Checkpoint**: Users can edit records (with audit timestamps) and delete records with confirmation

---

## Phase 7: User Story 5 - Dashboard & Insights (Priority: P2)

**Goal**: Display key metrics and insights on dashboard

**Independent Test**: Dashboard calculates and displays statistics correctly; updates when records change

### Tests for User Story 5

- [ ] **T043** [P] [US5] Unit tests for dashboard calculations
  - Test `getWeeklyStats()`: calculate this week's total time and avg outcome
  - Test `getHighestEfficiencySubject()`: find subject with best correlation
  - Test `getStreak()`: count consecutive study days

### Implementation for User Story 5

- [ ] **T044** [P] [US5] Create DashboardWidget component (`assets/js/ui/DashboardWidget.js`)
  - Display 4 key metrics in card grid:
    1. This Week Total Time (hours:minutes format)
    2. Average Outcome Score (across all subjects)
    3. Highest Efficiency Subject (name + correlation coefficient)
    4. Study Streak (consecutive days with records)
  - Add visual icons/badges for each metric
  - Implement responsive card layout (2x2 on desktop, 1x4 on mobile)
  - Color-code metrics (red for warning, green for good, blue for neutral)

- [ ] **T045** [US5] Create index.html (Dashboard page)
  - Add semantic HTML structure with `<header>`, `<main>`
  - Add page title: 「學習儀表板」
  - Include navigation to other pages (record, history, analysis)
  - Embed DashboardWidget
  - Include quick action buttons:
    - 「+ 新增記錄」 → record.html
    - 「查看歷史」 → history.html
    - 「分析趨勢」 → analysis.html
  - Add greeting message (「歡迎回來, [date]」)

- [ ] **T046** [US5] Implement index-page.js logic (`assets/js/pages/index-page.js`)
  - Initialize DashboardWidget on load
  - Fetch all sessions and calculate statistics:
    - `getWeeklyStats(sessions)`: filter this week's sessions, sum time, avg outcome
    - `getHighestEfficiencySubject(sessions)`: calculate correlation per subject, return top one
    - `getStreak(sessions)`: count consecutive study days (backwards from today)
  - Display metrics in DashboardWidget
  - Refresh metrics every 5 minutes (if page stays open)
  - Add animation when metrics update

**Checkpoint**: Dashboard displays all key metrics; users get quick insight into study progress

---

## Phase 8: Polish & Optimization

**Purpose**: Final quality assurance, performance tuning, accessibility validation

- [ ] **T047** Create navigation header component (`assets/js/ui/Header.js`)
  - Add logo and site title
  - Add navigation menu (4 pages: Dashboard, Record, History, Analysis)
  - Highlight current page
  - Add dark mode toggle button
  - Responsive hamburger menu on mobile
  - Keyboard navigation support

- [ ] **T048** [P] Test all pages for responsive design
  - Test on desktop (1920x1080), tablet (768x1024), mobile (375x667)
  - Verify no horizontal scroll
  - Check touch target sizes (min 44x44px)
  - Verify text readability at all sizes

- [ ] **T049** [P] Accessibility audit (WCAG 2.1 AA)
  - Check color contrast ratios (min 4.5:1 for normal text)
  - Verify keyboard navigation (Tab, Enter, Arrow keys, Esc)
  - Test with screen reader (NVDA or JAWS)
  - Verify ARIA labels on dynamic content
  - Check focus indicators visibility

- [ ] **T050** [P] Performance optimization
  - Run Lighthouse audit (target > 80 on all metrics)
  - Minimize CSS and inline critical styles
  - Defer non-critical JavaScript
  - Optimize images/icons (use SVG where possible)
  - Test load time with 1000+ records (target < 2s)
  - Test chart rendering time (target < 1s)

- [ ] **T051** [P] Cross-browser testing
  - Test on Chrome (latest)
  - Test on Firefox (latest)
  - Test on Safari (latest)
  - Test on Edge (latest)
  - Verify localStorage and IndexedDB work on all browsers
  - Check for console errors

- [ ] **T052** Implement error handling & logging
  - Add try-catch to all async operations
  - Log errors to browser console (with context)
  - Show user-friendly error messages (not technical stack traces)
  - Implement retry mechanism for failed saves

- [ ] **T053** [P] Create comprehensive inline documentation
  - Add JSDoc comments to all functions
  - Document data flow diagrams in README
  - Create troubleshooting guide in ACCESSIBILITY.md
  - Add API-like documentation for all services

- [ ] **T054** Final quality checklist
  - Verify all 17 functional requirements (FR-001 to FR-017) implemented
  - Verify all 15 success criteria (SC-001 to SC-015) met
  - Verify all user stories (US1-US5) fully functional
  - Verify no browser console errors
  - Verify no broken links or 404s
  - Test with real data (create 50+ records manually)

- [ ] **T055** Create deployment guide
  - Document how to deploy to GitHub Pages
  - Create `.gitignore` with appropriate patterns
  - Add GitHub Pages specific meta tags (if needed)
  - Test deployment on GitHub Pages

**Checkpoint**: Production-ready static website, all tests passing, full accessibility compliance

---

## Task Dependencies

```
Phase 1 (Setup)
    ↓
Phase 2 (Storage Infrastructure)
    ├→ Phase 3 (US1: Record)
    │    ↓
    ├→ Phase 4 (US2: History) [depends on US1]
    │    ↓
    ├→ Phase 5 (US3: Analysis) [depends on US2]
    │    ↓
    ├→ Phase 6 (US4: Edit/Delete) [depends on US2]
    │
    ├→ Phase 7 (US5: Dashboard) [depends on US1]
    │
    └→ Phase 8 (Polish & Optimization) [depends on all phases]
```

## Parallel Execution Opportunities

**Phase 1 Setup** (T002-T007 can run in parallel):
- T002, T003, T004 are independent CSS/JS setup tasks
- T005 library download independent of T002-T004
- All can complete in parallel before Phase 2

**Phase 2 Storage** (T009-T015 can run partially in parallel):
- T009 (LocalStorage) independent of T008 (IndexedDB)
- T010, T011 (models) independent of each other
- T012, T013, T014, T015 (services) can partially overlap

**Phase 3, 4, 5, 6, 7** can mostly run in parallel after Phase 2 completes, since:
- Each user story affects different HTML pages
- Services don't conflict (reading same data, writing to different sessions)
- Analysis (US3) depends on records existing (US1), but not on UI details
- Dashboard (US5) only depends on StudyService

**Recommended Parallel Groups**:
1. All Phase 1 tasks (except T007 depends on T005)
2. T008 + T009 (storage layers) in parallel
3. T010, T011 (models) in parallel
4. T012, T013, T014, T015 (services) - mostly sequential but T012-T013 parallelizable
5. T018-T023 (US1 UI), T026-T030 (US2 UI), T033-T037 (US3 UI) can overlap
6. T047-T054 (Polish) can run as cleanup at end

---

## Implementation Strategy

### MVP First (Phase 1-5): Core Functionality
- Phases 1-2: Foundation
- Phase 3: Single record creation (MVP core)
- Phase 4: View history (MVP support)
- Phase 5: Visualization (MVP value)

### Incremental Delivery
- Deploy Phase 1-5 as MVP v1.0
- Then add Phase 6 (Edit/Delete) as v1.1
- Then add Phase 7 (Dashboard) as v1.2
- Phase 8 (Polish) ongoing throughout

### Acceptance Criteria by Phase

**Phase 1**: ✅ Project structure created, CSS framework working, JS modules loadable
**Phase 2**: ✅ Can create, read, update, delete records in IndexedDB/localStorage
**Phase 3**: ✅ Can record a single study session with validation
**Phase 4**: ✅ Can view history with filters and see previously saved records
**Phase 5**: ✅ Can see scatter plot and correlation analysis
**Phase 6**: ✅ Can edit records (with timestamps) and delete with confirmation
**Phase 7**: ✅ Dashboard displays 4 key metrics
**Phase 8**: ✅ All lighthouse scores > 80, WCAG AA compliant, all 15 success criteria met

---

**版本**: 1.0.0  
**狀態**: Ready for Implementation  
**作成日期**: 2025-11-10  
**總任務數**: 55  
**預估工時**: 40-60 小時 (depends on parallelization)
