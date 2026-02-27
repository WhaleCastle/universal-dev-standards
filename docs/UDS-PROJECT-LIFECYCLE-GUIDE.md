# UDS 大型專案開發生命週期指南

# UDS Project Lifecycle Guide for Large-Scale Projects

> **目標**：利用 Universal Development Standards (UDS) 的 Skills 與 Core Standards，從零開始完成一個大型專案的設計、開發、測試到上線投產。
>
> **適用對象**：技術主管、開發者、AI 輔助開發團隊

---

## 目錄

- [全局總覽](#全局總覽)
- [Phase 0：探索與構思](#phase-0探索與構思)
- [Phase 1：需求工程](#phase-1需求工程)
- [Phase 2：規格設計](#phase-2規格設計)
- [Phase 3：測試推導](#phase-3測試推導)
- [Phase 4：實作（單元測試驅動）](#phase-4實作單元測試驅動)
- [Phase 5：整合與端到端測試](#phase-5整合與端到端測試)
- [Phase 6：品質驗證](#phase-6品質驗證)
- [Phase 7：提交與版本管理](#phase-7提交與版本管理)
- [Phase 8：上線投產](#phase-8上線投產)
- [UDS 涵蓋範圍 vs 團隊職責](#uds-涵蓋範圍-vs-團隊職責)
- [多 Spec 大型專案管理策略](#多-spec-大型專案管理策略)
- [常見問題](#常見問題)

---

## 全局總覽

### 完整流程圖

```
Phase 0        Phase 1        Phase 2         Phase 3
探索與構思      需求工程        規格設計         測試推導
┌──────┐     ┌──────┐      ┌──────┐       ┌──────────┐
│/discover│──→│/require│──→ │ /sdd │──→    │/derive   │
│/brainstorm│ │-ment  │    │      │       │  all     │
└──────┘     └──────┘      └──────┘       └────┬─────┘
                                               │
                           ┌───────────────────┼───────────────────┐
                           ↓                   ↓                   ↓
                      TDD 骨架            BDD .feature        ATDD 場景
                      [TODO]              Gherkin              驗收條件
                           │                   │
Phase 4                    ↓         Phase 5   ↓          Phase 6
單元測試驅動          ┌──────┐       整合與 E2E       ┌──────────┐
                     │ /tdd │       ┌──────┐        │/coverage │
                     └──┬───┘       │ /bdd │        │/refactor │
                        │           └──┬───┘        │/review   │
                        ↓              ↓            │/checkin  │
                    ✅ Unit Tests  ✅ Integration   └──────────┘
                    ✅ 實作程式碼   ✅ E2E Tests
                                                         │
Phase 7                    Phase 8                        ↓
提交與版本管理              上線投產                   品質驗證通過
┌──────────┐          ┌──────────────┐
│/commit   │──→       │ 環境配置      │
│/changelog│          │ 部署 & 監控   │
│/release  │          │ 正式上線      │
└──────────┘          └──────────────┘
```

### 各階段與 UDS 指令對照

| Phase | 名稱 | UDS 指令 | 產出物 | 耗時佔比 |
|-------|------|----------|--------|----------|
| 0 | 探索與構思 | `/discover` `/brainstorm` | 評估報告、方案決策 | 5% |
| 1 | 需求工程 | `/requirement` | User Stories + AC | 10% |
| 2 | 規格設計 | `/sdd` | SPEC-XXX.md | 15% |
| 3 | 測試推導 | `/derive all` | .test.js + .feature | 5% |
| 4 | 單元測試實作 | `/tdd` | Unit Tests + 程式碼 | 25% |
| 5 | 整合 & E2E | `/bdd` | Step Definitions + E2E | 15% |
| 6 | 品質驗證 | `/coverage` `/refactor` `/review` `/checkin` | 審查報告 | 10% |
| 7 | 提交版本 | `/commit` `/changelog` `/release` | Git Tags, CHANGELOG | 5% |
| 8 | 上線投產 | （UDS 範圍外） | Production 環境 | 10% |

---

## Phase 0：探索與構思

> **目的**：了解要做什麼、可能遇到什麼風險、選擇最佳方案。

### 步驟

```bash
# Step 1：專案探索
/discover

# Step 2：方案構思（可選，需求不明確時使用）
/brainstorm "電商平台的支付系統設計"
```

### `/discover` 會做什麼

- 掃描現有程式碼結構
- 識別技術風險與依賴關係
- 評估專案健康狀態
- 產出評估報告

### `/brainstorm` 會做什麼

- 針對問題產生 3-5 個方案
- 用評分矩陣比較各方案
- 推薦最佳選項及理由

### 產出物

```
docs/
├── discovery-report.md      # 專案評估報告
└── brainstorm-decisions.md   # 方案決策記錄（可選）
```

### 進入下一階段的條件

- [ ] 清楚知道要建什麼系統
- [ ] 識別主要風險和依賴
- [ ] 如有多方案，已做出決策

---

## Phase 1：需求工程

> **目的**：將模糊的需求轉化為結構化的 User Story + Acceptance Criteria。

### 步驟

```bash
/requirement "用戶可以使用信用卡完成付款"
```

### `/requirement` 會做什麼

- 按 INVEST 原則撰寫 User Story
- 定義 Acceptance Criteria（驗收條件）
- 識別邊界條件和例外情況

### 產出物範例

```markdown
## User Story

As a 購物者
I want to 使用信用卡付款
So that 我可以完成訂單購買

## Acceptance Criteria

AC-1: 用戶輸入有效信用卡資訊後，付款成功，訂單狀態更新為「已付款」
AC-2: 信用卡資訊無效時，顯示具體錯誤訊息
AC-3: 付款超時（30 秒），顯示超時提示並允許重試
AC-4: 付款金額與訂單金額一致
AC-5: 付款成功後發送確認 email
```

### 大型專案的需求拆分

```
系統層級需求
├── 模組 A：用戶管理
│   ├── /requirement "用戶註冊"
│   ├── /requirement "用戶登入"
│   └── /requirement "密碼重設"
├── 模組 B：商品管理
│   ├── /requirement "商品上架"
│   └── /requirement "庫存管理"
└── 模組 C：訂單系統
    ├── /requirement "建立訂單"
    ├── /requirement "付款處理"
    └── /requirement "訂單追蹤"
```

### 進入下一階段的條件

- [ ] 所有功能都有對應的 User Story
- [ ] 每個 Story 有明確的 Acceptance Criteria
- [ ] 邊界條件和例外情況已識別

---

## Phase 2：規格設計

> **目的**：為每個功能模組建立詳細的技術規格書，作為實作的唯一真相來源。

### 步驟

```bash
# 針對每個功能模組建立 Spec
/sdd
```

AI 會引導你完成：
1. 定義 Spec 範圍
2. 描述 API / 介面設計
3. 定義資料結構
4. 列出所有 AC 的技術實現方式
5. 審核通過

### 產出物

```
docs/specs/
├── SPEC-001-user-registration.md
├── SPEC-002-user-authentication.md
├── SPEC-003-product-management.md
├── SPEC-004-order-creation.md
├── SPEC-005-payment-processing.md
└── SPEC-006-order-tracking.md
```

### Spec 審核流程

```
/sdd create  ──→  Draft（草稿）
                    ↓
/sdd review  ──→  Review（審核）
                    ↓
                 Approved（通過）  ──→ 可以開始 /derive
```

> **重要**：Spec 必須審核通過後才能進入下一階段。這是品質的關鍵閘門。

### 進入下一階段的條件

- [ ] 所有模組都有對應的 SPEC 文件
- [ ] 每份 Spec 狀態為 Approved
- [ ] API 介面、資料結構已明確定義

---

## Phase 3：測試推導

> **目的**：從 Approved Spec 自動產生測試骨架，為實作階段做準備。

### 步驟

```bash
# 從每份 Spec 推導測試
/derive all specs/SPEC-001-user-registration.md
/derive all specs/SPEC-002-user-authentication.md
# ... 每份 Spec 各跑一次
```

### `/derive all` 會做什麼

從一份 Spec 同時產生三種測試骨架：

```
/derive all SPEC-001
    │
    ├── TDD 骨架（.test.js）
    │   └── 帶有 [TODO] markers 的單元測試
    │
    ├── BDD 場景（.feature）
    │   └── Gherkin 格式的行為場景
    │
    └── ATDD 場景（驗收測試）
        └── 客戶可讀的驗收條件
```

### 產出物

```
tests/
├── unit/
│   ├── user-registration.test.js      # [TODO] markers
│   ├── user-authentication.test.js
│   └── payment-processing.test.js
├── features/
│   ├── user-registration.feature      # Gherkin 場景
│   ├── user-authentication.feature
│   └── payment-processing.feature
└── acceptance/
    └── ...
```

### 產出物內容範例

**TDD 骨架**（帶 `[TODO]`）：
```javascript
describe('SPEC-005: Payment Processing', () => {
  describe('AC-1: Successful payment', () => {
    test('should update order status to paid when payment succeeds', () => {
      // Arrange - [TODO] Set up order and payment info
      // Act - [TODO] Process payment
      // Assert - [TODO] Verify order status is "paid"
    });
  });
});
```

**BDD Feature**：
```gherkin
@SPEC-005 @AC-1
Scenario: Successful credit card payment
  Given a user has an order of $100
  And the user enters valid credit card information
  When the payment is processed
  Then the order status should be "paid"
  And a confirmation email should be sent
```

### 進入下一階段的條件

- [ ] 所有 Spec 都已產生測試骨架
- [ ] TDD 骨架包含所有 AC 的對應測試
- [ ] BDD Feature 場景覆蓋所有使用者行為

---

## Phase 4：實作（單元測試驅動）

> **目的**：用 TDD Red-Green-Refactor 循環，將測試骨架填入真正的測試和實作程式碼。

### 步驟

```bash
# 逐份 Spec 進行 TDD 實作
/tdd specs/SPEC-001-user-registration.md
/tdd specs/SPEC-002-user-authentication.md
# ... 每份 Spec 各跑一次
```

### `/tdd` 會做什麼（每個 AC 的循環）

```
🔴 RED      寫出/填入失敗測試 → 執行 → 確認失敗
                ↓
🟢 GREEN    寫最少的程式碼 → 執行 → 確認通過
                ↓
🔵 REFACTOR 改善程式碼結構 → 執行 → 確認仍通過
                ↓
            下一個測試案例...
```

### 此階段結束後的狀態

```
✅ 已完成                          ⚠️ 尚未完成
─────────────                     ─────────────
所有單元測試通過                     模組間整合測試
每個函式有對應測試                   端到端使用者流程
業務邏輯已實作                      外部服務真實串接
程式碼經過重構                      UI 整合
```

### Hardcoded 值的消除機制

TDD 的多案例會自然消除 hardcoding：

```
測試 1: validate("valid@email.com") → true     GREEN: return true（hardcode）
測試 2: validate("invalid")         → false    GREEN: 必須寫真正的驗證邏輯
測試 3: validate("")                → false    驗證邏輯正確處理空值
```

### 進入下一階段的條件

- [ ] 所有 `[TODO]` markers 已替換為真正的測試
- [ ] 所有單元測試通過
- [ ] 程式碼已經過 REFACTOR 階段的改善
- [ ] 無 hardcoded mock data 殘留在 production code

---

## Phase 5：整合與端到端測試

> **目的**：把各模組串接起來，驗證模組之間的溝通和完整使用者流程。

### 步驟

```bash
# 用 BDD 實作整合測試和 E2E 測試
/bdd specs/SPEC-001-user-registration.md
/bdd specs/SPEC-004-order-creation.md
# ... 每份涉及跨模組互動的 Spec
```

### `/bdd` 會做什麼

將 Phase 3 產出的 `.feature` 檔案接上可執行的 Step Definitions：

```
.feature（已存在）          Step Definitions（/bdd 產生）
─────────────────          ──────────────────────────
Given a user has           → 建立測試用戶、準備訂單資料
  an order of $100
When the payment           → 呼叫付款 API、等待回應
  is processed
Then the order status      → 查詢資料庫、驗證狀態欄位
  should be "paid"
```

### 測試金字塔對應

```
         /\
        /  \        E2E Tests (10%)
       / /bdd\      ← 完整使用者流程
      /────────\
     /          \   Integration Tests (20%)
    /   /bdd     \  ← 模組間串接
   /──────────────\
  /                \ Unit Tests (70%)
 /     /tdd         \ ← 個別函式邏輯
/────────────────────\
```

### 進入下一階段的條件

- [ ] Step Definitions 全部實作完成
- [ ] 整合測試通過（模組間呼叫正確）
- [ ] E2E 測試通過（使用者流程完整）
- [ ] 所有 .feature 場景都能執行

---

## Phase 6：品質驗證

> **目的**：全面檢查程式碼品質、測試覆蓋率、安全性，確保可以提交。

### 步驟

```bash
# Step 1：檢查測試覆蓋率（8 維度）
/coverage

# Step 2：改善程式碼品質
/refactor

# Step 3：程式碼審查
/review

# Step 4：提交前品質閘門
/checkin
```

### 各指令職責

| 指令 | 做什麼 | 產出 |
|------|--------|------|
| `/coverage` | 分析 8 維度測試覆蓋率、找出盲點 | 覆蓋率報告 + 補測建議 |
| `/refactor` | 識別 code smell、建議重構策略 | 重構後的程式碼 |
| `/review` | 10 類別系統化審查 | 審查報告（BLOCKING / IMPORTANT / SUGGESTION） |
| `/checkin` | 7 道品質閘門自動檢查 | Pass / Fail 報告 |

### `/coverage` 的 8 維度

```
1. 行覆蓋率 (Line)          — 每行程式碼都被測到
2. 分支覆蓋率 (Branch)       — 每個 if/else 都被測到
3. 函式覆蓋率 (Function)     — 每個函式都被呼叫
4. 路徑覆蓋率 (Path)         — 邏輯路徑組合
5. 邊界值 (Boundary)         — 邊界條件
6. 異常處理 (Exception)      — 錯誤路徑
7. 資料變異 (Mutation)       — 改程式碼後測試會不會抓到
8. 需求覆蓋率 (Requirement)  — AC 對應測試完整度
```

### `/checkin` 的 7 道閘門

```
Gate 1: Build       ── 編譯成功
Gate 2: Tests       ── 所有測試通過
Gate 3: Coverage    ── 覆蓋率未下降
Gate 4: Code Quality ── 符合 coding standards
Gate 5: Security    ── 無硬編碼密碼/API key
Gate 6: Documentation ── 文件已更新
Gate 7: Workflow    ── 分支命名、commit 格式正確
```

### 進入下一階段的條件

- [ ] 測試覆蓋率 ≥ 80%（建議 85%）
- [ ] 無 BLOCKING 級別的 review 問題
- [ ] `/checkin` 全部 7 道閘門通過
- [ ] 無安全漏洞

---

## Phase 7：提交與版本管理

> **目的**：規範化提交程式碼、記錄變更、管理版本發布。

### 步驟

```bash
# Step 1：產生規範的 commit message
/commit

# Step 2：更新變更日誌
/changelog

# Step 3：版本發布
/release
```

### Commit Message 格式（Conventional Commits）

```
feat(payment): add credit card payment processing

- Implement payment gateway integration
- Add card validation with Luhn algorithm
- Handle timeout and retry logic

Closes #42
```

### 版本發布策略（大型專案推薦）

```
Phase A: Alpha（內部驗證）
─────────────────────────
/release start 1.0.0-alpha.1
→ 內部團隊測試
→ 修復問題後 alpha.2, alpha.3...

Phase B: Beta（公開測試）
─────────────────────────
/release start 1.0.0-beta.1
→ 早期用戶測試
→ 收集回饋、修復問題

Phase C: Stable（正式上線）
─────────────────────────
/release start 1.0.0
→ 正式發布
→ npm publish / GitHub Release
```

### 進入下一階段的條件

- [ ] 所有程式碼已 commit 並 push
- [ ] CHANGELOG.md 已更新
- [ ] Git tag 已建立（vX.Y.Z）
- [ ] Alpha / Beta 測試通過（如適用）

---

## Phase 8：上線投產

> **目的**：將系統部署到 production 環境。
>
> ⚠️ **此階段超出 UDS 工具範圍**，由團隊的 DevOps / Infra 負責。

### UDS 完成後，上線前還需要的工作

| 類別 | 工作項目 | 說明 |
|------|----------|------|
| **環境配置** | `.env` / secrets | Production 環境變數、API keys、DB 連線 |
| **外部服務** | 真實串接 | 替換 mock 為真實的 DB、第三方 API、郵件服務 |
| **部署配置** | CI/CD pipeline | GitHub Actions / GitLab CI / Jenkins |
| **容器化** | Docker / K8s | Production 容器映像、編排配置 |
| **資料庫** | Schema migration | 建表、索引、seed data |
| **監控** | Logging & APM | 日誌收集、效能監控、告警 |
| **安全** | SSL / WAF / 掃描 | HTTPS、防火牆、OWASP 安全掃描 |
| **效能** | 壓力測試 | 負載測試、確認系統能承受預期流量 |
| **備份** | 備份策略 | 資料庫備份、災難復原計畫 |
| **文件** | 維運文件 | Runbook、故障排除手冊 |

### 上線檢查清單

```
□ Production 環境配置完成
□ 資料庫 migration 執行成功
□ 外部服務串接測試通過（真實環境）
□ SSL 憑證配置正確
□ 監控和告警設定完成
□ 備份策略已執行並驗證
□ 負載測試通過
□ 安全掃描無高風險漏洞
□ Rollback 計畫準備就緒
□ 團隊已進行上線演練
```

---

## UDS 涵蓋範圍 vs 團隊職責

```
┌─────────────────────────────────────────────────────────────┐
│                    UDS 涵蓋範圍                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Phase 0-7：探索 → 需求 → 規格 → 測試 → 實作 → 品質 → 發布│ │
│  │                                                        │ │
│  │ ✅ 需求分析        ✅ 程式碼品質      ✅ 版本管理        │ │
│  │ ✅ 規格設計        ✅ 測試覆蓋率      ✅ Commit 規範     │ │
│  │ ✅ 測試驅動開發    ✅ 程式碼審查      ✅ CHANGELOG       │ │
│  │ ✅ 業務邏輯實作    ✅ 安全基本檢查    ✅ 發布流程        │ │
│  └────────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│                    團隊自行負責                               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Phase 8：上線投產                                       │ │
│  │                                                        │ │
│  │ 🔧 環境配置        🔧 CI/CD Pipeline  🔧 容器化部署     │ │
│  │ 🔧 外部服務串接    🔧 效能壓測        🔧 安全掃描      │ │
│  │ 🔧 資料庫維運      🔧 監控告警        🔧 災難復原      │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 多 Spec 大型專案管理策略

### 開發順序規劃

大型專案有多份 Spec 時，不要同時全部開工，依**依賴關係**排序：

```
第 1 輪：基礎模組（無依賴）
├── SPEC-001: 用戶管理
└── SPEC-002: 基礎設施（DB, Cache, Logger）

第 2 輪：核心業務（依賴第 1 輪）
├── SPEC-003: 商品管理
└── SPEC-004: 購物車

第 3 輪：進階功能（依賴第 1+2 輪）
├── SPEC-005: 訂單系統
└── SPEC-006: 付款處理

第 4 輪：輔助功能
├── SPEC-007: 通知系統
└── SPEC-008: 報表分析
```

### 每一輪的完整循環

```bash
# 以第 1 輪為例

# 1. 推導測試
/derive all specs/SPEC-001-user-management.md
/derive all specs/SPEC-002-infrastructure.md

# 2. TDD 實作
/tdd specs/SPEC-001-user-management.md
/tdd specs/SPEC-002-infrastructure.md

# 3. BDD 整合
/bdd specs/SPEC-001-user-management.md

# 4. 品質驗證
/coverage
/review
/checkin

# 5. 提交
/commit

# → 進入第 2 輪
```

### 里程碑追蹤

| 里程碑 | 包含 Spec | 預期產出 | 驗收標準 |
|--------|-----------|----------|----------|
| M1：基礎 | SPEC-001, 002 | 用戶系統可運行 | 註冊/登入 E2E 通過 |
| M2：核心 | SPEC-003, 004 | 商品+購物車可操作 | 加入購物車 E2E 通過 |
| M3：交易 | SPEC-005, 006 | 完整購買流程 | 下單到付款 E2E 通過 |
| M4：完整 | SPEC-007, 008 | 全功能系統 | 全部 E2E 通過 |

---

## 常見問題

### Q1：整個流程一定要每個步驟都跑嗎？

不一定。根據情境選擇：

| 情境 | 建議路徑 |
|------|----------|
| 全新大型專案 | 完整 Phase 0-8（本文描述的流程） |
| 新增中型功能 | Phase 1-7（跳過 Phase 0） |
| 快速修 Bug | `/tdd` → `/checkin` → `/commit` |
| 維護舊系統 | `/discover` → `/reverse` → `/refactor` → `/tdd` |
| 緊急 Hotfix | 直接修復 → `/checkin` → `/commit`（事後補文件） |

### Q2：每個 UDS 指令都需要我手動輸入嗎？

只需輸入指令（如 `/tdd specs/SPEC-001.md`），AI 會自動執行整個循環。你的角色是：
- **下指令**
- **回答 AI 的問題**（如有）
- **審查產出結果**
- **方向修正**（如 AI 理解錯誤）

### Q3：`/tdd` 完成後 production code 裡會有 mock data 嗎？

不會。TDD 的多案例測試會自然消除 hardcoding。但**測試檔案**中的 mock 和 fixture 是正常且必要的。

### Q4：UDS 可以保證系統上線不出問題嗎？

UDS 保證的是 **程式碼品質和邏輯正確**。上線成功還取決於：
- 環境配置正確
- 外部服務串接正常
- 系統效能足夠
- 安全防護到位

這些屬於 Phase 8（團隊自行負責）的範圍。

### Q5：大型專案有多少份 Spec 是合理的？

取決於系統複雜度。參考值：

| 專案規模 | Spec 數量 | 開發週期 |
|----------|-----------|----------|
| 小型（MVP） | 3-5 份 | 2-4 週 |
| 中型 | 8-15 份 | 1-3 個月 |
| 大型 | 20-50 份 | 3-12 個月 |
| 企業級 | 50+ 份 | 6 個月以上 |

---

## 快速參考卡

```
/discover        → 我要做什麼？有什麼風險？
/brainstorm      → 有哪些方案？哪個最好？
/requirement     → 用戶要什麼？怎樣算完成？
/sdd             → 技術上怎麼做？API 長什麼樣？
/derive all      → 自動產生測試骨架
/tdd             → 寫測試 → 寫程式碼 → 重構（單元層級）
/bdd             → 整合測試 + E2E 測試（系統層級）
/coverage        → 測試夠完整嗎？哪裡有盲點？
/refactor        → 程式碼可以更好嗎？
/review          → 程式碼審查，找問題
/checkin         → 提交前最後檢查
/commit          → 規範化 commit message
/changelog       → 更新變更日誌
/release         → 版本發布
```

---

> **本文件版本**：1.0
>
> **適用 UDS 版本**：5.0.0+
>
> **最後更新**：2026-02-27
