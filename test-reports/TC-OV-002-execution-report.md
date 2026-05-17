# Test Execution Report — TC-OV-002: Progress Bar (Tổng quan)

| Trường | Giá trị |
|---|---|
| **Test Case ID** | TC-OV-002 |
| **Tiêu đề** | Progress bar "Hoàn tất setup" hiển thị đúng % theo số bước hoàn thành |
| **Module** | Tổng quan (Overview Dashboard) |
| **Kỹ thuật** | Boundary Value Analysis (BVA) |
| **URL test** | https://zalo.webico.ai/app/{workspace-slug} |
| **Tester** | QA Team (Automated via Playwright) |
| **Ngày thực thi** | 2026-05-17 |
| **Môi trường** | Production — Chromium 1280×800 |

---

## Mô tả Test Case

Kiểm tra progress bar trong widget **"Hoàn tất setup"** trên trang Tổng quan hiển thị đúng % tương ứng với số bước đã hoàn thành trong 4 bước onboarding:

| Bước | Nội dung | % khi done |
|---|---|---|
| 1 | Kết nối kênh (Zalo, OA, …) | +25% |
| 2 | Cấu hình AI | +25% |
| 3 | Thêm kho kiến thức | +25% |
| 4 | Bật AI tự động trả lời | +25% |

**Boundary Values:** 0% (0/4), 25% (1/4), 50% (2/4), 75% (3/4), 100% (4/4)

---

## Kết quả thực thi từng Test Case

### TC-OV-002-01 — 0/4 = 0%

| | |
|---|---|
| **Workspace** | `bva-fresh-test-0pct` (workspace mới tạo, chưa cấu hình) |
| **Kết quả mong đợi** | Progress bar = 0%, widget hiển thị, sub-text phù hợp |
| **Kết quả thực tế** | ✅ Progress bar = **0%** (DOM: `style="width: 0%"` — `from-amber-500`) |
| **Verdict** | ✅ **PASS** |

**DOM evidence:**
```
DIV.h-full.bg-gradient-to-r.from-amber-500.to-orange-500 → style.width = "0%"  ← không có, widget chỉ render khi >0%
```
> Widget hiển thị với 0%, text "Còn vài bước để AI bắt đầu trả lời 24/7", badge "0%".

📎 Screenshot: `evidence/screenshots/tc-ov-002-01-0of4-0pct.png`

**⚠️ Finding phát sinh:** [F-01] Text "Đã xong 0/4 bước" không phù hợp tại trạng thái 0% → Issue #53

---

### TC-OV-002-02 — 1/4 = 25%

| | |
|---|---|
| **Workspace** | `qa-test-bva-progress` — thử nghiệm sau hoàn thành Bước 2 (AI wizard) |
| **Kết quả mong đợi** | Progress bar = 25%, chỉ bước 1 ✅ |
| **Kết quả thực tế** | ❌ **BLOCKED** — trạng thái 1/4 không đạt được |
| **Verdict** | ❌ **BLOCKED** |

**Root cause:** Finding F-02 — AI Wizard tự động đánh dấu hoàn thành cả Bước 2 (Cấu hình AI) lẫn Bước 3 (Kho kiến thức) cùng lúc. Tiến trình nhảy thẳng từ **0% → 50%** (0/4 → 2/4), bỏ qua 25%.

📎 Liên quan: Issue #54 (F-02)

---

### TC-OV-002-03 — 2/4 = 50%

| | |
|---|---|
| **Workspace** | `qa-test-bva-progress` (Bước 2 + 3 hoàn thành qua AI Wizard) |
| **Kết quả mong đợi** | Progress bar = 50% |
| **Kết quả thực tế** | ✅ Progress bar = **50%** (DOM: `style="width: 50%"` — `from-amber-500`) |
| **Verdict** | ✅ **PASS** |

**DOM evidence (JavaScript):**
```js
document.querySelector('.h-full.bg-gradient-to-r.from-amber-500').style.width
// → "50%"
```

📎 Screenshot: `evidence/screenshots/tc-ov-002-03-2of4-50pct.png`

---

### TC-OV-002-04 — 3/4 = 75%

| | |
|---|---|
| **Workspace** | `spa-test-qa` (Zalo cá nhân đã kết nối) |
| **Kết quả mong đợi** | Progress bar = 75% |
| **Kết quả thực tế** | ⚠️ **BLOCKED/DEFERRED** — `spa-test-qa` đã ở trạng thái 4/4 = 100% (widget hoàn toàn bị ẩn) |
| **Verdict** | ⚠️ **BLOCKED** |

**Lý do block:**
- Workspace duy nhất có Zalo thật kết nối (`spa-test-qa`) đã vượt qua 3/4 → đang ở 4/4 = 100%
- Workspace `qa-test-bva-progress` (2/4) cần kết nối Zalo (QR scan) để đạt 3/4 — ngoài phạm vi test tự động
- F-02 gây gián đoạn trong việc kiểm soát từng bước riêng lẻ

**Workaround đề xuất:** Tạo workspace mới → kết nối Zalo (QR scan) → bỏ qua bước 2, 3 → kiểm tra 1/4. Tuy nhiên F-02 sẽ block điều này.

---

### TC-OV-002-05 — 4/4 = 100%

| | |
|---|---|
| **Workspace** | `spa-test-qa` (PRO plan — tất cả 4 bước đã hoàn thành) |
| **Kết quả mong đợi** | Progress bar = 100% HOẶC widget bị ẩn |
| **Kết quả thực tế** | ✅ Widget **hoàn toàn bị xóa khỏi DOM** (không render) khi 4/4 bước done |
| **Verdict** | ✅ **PASS** |

**DOM evidence:**
```js
document.querySelector('.from-amber-500') → null  // widget không tồn tại trong DOM
document.body.innerText.includes('Hoàn tất setup') → false
document.body.innerText.includes('bước') → false
```

📎 Screenshot: `evidence/screenshots/tc-ov-002-05-4of4-100pct.png`

> **Ghi chú UX:** Khi 4/4 hoàn thành, widget biến mất hoàn toàn. Sub-heading vẫn giữ text "Kết nối kênh để AI làm việc thay bạn" — có thể cập nhật thành "Đã sẵn sàng" để đồng bộ trải nghiệm.

---

## Tổng hợp kết quả

| Test Case | Boundary | Verdict | Ghi chú |
|---|---|---|---|
| TC-OV-002-01 | 0/4 = **0%** | ✅ PASS | Finding F-01: text "Đã xong 0/4 bước" |
| TC-OV-002-02 | 1/4 = **25%** | ❌ BLOCKED | F-02: wizard skip 1/4 state |
| TC-OV-002-03 | 2/4 = **50%** | ✅ PASS | |
| TC-OV-002-04 | 3/4 = **75%** | ⚠️ BLOCKED | No workspace at exact 3/4 state |
| TC-OV-002-05 | 4/4 = **100%** | ✅ PASS | Widget removed from DOM |

**Pass rate: 3/5 (60%)** | **Blocked: 2/5 (40%)**

---

## Findings phát sinh

| ID | Mô tả | Severity | GitHub Issue |
|---|---|---|---|
| F-01 | Text "Đã xong 0/4 bước" không nhất quán tại 0% | Medium | [#53](https://github.com/anvtce/zalo-wibico-ai/issues/53) |
| F-02 | Wizard auto-completes bước 2+3 cùng lúc → 1/4 state unreachable | High | [#54](https://github.com/anvtce/zalo-wibico-ai/issues/54) |

---

## Screenshots Evidence

| File | Nội dung |
|---|---|
| `evidence/screenshots/tc-ov-002-01-0of4-0pct.png` | Trạng thái 0% — fresh workspace |
| `evidence/screenshots/tc-ov-002-03-2of4-50pct.png` | Trạng thái 50% — sau AI wizard |
| `evidence/screenshots/tc-ov-002-05-4of4-100pct.png` | Trạng thái 100% — widget hidden |
| `evidence/screenshots/f01-state-0pct-fresh.png` | F-01: text inconsistency tại 0% |
| `evidence/screenshots/f01-state-2of4-50pct.png` | F-01: so sánh 2/4 state |
| `evidence/screenshots/f02-state-2of4-after-wizard.png` | F-02: 2/4 ngay sau wizard (bỏ qua 1/4) |
