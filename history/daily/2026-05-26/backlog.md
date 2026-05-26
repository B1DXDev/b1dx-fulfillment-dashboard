# Product Backlog — b1dx OMS Fulfillment

> อัพเดทล่าสุด: 2026-05-01 — archived done tasks to 2026-05-backlog.md
> Source: docs/mockups/b1dx_full_app_v0.html + feature requests + spec analysis
> Sprint Plan: `tasks/sprints/sprint-plan.md`

---

## Priority Legend

| Priority | ความหมาย                                                            |
| -------- | ------------------------------------------------------------------- |
| `P0`     | Prerequisite — infrastructure ที่ทุก feature ต้องพึ่ง (ไม่มีไม่ได้) |
| `P1`     | Core — ต้องมีก่อน ระบบใช้งานจริงไม่ได้ถ้าขาด                        |
| `P2`     | Important — สำคัญ build เร็วๆ หลัง P1 เสร็จ                         |
| `P3`     | Nice-to-have — ทำทีหลังได้ ไม่ block production                     |

---

## GROUP 0 — Foundation Prerequisites `P0`

> ✅ All Done — ดู tasks/archive/2026-04-backlog.md

---

## GROUP 0a — Observability (ERP Sync) `P2`

> **Spec:** `docs/specs/observability-grafana-local.md` ✅ Ready for implementation
> **Effort:** 4.5h (5 subtasks)
> **Goal:** Structured logging + OTel metrics for ERP→OMS sync pipeline
> **Resilience:** Worker operates normally if observability stack fails (graceful degradation)

### [P2] S2-ERP-08d: Local Grafana Stack — docker-compose.yml `[1h]`

**Goal:** Infrastructure-as-code for local development — Loki (logs), Prometheus (metrics), Grafana (visualization)
**Type:** DevOps / Infrastructure
**Spec Ref:** `docs/specs/observability-grafana-local.md` § Local Docker Compose Setup

**Files to Create:**

- `docker-compose.yml` — 3 services (Loki, Prometheus, Grafana) + volumes
- `.docker/loki-config.yml`
- `.docker/prometheus.yml`
- `.docker/grafana/datasources.yml`

**Acceptance Criteria:**

- [ ] `docker-compose.yml` created with 3 services: Grafana, Prometheus, Loki
- [ ] Grafana accessible at `http://localhost:3000`
- [ ] Prometheus accessible at `http://localhost:9090`
- [ ] Loki accessible at `http://localhost:3100`
- [ ] All configs use relative paths (portable across machines)

**Dependencies:** None — infrastructure-only
**Effort estimate:** 1h

---

### [P2] S2-ERP-08e: Worker Serilog + OTel Integration `[1h]`

**Goal:** Wire Serilog GrafanaLoki sink + OTel Prometheus exporter into worker background service with graceful degradation (timeout/fallback)
**Type:** BE — Sync Worker Infrastructure
**Spec Ref:** `docs/specs/observability-grafana-local.md` § Worker Integration & Resilience

**Acceptance Criteria:**

- [ ] Worker starts and logs to console + Loki simultaneously
- [ ] Structured logs appear in Grafana Loki UI
- [ ] Metrics appear in Prometheus
- [ ] Serilog Loki sink has 5s timeout with try-catch fallback
- [ ] OTel Prometheus exporter has 5s timeout with try-catch fallback
- [ ] Stop Loki container: worker logs continue to console, no exceptions in log
- [ ] `dotnet build && dotnet test` passes

**Dependencies:** S2-ERP-08a, S2-ERP-08b, S2-ERP-08c, S2-ERP-08d
**Effort estimate:** 1h

---

## GROUP 1 — Auth Foundation `P1`

> ✅ Done items archived → tasks/archive/2026-04-backlog.md

### [IMPROVE][P2] Email Templates — รองรับ TH/EN ตามภาษาที่ผู้ใช้เลือก

**Type:** BE + Email Template
**Goal:** `password-reset.html` และ `otp-verification.html` แสดงภาษาตาม preference ของผู้ใช้
**Current state:** hardcode ภาษาไทยทั้งหมด ไม่มี locale field ใน AppUser/UserProfile
**Scope:**

1. เพิ่ม `PreferredLocale` (`"th"` | `"en"`, default `"th"`) ใน `UserProfile` domain entity + migration
2. ส่ง `locale` เข้า template model ใน `SendPasswordResetEmailAsync` + `SendOtpEmailAsync`
3. ปรับ template ทั้งสองให้ใช้ Liquid `{% if locale == "en" %}` แยก TH/EN strings
4. FE ต้องเก็บ locale ที่ผู้ใช้เลือกและส่งไปอัพเดท UserProfile
   **Depends on:** UserProfile update endpoint

---

### [IMPROVE][P2] Reset Password Email — ปรับ theme เป็น light

**Type:** Email Template
**Symptom:** `password-reset.html` ใช้ dark theme (พื้นหลังดำ/navy) — อ่านยากใน Gmail light mode
**Expected:** default เป็น light theme — พื้นขาว ข้อความดำ contrast ชัดเจน
**File:** `src/B1dx.OmsFulfillment.Infrastructure/Email/Templates/password-reset.html`

---

### [BUG][P1] Reset Password — Link redirect ไม่ถูกต้อง

**Type:** FE
**Symptom:** คลิก link ใน email → redirect ไปหน้า login แทนหน้า reset password
**Expected:** `/reset?token=...` → แสดงหน้า ResetPasswordForm
**Actual:** redirect ไปหน้า login
**Area:** Next.js routing — middleware หรือ `(auth)` route guard อาจบล็อก `/reset` route ก่อน token ถูก read

---

### [P1] Forgot / Reset Password BE — Missing

**Notes:**

- FE (ForgotPasswordForm wire, ResetPasswordForm wire) — ✅ Done (Sprint 1)
- BE (3 endpoints + DB table: S1-06 to S1-09) — ❌ Missing (in progress)

---

## GROUP 2 — Order Management `P1`

> ✅ All Done — ดู tasks/archive/2026-04-backlog.md

---

## GROUP 2 — ERP Order Sync (oms_erp_db → OmsDb) `P1`

> **Spec:** `docs/specs/sync-order-erp.md` ✅ Ready
> **Working dir:** `sync_worker_repo_path` จาก `workspace.local.json`

### [P2] S2-ERP-06: FulfillmentRequest Auto-Create ❌

**Type:** Sync Worker — `ErpOrderRepository.cs`
**Effort:** S | **Depends on:** S2-ERP-03, S2-OD-02

---

## GROUP 2 — Marketplace Sync Worker (Remaining Tasks) `P1–P2`

### [P3] S2-MSY-07: tracking_no → oms.shipments ❌

**Depends on:** S2-OD-02 (shipments table ต้องมีก่อน), S2-MSY-03e

> **Full task detail** สำหรับทุก sub-task ของ S2-MSY-03, S2-MSY-04, S2-MSY-08 อยู่ใน archive:
> `tasks/archive/2026-04-backlog.md` § GROUP 2.5 (Remaining Infrastructure sections)

### GROUP 2b — Bulk Actions `P2`

**Spec:** [`docs/specs/order-bulk-actions.md`](../docs/specs/order-bulk-actions.md) ✅
**Mockup:** `docs/mockups/order-list-full-improve.html`
**Foundation:** `FIX-FE-ORDER-STATE-CODES` (bulk bar + canDo rules) — merged 2026-05-05

> FEAT-BULK-01, 02, 04, 07 เริ่มได้เลย — BE endpoints พร้อมแล้ว
> FEAT-BULK-06 ขึ้นกับ FEAT-BULK-05 (BE export endpoint)

---

### [P2] FEAT-BULK-01 — FE: Bulk Confirm

**Type:** `[FE]` | **Effort:** S ~1-2h
**Spec:** `order-bulk-actions.md § WF-01`
**Scope:**
- เมื่อ user คลิก "ยืนยัน" — ไม่มี dialog, call tr直接
- loop `POST /api/oms/orders/{id}/transitions { toState: 400 }` สำหรับแต่ละ order ที่ eligible
- toast "ยืนยันแล้ว N orders" + clear selection + refresh
- partial failure: toast "สำเร็จ X/N" + error detail
**Test Cases:**
- [ ] เลือก orders status=100 → confirm → status เปลี่ยนเป็น 400
- [ ] เลือก orders status=300 → confirm → status เปลี่ยนเป็น 400
- [ ] บางใบ fail → partial toast แสดงจำนวนที่สำเร็จ

---

### [P2] FEAT-BULK-02 — FE: BulkPrintDialog + print_ticket flow

**Type:** `[FE]` | **Effort:** S ~1-2h
**Spec:** `order-bulk-actions.md § WF-02`
**Scope:**
- BulkPrintDialog (mode='ticket'): format radio A4/A5/Thermal 80mm + submit button
- loop `POST /api/oms/orders/{id}/print { jobType:"ticket", format }` สำหรับ eligible orders
- toast "สร้าง Print Job แล้ว N ใบ" — ไม่ clear selection
**Test Cases:**
- [ ] เลือก orders status=400 → print ticket dialog แสดง
- [ ] เลือก format A4 → submit → N print jobs ถูก create
- [ ] orders status ที่ไม่ eligible ถูก skip

---

### [P2] FEAT-BULK-03 — FE: print_label flow (size/weight input)

**Type:** `[FE]` | **Effort:** S ~1-2h
**Spec:** `order-bulk-actions.md § WF-03`
**Depends:** FEAT-BULK-02 (reuse BulkPrintDialog mode='label')
**Scope:**
- BulkPrintDialog (mode='label'): size radio 10×10/4×6 + weight input (optional)
- loop `POST /api/oms/orders/{id}/print { jobType:"label", format, weightKg? }`
- toast "Label N ใบ กำลังพิมพ์"
**Test Cases:**
- [ ] เลือก orders status=450/480/490 → label dialog แสดง
- [ ] กรอก weight → submit → weightKg ถูกส่งใน body
- [ ] ไม่กรอก weight → submit → weightKg undefined (BE ใช้ค่าเดิม)

---

### [P2] FEAT-BULK-04 — FE: BulkTrackPanel — side panel tracking list

**Type:** `[FE]` | **Effort:** M ~3-4h
**Spec:** `order-bulk-actions.md § WF-04`
**Scope:**
- Side panel เปิดทางขวา แสดง list tracking ของ orders ที่ eligible
- parallel `GET /api/oms/orders/{id}/tracking` สำหรับแต่ละ order
- แสดง: orderNo | courier | trackingNumber | status | ลิงก์เปิด courier website (new tab)
- loading skeleton, error per row
- ปิด panel → selection ยังอยู่
**Test Cases:**
- [ ] เลือก orders + trackingNumber → panel เปิด + แสดง tracking rows
- [ ] คลิกลิงก์ courier → เปิด new tab
- [ ] order ที่ไม่มี trackingNumber → ไม่แสดงใน panel

---

### [P2] FEAT-BULK-05 — BE: `POST /api/oms/orders/export`

**Type:** `[BE]` | **Effort:** M ~3-4h
**Spec:** `order-bulk-actions.md § API Contract`
**Scope:**
- endpoint `POST /api/oms/orders/export`
- รับ `{ ids: string[] }` หรือ `{ filters: OrderFilters }`
- Accept header: `text/csv` หรือ `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`
- return file blob + `Content-Disposition: attachment`
- permission: `orders:view`
- max 1000 rows ต่อ export
**Test Cases:**
- [ ] POST ids → return CSV ที่มี rows ตรงกับ ids
- [ ] POST filters → return CSV filtered
- [ ] Accept xlsx → return Excel file
- [ ] ids > 1000 → 400 error
- [ ] ไม่มี permission → 403

---

### [P2] FEAT-BULK-06 — FE: BulkExportDialog + download flow

**Type:** `[FE]` | **Effort:** S ~1-2h
**Spec:** `order-bulk-actions.md § WF-05`
**Depends:** FEAT-BULK-05
**Scope:**
- BulkExportDialog: format radio CSV/Excel + scope radio selected/all-filtered
- call `POST /api/oms/orders/export` → trigger browser download via blob URL
- toast "Export เสร็จแล้ว"
**Test Cases:**
- [ ] เลือก CSV + selected → download .csv
- [ ] เลือก Excel + all → download .xlsx
- [ ] export 0 orders → toast error

---

### [P2] FEAT-BULK-07 — FE: BulkCancelDialog + cancel flow

**Type:** `[FE]` | **Effort:** M ~3-4h
**Spec:** `order-bulk-actions.md § WF-06`
**Scope:**
- BulkCancelDialog: header "ยืนยันยกเลิก N orders?", reason dropdown (required), detail textarea (optional)
- ปุ่ม "ยืนยันยกเลิก" สีแดง
- loop `POST /api/oms/orders/{id}/transitions { toState: 800, cancelReason, cancelDetail? }`
- toast "ยกเลิกแล้ว N orders" + clear selection + refresh
- orders ที่ terminal (700/650/800) ถูก skip โดย canDoBulkAction
**Test Cases:**
- [ ] คลิก "ยกเลิก" → dialog เปิด
- [ ] ไม่กรอก reason → submit disabled
- [ ] กรอก reason → confirm → orders เปลี่ยน status=800
- [ ] orders terminal ถูก skip, นับใน X/N toast

---

## GROUP 3 — Access Control `P1`

> Done items archived → tasks/archive/2026-04-backlog.md

### [P1] User Management (Add / Edit / Detail)

**Goal:** Admin จัดการ User ในระบบ — เพิ่ม, แก้ไข, ดูรายละเอียด, deactivate
**Screens:** `ac-users`
**Spec:** `docs/specs/user-management.md` ✅
**Notes:** Spec ครบ — Sprint 4 (ตรวจสอบ build status ก่อน → complete ส่วนที่ขาด)

### [P2] User Form: Multiple Role Assignments

**Goal:** Add/Edit user drawer รองรับ multiple roles
**Type:** FE only — `UserFormDrawer.tsx`, `userForm.ts`, `useAddUser.ts`, `useUpdateUser.ts`, MSW
**Detail:**

- Replace single `roleId` dropdown → dynamic list of role assignments (add/remove rows)
- Each row: role select + conditional scope (brand/warehouse) + remove button
- At least 1 role required
- Schema: `roleAssignments: { roleId, brandId?, warehouseId? }[]`
- API payload: `roleAssignments[]` instead of flat `roleId/brandId/warehouseId`

### [P1] S2-RBAC-05: FE — Permission Editor — dirty state + Save + unsaved confirm ❌

**Goal:** PermissionEditor บันทึกได้จริงผ่าน API
**Type:** FE only | **Effort:** S | **Depends on:** S2-RBAC-04
**Steps:**

1. Track dirty state per role column (copy initial state → compare on toggle)
2. Save → `PUT /api/user/roles/{id}/permissions` สำหรับแต่ละ role ที่ dirty
3. Success toast + reset dirty state
4. Unsaved-changes confirm dialog (`AlertDialog`) เมื่อออกจากหน้า/เปลี่ยน tab
5. `pnpm typecheck` ผ่าน

### [P2] S2-RBAC-06: FE + BE — Role Overview cards เพิ่ม user count ❌

**Goal:** card แต่ละ role แสดงจำนวน user ที่ใช้ role นั้น
**Type:** BE + FE | **Effort:** XS
**Steps (BE):** เพิ่ม `userCount: int` ใน `RoleResponse` — `LEFT JOIN user_memberships GROUP BY role_key`
**Steps (FE):** แสดง user count badge ใน `RoleOverviewCards.tsx`

### [P2] S2-RBAC-08b: FE — Users page — Role Quick-Filter Chips ❌

**Goal:** แทน dropdown ด้วย chip row ตาม mockup
**Type:** FE only | **Effort:** XS | **Depends on:** ไม่มี
**Spec:** chips `[ทั้งหมด | FF Member | Brand Admin | WH Admin | WH Worker | FF Manager]` — `sys-admin` hidden
**Steps:**

1. Replace role dropdown → chip row ใน `app/(app)/users/page.tsx`
2. Active chip: `bg-primary text-white`, inactive: `border`
3. Click chip → update `roleKey` URL param + reset `page=1`

### [P2] S2-RBAC-10: FE — Users page — Bulk Action Bar ❌

**Goal:** เลือกหลาย user พร้อมกัน → deactivate / export
**Type:** FE only | **Effort:** S | **Depends on:** ไม่มี

### [P2] S2-RBAC-11: FE — UserDetailDrawer — Reset Password button ❌

**Type:** FE only | **Effort:** XS | **Depends on:** ไม่มี
**Steps:**

1. เพิ่มปุ่ม "รีเซ็ตรหัสผ่าน" ใน footer (ซ่อนถ้า status = locked/disabled)
2. Confirm dialog: "ส่ง email รีเซ็ตรหัสผ่านไปยัง {email}?"
3. `POST /api/identity/auth/forgot-password` → success toast

### [P2] S2-RBAC-12: FE — UserDetailDrawer — Change Role dialog ❌

**Type:** FE only | **Effort:** S | **Depends on:** S2-RBAC-04

### [P2] S2-RBAC-13: FE — Users page — URL state sync (`useSearchParams`) ❌

**Type:** FE only | **Effort:** S | **Depends on:** ไม่มี
**Params:** `q`, `roleKey`, `status`, `page`, `pageSize`

### [P1] S2-RBAC-14: FE — Permission Editor Wire → Real API ❌

**Type:** FE only | **Effort:** S | **Depends on:** S2-RBAC-05, S2-RBAC-15b

### [P1] S2-RBAC-15: BE — อัปเดต `role_permissions` schema รองรับ 4 states ❌

**Type:** BE only | **Effort:** M | **Depends on:** ไม่มี
**Steps:**

1. Migration: เพิ่ม column `state varchar(10) DEFAULT 'allow'` + `override_value varchar(20) NULL`
2. อัปเดต `RolePermission` domain entity + EF config
3. อัปเดต `PUT /api/user/roles/{id}/permissions`
4. อัปเดต `GET /api/user/roles`
5. `dotnet build && dotnet test` ผ่าน

### [P2] S2-RBAC-16: BE — อัปเดต seed data ให้ตรงกับ Default Permission Matrix v2 ❌

**Type:** BE only | **Effort:** XS | **Depends on:** S2-RBAC-15

### [P2] S2-RH-02: FE — Wire RoleHierarchyTab → Real API ❌

**Goal:** `RoleHierarchyTab` แสดงข้อมูลจาก API จริง + สร้าง/ลบ rule ได้
**Type:** FE only | **Effort:** S | **Depends on:** S2-RH-01 ✅

### [P2] Status Permissions

**Goal:** กำหนดว่า Role ไหนทำ action อะไรกับ Order status ได้บ้าง
**Screens:** `ac-stperm`
**Notes:** permission matrix ระดับ status transition

---

## GROUP 4 — Warehouse WMS `P2`

### [P2] Inbound Management

**Goal:** รับสินค้าเข้า warehouse — scan, verify, putaway
**Screens:** `inbound`
**Spec:** ยังไม่มี → เขียนจาก mockup

### [P2] Pick / Pack

**Goal:** หยิบและแพ็คสินค้าตาม Order — print label, confirm
**Screens:** `pickpack`
**Spec:** ยังไม่มี → เขียนจาก mockup

### [P2] Returns & Non-Conforming

**Goal:** จัดการสินค้าคืน และสินค้าที่ไม่ผ่าน QC
**Screens:** `returns`
**Spec:** ยังไม่มี → เขียนจาก mockup

### [P2] Cycle Count (Stock Verification)

**Goal:** นับสต็อกจริง เทียบกับ system และ reconcile
**Screens:** `cyclecount`
**Spec:** ยังไม่มี → เขียนจาก mockup

---

## GROUP 5 — Admin & Reporting `P2`

### [P1] S2-DASH-00: Spec — Dashboard + Integration Health ❌

**Goal:** เขียน `docs/specs/dashboard.md`
**Type:** Spec (Architect) — ไม่มี code
**Effort:** S | **Depends on:** ไม่มี
**Mockup ref:** `view-dashboard` + `view-reports` ใน `docs/mockups/b1dx_full_app_v0.html`

### [P2] S2-MON-01: Sync Monitoring Dashboard 🆕

**Goal:** Admin เข้าดูรายงานสถานะการ sync ออเดอร์จากตลาดออนไลน์ → OMS ได้แบบ real-time
**Screens:** `(app)/admin/sync-monitoring`
**Type:** BE + FE
**Effort:** M (3–5 วัน)
**Depends on:** S2-MSY-02a (marketplace sync worker)

**BE Deliverables:**

- `GET /admin/sync-monitoring/summary?date=2026-03-30`
- `GET /admin/sync-monitoring/orders?date=...&status=failed&platform=lazada&page=1`
- `GET /admin/sync-monitoring/dead-letters?date=2026-03-30`

**FE Deliverables:**

- Summary cards: Total / Synced / Failed / Dead-lettered
- Date picker
- Failed orders table
- Dead-letter table
- Auto-refresh ทุก 60 วินาที

**Out of Scope (MVP):** Retry/resolve button, Export CSV, Notification/alert

### [P2] Reports & Analytics

**Goal:** รายงานสรุป order, inventory, performance ตาม period/brand/warehouse
**Screens:** `reports`
**Notes:** chart + export CSV/Excel, filter date range

### [P2] Audit Log

**Goal:** บันทึกทุก action ที่เกิดในระบบ — who did what, when
**Screens:** `audit`
**Notes:** immutable, searchable, retention 90 วัน

### [P2] Order Lifecycle Timeline

**Goal:** แสดง timeline ของ Order ตั้งแต่สร้างจนปิด
**Screens:** `lifecycle`

---

## GROUP 6 — Billing `P3`

### [P3] Billing & Invoices

**Goal:** สรุปค่าใช้จ่าย, ออก invoice, tracking การชำระ
**Screens:** `billing`
**Notes:** รอ payment gateway integration ก่อน

---

## GROUP 6.5 — Auth Technical Debt `P1–P2`

> ค้นพบระหว่าง Phase 3 E2E testing — tests ถูก skip เพราะ FE ยังไม่ implement หรือ test infra ขาด

### [P1] OtpScreen: แยก error 429 (OTP Rate Limit) ออกจาก generic error

**Type:** FE only — `OtpScreen.tsx`
**Detail:** `handleVerify()` มี catch แบบ generic ทำให้ 429 กับ 400 แสดง error เดียวกัน — ต้องแยก check `error.code === 'identity.otp_rate_limited'`
**E2E test:** `auth.spec.ts` — "5 failed OTP attempts → 429 → rate limit message shown" (ยัง skip)

### [P2] ActivityLog: เพิ่ม `otp.success` ใน S1-03

**Type:** BE only — `IdentityService.cs` → VerifyOtp handler
**Spec:** `docs/specs/auth-flow.md` → Activity Logs section

### [P2] ActivityLog: เพิ่ม `password.reset_completed` ใน S1-04/S1-08

**Type:** BE only — `ResetPasswordCommandHandler`

### [P2] OtpScreen: Resend Cooldown 60 วินาที

**Type:** FE only — `OtpScreen.tsx` (S1-18) ❌ Still TODO

### [P2] LoginForm: handle `otpRequired: false` → skip /otp page

**Type:** FE only — `LoginForm.tsx`

### [P2] Playwright: Setup project สร้าง `storageState` ก่อนรัน E2E suite

**Type:** Test infra — `playwright.config.ts` + `e2e/fixtures/auth.fixture.ts`

---

## GROUP 6.6 — Auth Form Refactor (RHF + Zod + Tailwind) `P2`

> Refactor auth screens ให้ใช้ React Hook Form + Zod resolver + Tailwind แทน inline styles
> Build order: Task 1–2 ก่อน → Tasks 3–7 ตาม dependency

### [P2] `RHFOtpInput` — OTP 6-digit input component ใน @b1dx/ui *(ทำก่อน)*

**Goal:** สร้าง `RHFOtpInput` ที่ใช้กับ RHF `<Controller>` ได้ทันที
**Type:** FE only — `packages/ui/src/components/forms/inputs/RHFOtpInput.tsx`
**Effort:** S

### [P2] `RoleSelectCard` — Role selection card ใน @b1dx/ui *(ทำก่อน)*

**Type:** FE only — `packages/ui/src/components/app/RoleSelectCard.tsx`
**Effort:** S

### [P2] `LoginForm` → RHF + Zod

**Type:** FE only — `src/features/auth/components/LoginForm.tsx`
**Effort:** M

### [P2] `ForgotPasswordForm` → RHF + Zod

**Type:** FE only — `src/features/auth/components/ForgotPasswordForm.tsx`
**Effort:** S

### [P2] `OtpScreen` → RHF + Zod + RHFOtpInput

**Type:** FE only — `src/features/auth/components/OtpScreen.tsx`
**Effort:** M

### [P2] `ResetPasswordForm` → RHF + Zod

**Type:** FE only — `src/features/auth/components/ResetPasswordForm.tsx`
**Effort:** M

### [P2] `RoleSelectScreen` → RHF + Zod + RoleSelectCard

**Type:** FE only — `src/features/auth/components/RoleSelectScreen.tsx`
**Effort:** M

---

## GROUP 6.8 — Known Issues (พบจาก Code Review) `P2`

### [P3] FE-OD-FIDELITY-01e-followup: MSW fixtures for skipped E2E in OrderItems Section `[1h]`

**Goal:** ปลด `test.skip` 2 cases ใน `e2e/order-detail/od-fidelity-01e-items-section.spec.ts` ที่รอ MSW mock support
**Type:** FE (test infra)
**Effort:** S (~1h)
**Files:**

- `apps/b1dx-oms-fulfillment/src/mocks/msw/db/` — เพิ่ม fixture `ord-status-100` (status=100 → all items pill warn ⟳)
- `apps/b1dx-oms-fulfillment/src/mocks/msw/db/` — เพิ่ม fixture `ord-with-gift` (มี item `unitPrice==0 && lineTotal==0` หรือถ้า BE มี `isGift` flag → ใช้ตรงนั้น)
- `e2e/order-detail/od-fidelity-01e-items-section.spec.ts` ลบ `test.skip` 2 cases → un-skip

**Test cases:**

- [ ] E2E: gift row shows GIFT tag + green ฿0 (แถม) total — pass
- [ ] E2E: status=100 order → all match pills show ⟳ "กำลังจับคู่" — pass

**Background:** เกิดจาก code review FE-OD-FIDELITY-01e (2026-05-22) — unit tests cover behavior ครบแล้ว, แต่ E2E ต้องการ fixture เพิ่ม

---

### [P2] FIX-INTEGRATION-TESTS: Integration Tests 5 ตัว fail

**Goal:** แก้ Integration Tests 5 ตัวที่ fail ใน `GetRolesUserCountTests` + `CustomRoleSliceTests` + `AuthorizationSliceTests`
**Type:** BE (test infra)
**Effort:** M (~1h)
**Symptoms:**

- `GetRolesUserCountTests.GetRolesQuery_UserCount_ReturnsCorrectCountPerRole` — Expected: 2, Actual: 0
- `GetRolesUserCountTests.GetRolesQuery_UserCount_IsTenantScoped` — Expected: 3, Actual: 0
- `CustomRoleSliceTests.GetRoles_ReturnsBothSystemAndCustomRoles` — fail
- `CustomRoleSliceTests.DeleteRole_NoUsers_Returns204` — fail
- `AuthorizationSliceTests.AssignRole_WithPlatformAdmin_ReturnsOk` — Expected: OK, Actual: Unauthorized
  **Root cause:** user_memberships ไม่ถูก seed ใน test setup → userCount = 0; PlatformAdmin sign-in ล้มเหลว
  **Branch:** `fix/integration-tests`
  **Test cases:**

- [ ] regression: `GetRolesQuery_UserCount_ReturnsCorrectCountPerRole` pass
- [ ] regression: `GetRolesQuery_UserCount_IsTenantScoped` pass
- [ ] regression: `GetRoles_ReturnsBothSystemAndCustomRoles` pass
- [ ] regression: `DeleteRole_NoUsers_Returns204` pass
- [ ] regression: `AssignRole_WithPlatformAdmin_ReturnsOk` pass

---

### [P2] FE-AUTH-REFACTOR-02 — Fix inline styles + `onMouseEnter` DOM mutation ใน auth components `[2h]`

**Goal:** แทนที่ `onMouseEnter`/`onMouseLeave` ที่ mutate `e.currentTarget.style` โดยตรง และลด inline style ที่ไม่ justified ใน `LoginForm`, `OtpScreen`, `RoleSelectScreen`
**Type:** FE — Improvement / Refactor
**Spec Ref:** `web-repo/CLAUDE.md` § Style Conventions

**Files to Edit:**

- `apps/b1dx-oms-fulfillment/src/features/auth/components/LoginForm.tsx`
- `apps/b1dx-oms-fulfillment/src/features/auth/components/OtpScreen.tsx`
- `apps/b1dx-oms-fulfillment/src/features/auth/components/RoleSelectScreen.tsx`

**Violations ที่ต้องแก้:**

1. `LoginForm` SSO button: `onMouseEnter`/`onMouseLeave` mutate style → `hover:` Tailwind arbitrary
2. `OtpScreen` back button: `onMouseEnter`/`onMouseLeave` → `hover:` Tailwind arbitrary
3. `OtpScreen` digit inputs: `digitStyle()` + `onFocus`/`onBlur` `Object.assign(e.currentTarget.style, ...)` → state-driven className แทน (`isFocused` state หรือ CSS `:focus` selector)
4. `OtpScreen`: hardcode `'#6366f1'` ใน `onFocus` → CSS var `var(--auth-ind)` หรือ Tailwind arbitrary
5. `RoleSelectScreen` role card + back button: `onMouseEnter`/`onMouseLeave` → `hover:` Tailwind arbitrary

**Exception ที่อนุญาต (dynamic/cannot Tailwind):**

- gradient `background` บน primary buttons (`linear-gradient(...)`) — ใช้ `style={{}}` ได้
- CSS var ที่ต้อง interpolate ใน string เช่น `border: \`1px solid ${isSelected ? '#6366f1' : 'var(--auth-role-border)'}\`` → แยกเป็น 2 className แทน

**Test Cases:**

- [ ] Unit `LoginForm`: SSO button hover ไม่ crash; ไม่มี `onMouseEnter` mutate style
- [ ] Unit `OtpScreen`: digit input focus/blur ทำงานถูกต้องด้วย state-driven approach
- [ ] Unit `RoleSelectScreen`: role card hover ไม่ crash; selected state แสดงถูกต้อง
- [ ] `pnpm lint && pnpm typecheck` ผ่าน

**Acceptance Criteria:**

- [ ] ไม่มี `onMouseEnter`/`onMouseLeave` ที่เรียก `e.currentTarget.style.X = ...` ใน 3 ไฟล์
- [ ] ไม่มี `Object.assign(e.currentTarget.style, ...)` ใน `OtpScreen`
- [ ] Hardcode `'#6366f1'` ใน `onFocus` เปลี่ยนเป็น CSS var
- [ ] Visual behavior ไม่เปลี่ยน

**Dependencies:** ไม่มี
**Effort estimate:** 2h

---

### [P3] FE-AUTH-REFACTOR-03 — i18n ครบทุกไฟล์ใน auth components `[1.5h]`

**Goal:** ครอบ hardcoded user-visible string ทุกตัวด้วย `t()` ใน auth components folder
**Type:** FE — Improvement
**Spec Ref:** `web-repo/CLAUDE.md` § i18n Rules

**Files to Edit:**

- `apps/b1dx-oms-fulfillment/src/features/auth/components/ChangePasswordForm.tsx` — label, button text, error messages
- `apps/b1dx-oms-fulfillment/src/features/auth/components/ForgotPasswordForm.tsx` — hardcode text เช่น `'ส่ง Email แล้ว'`, `'← กลับหน้า Login'`, tip box
- `apps/b1dx-oms-fulfillment/src/features/auth/components/LoginForm.tsx` — `'Sign in with Microsoft SSO'`, `'หรือ'`, `'มีปัญหา?'`, `'ติดต่อผู้ดูแล'`
- `apps/b1dx-oms-fulfillment/src/features/auth/components/OtpScreen.tsx` — `'Check your inbox'`, `'Expires in'`, `'← Back'`, `'ไม่ได้รับ?'`, `'ส่งอีกครั้ง'`, resend countdown
- `apps/b1dx-oms-fulfillment/src/features/auth/components/ResetPasswordForm.tsx` — `'ลิงก์ไม่ถูกต้องหรือหมดอายุ'`, `'ขอลิงก์ใหม่'`, `'← กลับ'`
- `apps/b1dx-oms-fulfillment/src/features/auth/components/RoleSelectScreen.tsx` — `'เข้าสู่ระบบ'`, `'← กลับ'`, `'ไม่พบ role สำหรับบัญชีนี้'`
- `apps/b1dx-oms-fulfillment/src/lib/i18n/locales/en.json` — เพิ่ม key ที่ขาด
- `apps/b1dx-oms-fulfillment/src/lib/i18n/locales/th.json` — เพิ่ม key ที่ขาด

**Test Cases:**

- [ ] ไม่มี hardcoded user-visible string ใน 6 component ไฟล์
- [ ] keys ครบใน `en.json` + `th.json`
- [ ] `pnpm lint && pnpm typecheck` ผ่าน

**Acceptance Criteria:**

- [ ] ทุก label, button text, error message ใช้ `t('key')` หรือ `t('key', { defaultValue })` pattern
- [ ] ไม่มี string ภาษาไทยหรืออังกฤษ hardcode ใน JSX (ยกเว้น placeholder เช่น `'your@company.com'`, `'••••••••'`)

**Dependencies:** ไม่มี (ทำ parallel กับ REFACTOR-01/02 ได้)
**Effort estimate:** 1.5h

---

### [P3] FE-AUTH-REFACTOR-04 — ย้าย `@keyframes` ออกจาก `SplashScreen.tsx` `[0.5h]`

**Goal:** ลบ `<style>` inline JSX ใน `SplashScreen` — ย้าย `@keyframes` และ CSS var ไปไว้ใน global CSS แทน
**Type:** FE — Refactor
**Spec Ref:** `web-repo/CLAUDE.md` § Style Conventions

**Files to Edit:**

- `apps/b1dx-oms-fulfillment/src/features/auth/components/SplashScreen.tsx` — ลบ `<style>` block
- `apps/b1dx-oms-fulfillment/src/app/globals.css` (หรือ equivalent) — เพิ่ม `@keyframes orb-drift`, `ping-slow`, `progress-slide` + `--splash-*` CSS vars

**Test Cases:**

- [ ] SplashScreen render ได้ปกติ ไม่มี animation หาย
- [ ] ไม่มี `<style>` tag ใน JSX output
- [ ] `pnpm lint && pnpm typecheck` ผ่าน

**Acceptance Criteria:**

- [ ] `<style>{...}</style>` ถูกลบออกจาก `SplashScreen.tsx`
- [ ] Animation ยังทำงานถูกต้องทุก browser

**Dependencies:** ไม่มี
**Effort estimate:** 0.5h

---

### [P2] FE-AUTH-REFACTOR-05 — Clarify SSO button stub ใน `LoginForm` `[?h]`

**Goal:** SSO "Sign in with Microsoft" button มี `onClick={() => {}}` ว่างเปล่า — ต้อง implement จริง หรือ hide จนกว่าจะพร้อม
**Type:** FE — Spec clarification required
**Spec Ref:** ต้องถาม PO / check spec ก่อน

**Options:**

- A) Implement Microsoft SSO (ต้องมี spec + BE endpoint)
- B) Hide button ด้วย feature flag หรือ env var จนกว่าจะพร้อม
- C) Disable + tooltip `'Coming soon'`

**Acceptance Criteria:**

- [ ] ไม่มี `onClick={() => {}}` ที่ไม่ทำอะไรใน production build

**Dependencies:** ต้องรอ spec decision
**Effort estimate:** TBD (ขึ้นกับ option ที่เลือก)

---

### [P2] FE-ORDERS-REVIEW-01 — Fix bugs ใน processing components: duplicate form, unused import, console.log `[1.5h]`

**Goal:** แก้ bug ที่พบจาก code review ใน `processing/` folder — duplicate form state, import error, console.log ใน production
**Type:** FE — Bug Fix
**Spec Ref:** `web-repo/CLAUDE.md` § TypeScript Conventions

**Files to Edit:**

- `apps/b1dx-oms-fulfillment/src/features/orders/components/processing/filter-products/FilterProductNumberItem.tsx` — ลบ `useForm()` ภายใน, ใช้ `control` prop ที่รับมาแทน; ลบ console.log
- `apps/b1dx-oms-fulfillment/src/features/orders/components/processing/filter-products/FilterProductGroup.tsx` — ลบ `useForm()` ภายใน, ใช้ `control` prop ที่รับมาแทน
- `apps/b1dx-oms-fulfillment/src/features/orders/components/processing/reserve-stock/ReserveStockFailedTab.tsx` — ลบ `import { Section } from 'lucide-react'` (ไม่มีใน lucide-react + ไม่ได้ใช้)
- `apps/b1dx-oms-fulfillment/src/features/orders/components/processing/change-status/ChangeStatusSectionWrapper.tsx` — ลบ `console.log` 2 บรรทัด (line 45–46)
- `apps/b1dx-oms-fulfillment/src/features/orders/components/processing/filter-products/FilterProductsSection.tsx` — ลบ `console.log` (line 280)
- `apps/b1dx-oms-fulfillment/src/features/orders/components/processing/reserve-stock/ReserveStockSectionWrapper.tsx` — ลบ `console.log` 2 บรรทัด (line 45–46)

**Root Causes:**

- `FilterProductNumberItem` + `FilterProductGroup`: รับ `control` จาก parent form แต่สร้าง `useForm()` ซ้ำภายใน → submit อ่านค่าจาก form ผิดชุด
- `ReserveStockFailedTab`: `Section` ไม่มีใน lucide-react → build error
- console.log ทั้ง 3 ไฟล์: debug code หลงเหลือจาก development

**Test Cases:**

- [ ] regression: `FilterProductNumberItem` submit ส่งค่าจาก parent form ได้ถูกต้อง (ไม่ใช่ internal form)
- [ ] regression: `FilterProductGroup` submit ส่งค่าจาก parent form ได้ถูกต้อง
- [ ] `pnpm build` ผ่านโดยไม่มี import error จาก `ReserveStockFailedTab`
- [ ] ไม่มี `console.log` หลงเหลือใน 3 ไฟล์
- [ ] `pnpm lint && pnpm typecheck` ผ่าน

**Acceptance Criteria:**

- [ ] ไม่มี `useForm()` ซ้ำซ้อนใน `FilterProductNumberItem` และ `FilterProductGroup`
- [ ] ไม่มี import ที่ไม่มีอยู่จริงใน lucide-react
- [ ] ไม่มี `console.log` ใน production code ทั้ง folder

**Dependencies:** ไม่มี
**Effort estimate:** 1.5h

---

### [P3] FE-ORDERS-REVIEW-02 — Fix type typo + default export consistency ใน orders/components `[0.5h]`

**Goal:** แก้ type name typo และ standardize export style ให้ใช้ named export ทั้ง folder
**Type:** FE — Refactor
**Spec Ref:** `web-repo/CLAUDE.md` § TypeScript Conventions

**Files to Edit:**

- `apps/b1dx-oms-fulfillment/src/features/orders/components/processing/filter-products/FilterProductNumberItem.tsx` — แก้ `FilterProductNumberItemrops` → `FilterProductNumberItemProps`; เปลี่ยน `export default` → `export function`
- `apps/b1dx-oms-fulfillment/src/features/orders/components/processing/filter-products/FilterProductSKU.tsx` — เปลี่ยน `export default` → `export function`
- `apps/b1dx-oms-fulfillment/src/features/orders/components/order-detail/CustomerInfo.tsx` — เปลี่ยน `export default function` → `export function`
- ไฟล์ที่ import จาก 3 ไฟล์ข้างต้น — อัพเดท import syntax ให้ตรง

**Test Cases:**

- [ ] `pnpm typecheck` ผ่านหลังแก้ type name
- [ ] ไม่มี `export default` เหลือใน folder (ยกเว้น Next.js page files)
- [ ] Import ทุกจุดที่ใช้ component เหล่านี้ยังทำงานถูกต้อง

**Acceptance Criteria:**

- [ ] Type name `FilterProductNumberItemProps` ถูกต้อง (ไม่มี typo)
- [ ] Named export ทั้ง 3 ไฟล์

**Dependencies:** FE-ORDERS-REVIEW-01 (ควรทำพร้อมกันหรือหลัง)
**Effort estimate:** 0.5h

---

### [P3] FE-ORDERS-REVIEW-03 — Fix inline style + hardcoded default props ใน order-detail `[0.5h]`

**Goal:** แปลง inline style ที่ใช้ Tailwind แทนได้ + ลบ test data ออกจาก default props
**Type:** FE — Improvement
**Spec Ref:** `web-repo/CLAUDE.md` § Style Conventions

**Files to Edit:**

- `apps/b1dx-oms-fulfillment/src/features/orders/components/order-detail/OrderDetailView.tsx:194` — `style={{ background: "rgba(255,255,255,.12)" }}` → `className="bg-white/[.12]"`
- `apps/b1dx-oms-fulfillment/src/features/orders/components/order-detail/OrderHero.tsx:57` — `style={{ background: "rgba(255,255,255,.12)" }}` → `className="bg-white/[.12]"`
- `apps/b1dx-oms-fulfillment/src/features/orders/components/order-list/OrderAllWrapper.tsx:268` — `style={{ width: 36 }}` → `className="w-9"`
- `apps/b1dx-oms-fulfillment/src/features/orders/components/order-detail/OrderCustomerSection.tsx` — เปลี่ยน default props `phone = "086-050-5753"`, `avatarUrl = "https://github.com/shadcn.png"` → `undefined` หรือ empty string

**Exception ที่อนุญาต (dynamic — ยกเว้น):**

- `OrderInfoSection.tsx`: `style={{ background: order.platformColor }}` — runtime dynamic value
- `OrderAllWrapper.tsx`: `style={{ width: widths[col.key] }}`, `style={{ backgroundColor: PLATFORM_COLOR[...] }}` — computed dynamic value
- `OrderAllWrapper.tsx`: `style={{ tableLayout: 'fixed', minWidth: ... }}` — computed dynamic value

**Test Cases:**

- [ ] Visual ของ `OrderDetailView` และ `OrderHero` ไม่เปลี่ยน หลังแทน Tailwind
- [ ] `OrderCustomerSection` ไม่แสดง test phone number หรือ github avatar เมื่อ prop ไม่ถูกส่งมา
- [ ] `pnpm lint && pnpm typecheck` ผ่าน

**Acceptance Criteria:**

- [ ] `style={{ background: "rgba(255,255,255,.12)" }}` ไม่มีเหลือในทั้ง 2 ไฟล์
- [ ] `style={{ width: 36 }}` เปลี่ยนเป็น `w-9`
- [ ] Default prop ไม่มี hardcoded test data

**Dependencies:** ไม่มี
**Effort estimate:** 0.5h

---

## GROUP 6b — Shared Infrastructure `P2`

> *(เดิม GROUP 6 Shared Infrastructure — เปลี่ยนชื่อเพื่อไม่ชนกับ GROUP 6 Billing)*
> Centralized components ที่ reusable ได้ง่ายๆ สำหรับทุก module

### [P2] S2-SHARED-04: Platform Credentials Master Management 🆕

**Goal:** Centralized, reusable platform credentials service — support Shopee, Lazada, TikTok, etc.
**Type:** BE only
**Effort:** M (3-5 days)
**Depends on:** S2-SHARED-03 (Master Data Foundation)
**Target Sprint:** Sprint 3

**Deliverables:**

1. Domain Models: `PlatformCredential`, `PlatformType` enum, `CredentialStatus` enum
2. Infrastructure Layer: `PlatformCredentialsRepository`, `PlatformApiConnector` interface + impls (Shopee, Lazada, TikTok), `PlatformApiFactory`, `CredentialEncryption`
3. Application Services: `IPlatformCredentialsService`, `IPlatformApiMetadataService`
4. Configuration & Options: `PlatformCredentialsOptions` + per-platform settings
5. Database Schema: reuse `channel.platform_credentials` + optional `channel.platform_api_endpoints`, `channel.platform_rate_limits`
6. Test Coverage: unit + integration + cache refresh stress test

**Implementation Steps:**

1. Extract `PlatformType` enum + `CredentialStatus` enum → Domain
2. Rename `ShopCredential` → `PlatformCredential` (+ add fields)
3. Create `IPlatformCredentialsService` + cache-based implementation
4. Create `PlatformApiFactory` + connector interfaces
5. Create `PlatformApiMetadataService`
6. Register in DI + add to `appsettings.json`
7. Update `ShopCredentialCache` → delegate to `PlatformCredentialsService`
8. Tests + documentation

---

## GROUP 7 — Features นอก Mockup `P2–P3`

> งานเหล่านี้ไม่มีใน mockup — ต้องเขียน spec เองก่อน build

### [P2] Email / Notification Service `BE only`

**Goal:** ส่ง email / push notification เมื่อ event สำคัญเกิดขึ้น
**Type:** BE only — ไม่มี UI (config อยู่ใน Settings)
**Spec:** `docs/specs/email-infra.md` ✅
**Notes:** background queue, retry, template-based

#### [P2] S2-EMAIL-INFRA-01: Contracts — IEmailService + IEmailTemplateRenderer + EmailMessage ❌

**Type:** BE — `Application/Contracts/`
**Effort:** XS | **Depends on:** ไม่มี

#### [P2] S2-EMAIL-INFRA-02: EmailOptions + Config Schema ❌

**Type:** BE — `Infrastructure/Email/EmailOptions.cs`
**Effort:** XS | **Depends on:** ไม่มี

#### [P2] S2-EMAIL-INFRA-03: IEmailProvider + SmtpEmailProvider + NullEmailProvider ❌

**Type:** BE — `Infrastructure/Email/Providers/`
**Effort:** S | **Depends on:** INFRA-02
**NuGet:** `MailKit`

#### [P2] S2-EMAIL-INFRA-04: DB Migration — notification.email_delivery_log ❌

**Type:** BE — `Infrastructure/Persistence/`
**Effort:** XS | **Depends on:** ไม่มี

#### [P2] S2-EMAIL-INFRA-05: EmailService + EmailBackgroundService ❌

**Type:** BE — `Infrastructure/Email/`
**Effort:** M | **Depends on:** INFRA-02, INFRA-03, INFRA-04

#### [P2] S2-EMAIL-INFRA-06: EmailTemplateRenderer (Fluid) + DI Registration ❌

**Type:** BE — `Infrastructure/Email/`
**Effort:** S | **Depends on:** INFRA-05
**NuGet:** `Fluid.Core`

### [P2] S2-EMAIL-03: BE — Welcome / Invite Email Template `[~] assigned: srattha 2026-04-03`

**Goal:** implement HTML email template สำหรับ invite user + wire เข้า `InviteUserCommandHandler`
**Type:** BE only
**Effort:** S
**Steps:**

1. สร้าง Liquid template — `Infrastructure/Email/Templates/welcome.html` ตาม `docs/specs/email-templates.md`
2. Variables: `display_name`, `tenant_name`, `role_name`, `login_url`, `temp_password` (optional), `inviter_name`, `force_password_change`
3. Wire ใน `InviteUserCommandHandler` เมื่อ `sendWelcomeEmail = true`
4. Conditional block `{% if temp_password %}` — ไม่แสดงถ้า `initialPassword` ไม่ได้ set
5. Unit test: template variants (with/without password), `force_password_change` warning block
   **Depends on:** S2-EMAIL-01 ✅, S2-EMAIL-INFRA-01 through INFRA-06

### [P2] S2-EMAIL-04: BE — OTP Verification Email Template ❌

**Goal:** implement OTP email template + wire เข้า `IdentityService`
**Type:** BE only | **Effort:** S
**Depends on:** S2-EMAIL-01 ✅, S2-EMAIL-INFRA-01 through INFRA-06

### [P2] Webhook Outbound

**Goal:** ส่ง event ออกไปยัง external system เมื่อ order status เปลี่ยน
**Type:** BE only
**Notes:** signature verification, retry with backoff, delivery log

### [P2] Tenant Settings

**Goal:** Admin ตั้งค่าระดับ tenant — timezone, currency, branding, feature flags
**Type:** BE + FE (หน้า Settings — ไม่มีใน mockup v0)

### [P3] API Keys / Integrations

**Goal:** สร้างและจัดการ API Key สำหรับ external integration
**Type:** BE + FE

### [P3] Multi-language / i18n

**Goal:** รองรับ TH/EN ครบทุก screen
**Type:** FE (ส่วนใหญ่) + BE (error messages)
**Notes:** เริ่มทำควบคู่ทุก feature ไม่ทำทีเดียวหลังสุด

---

## GROUP 7b — SyncOrder Monitoring `P2`

> *(เดิม GROUP 7 ที่สอง — เปลี่ยนชื่อเพื่อไม่ชนกับ GROUP 7 Features นอก Mockup)*
> **เป้าหมาย:** Observability ครบทั้ง 3 pipeline — Go Ingest (A) → Marketplace Sync (B) → ERP Sync (C)
> **Spec:** `docs/specs/sync-order-logging-design.md` v1
> **Stack:** Serilog (JSON) + OpenTelemetry Metrics + Loki + Prometheus

### [P1] S2-MON-00: DB Column — `go_received_at` บน TmpOrderDb ❌

**Goal:** เพิ่ม column สำหรับ track เวลาที่ Go Service เขียน order ลง TmpOrderDb
**Type:** DB Migration (TmpOrderDb) + Go Service update
**Effort:** XS

```sql
ALTER TABLE oms.orders ADD COLUMN IF NOT EXISTS go_received_at timestamptz NULL;
```

### [P2] S2-MON-01: Go Service — Prometheus Metrics + Health Endpoint ❌

**Type:** Go Service
**Effort:** M

**Metrics ที่ต้องเพิ่ม:**
| Metric | Type | Labels |
|---|---|---|
| `oms_ingest_webhook_total` | Counter | `platform`, `status` |
| `oms_ingest_api_pull_total` | Counter | `platform`, `status` |
| `oms_ingest_write_db_total` | Counter | `platform`, `result` |
| `oms_ingest_write_db_duration_ms` | Histogram | `platform` |
| `oms_ingest_last_pull_at` | Gauge | `platform` |
| `oms_ingest_pending_in_tmpdb` | Gauge | `platform` |
| `oms_ingest_webhook_error_total` | Counter | `platform`, `error_type` |

### [P2] S2-MON-02: Structured Logging Implementation — All Pipelines ❌

**Type:** Implementation — Sync Worker (.NET) + Go Service
**Effort:** M
**Depends on:** S2-MON-SPEC-00, S2-ERP-04

### [P2] S2-MON-03-Metrics: OpenTelemetry Metrics (Pipeline B + C) ❌

**Type:** Sync Worker (.NET 10)
**Effort:** M
**Depends on:** S2-ERP-04

### [P2] S2-MON-03: .NET Sync Worker — Health Endpoints ❌

**Endpoints:** `GET /health/live`, `GET /health/ready`, `GET /health/status`
**Depends on:** S2-MON-03-Metrics

### [P2] S2-MON-04: Alert Rules (Prometheus Alertmanager) ❌

**Type:** DevOps / Infrastructure
**Effort:** S

**Critical Alerts:**

- `IngestServiceSilent` — `oms_ingest_last_pull_at` age > 10 min
- `MktSyncWorkerSilent` — `mkt_sync_last_batch_at` age > 2 × PollingSeconds
- `ErpSyncWorkerSilent` — `erp_sync_last_batch_at` age > 2 × PollingSeconds
- `TmpDbPendingExplosion` — `mkt_sync_pending_count` > 1,000 orders

**Warning Alerts:**

- `IngestWebhookFailRateHigh` — webhook fail rate > 5% ใน 5 นาที
- `MktSyncFailRateHigh`, `MktSyncDeadLetterGrowing`, `ErpSyncDeadLetterGrowing`, `ErpPendingHigh`, `WatermarkLagHigh`

### [P2] S2-MON-05: E2E Latency Diagnostic Queries ❌

**Type:** Documentation / Runbook
**Effort:** XS
**Output:** `docs/runbooks/syncorder-monitoring-queries.md`

### [P3] S2-MON-06: Grafana Dashboard ❌

**Type:** DevOps / Grafana JSON
**Effort:** M
**Depends on:** S2-MON-02, S2-MON-01

---

## GROUP 8 — Order State Machine `P1–P3`

> breakdown จาก `docs/b1dx_roadmap_planner.html` (2026-04-15)
> **Spec (State Machine):** `docs/specs/order-state-machine.md` — config_json schema, transition table ครบ, courier webhook event mapping
> **Spec (Mock Data):** `docs/specs/osm-mock-data.md` — 20 predefined test orders (Group A–D), statusHistory fixtures, acceptance criteria ครบ
> Dependency sequence: `OSM-BE-01` + `OSM-FE-01` → `OSM-BE-02` → `OSM-FE-02` + `OSM-FE-03` + `OSM-FE-04` → `OSM-INT-01` → `OSM-FE-05` → `OSM-INT-02`

### Q2 — วางแผนดีๆ (ยาก + High Value)

### [P2] OSM-INT-02: Courier webhook handler

- **Type:** `[BE+Integration]`
- **Effort:** L · 5-7 วัน
- **Dependency:** OSM-BE-02, queue service
- **Scope:**
  - endpoint: `POST /webhooks/courier/{provider}` (spx, jnt, flash, kex)
  - Verify HMAC-SHA256 signature
  - Map `eventType` → `toStatus`
  - Idempotency: `X-Idempotency-Key` + `oms.webhook_events` table
  - DLQ: retry 3 ครั้ง → insert `oms.webhook_dead_letters`
- **Acceptance Criteria:**
  - [ ] 500→550, 550→600, 500→900 อัปเดตอัตโนมัติจาก webhook
  - [ ] duplicate webhook → idempotent
  - [ ] invalid signature → 401 `WHK-4001`

---

### Q3 — ทำเสริม (ง่าย + Medium Value)

### [P2] OSM-FE-07: Order Timeline UI ← moved to current-sprint (2026-04-21)

- **Type:** `[FE]` | **Effort:** XS · ครึ่งวัน | **Claimed:** wat · `branch: feat/osm-fe-07-order-timeline`
- **Dependency:** OSM-BE-02 ✅

### [P2] OSM-BE-03: Audit log per transition

- **Type:** `[BE+FE]` | **Effort:** S · 1-2 วัน | **Dependency:** OSM-BE-02

### [P3] OSM-FE-08: Admin UI สำหรับแก้ไข JSON config

- **Type:** `[FE]` | **Effort:** M · 2-3 วัน | **Dependency:** OSM-BE-01

---

### Q3b — Mock Data / Test Fixtures

> **Spec:** `docs/specs/osm-mock-data.md` — 20 predefined orders, statusHistory fixtures, acceptance criteria ครบ

### 🔧 [P2] OSM-MOCK-02 — Cancel Flow Snapshots: mock data สำหรับ cancel scenarios (Group B)

> **Spec:** `docs/specs/osm-mock-data.md` § Group B
> **Dependency:** OSM-MOCK-01 (branch เดียวกันได้)

**Found:** 2026-04-22 | **Effort:** ~1h | **Tags:** `[FE]`

- **Scope (FE only):**
  1. `mockOrderList.ts` — inject Group B (5 orders: `order-osm-cancel-*`)
  2. `orders.ts` — `OSM_MOCK_DETAILS` สำหรับ cancel orders พร้อม statusHistory + reason
- **Test Cases:**
  - [ ] `order-osm-cancel-requested` (ST 750) — ปุ่ม approve_cancel แสดงเฉพาะ ff-manager+, ปุ่ม reject_cancel แสดงด้วย
  - [ ] `CancelOrderDialog` `order-osm-cancel-requested` — confirm button disabled จนกว่าจะเลือก reason
  - [ ] `order-osm-cancel-approved` (ST 800) history — มี entry 400→750→800 พร้อม reason
  - [ ] `order-osm-cancel-rejected` (ST 400) history — มี entry 400→750→400
  - [ ] `order-osm-cancel-st490` (ST 800) — sideEffects ใน history แสดง void_label + release_stock
  - [ ] filter tab "cancelled" — แสดง cancel orders ทั้ง 5 ถูกต้อง

---

### 🔧 [P3] OSM-MOCK-03 — Return + Terminal Snapshots: mock data สำหรับ return และ terminal states (Groups C+D)

> **Spec:** `docs/specs/osm-mock-data.md` § Group C + D
> **Dependency:** OSM-MOCK-01

**Found:** 2026-04-22 | **Effort:** ~0.5h | **Tags:** `[FE]`

- **Scope (FE only):**
  1. `mockOrderList.ts` — inject Group C+D (5 orders: return + terminal)
  2. `orders.ts` — `OSM_MOCK_DETAILS` สำหรับ return orders + terminal states
- **Test Cases:**
  - [ ] `order-osm-buyer-return` (ST 900) history — มี entry 600→900 พร้อม reason + sideEffects: release_stock, create_refund
  - [ ] `order-osm-courier-return` (ST 900) history — มี entry 500→900 พร้อม sideEffects: release_stock, close_fulfillment_request
  - [ ] `order-osm-terminal-700/800/900` — `OrderActionPanel` ไม่แสดงปุ่มเลย (transitions=[])
  - [ ] filter tab "cancelled" — return orders (900) ปรากฏ

---

### Q4 — เลื่อน Phase 2

### [P3] OSM-INT-03: Courier API fallback flow (retry + manual)

- **Type:** `[BE+FE+Integration]` | **Effort:** L · 5-7 วัน | **Note:** B1-10022 — edge case, Phase 2

### [P3] OSM-BE-04: Per-tenant JSON config override

- **Type:** `[BE]` | **Effort:** M · 2-3 วัน | **Note:** Phase 2 — ไม่จำเป็นสำหรับ MVP

### [P3] OSM-BE-05: SLA alert engine

- **Type:** `[BE]` | **Effort:** L · 5-7 วัน | **Note:** B1-10002 — Phase 2

### [P3] OSM-INT-04: Video recording PWA (Pack + Return)

- **Type:** `[FE+Integration]` | **Effort:** XL · 2-3 สัปดาห์ | **Note:** Phase 2 — complex

---

### Summary — Recommended Sprint Sequence

| Phase                     | Tasks                                    | Effort รวม   | ผลลัพธ์                                |
| ------------------------- | ---------------------------------------- | ------------ | -------------------------------------- |
| **Sprint A** (Q1+Q2 core) | OSM-BE-01 + OSM-FE-01,02,03,04           | ~6-8 วัน     | Foundation พร้อม, FE ทำงานได้บน MSW    |
| **Sprint B** (Q2 main)    | OSM-BE-02 + OSM-FE-05 + OSM-INT-01       | ~8-10 วัน    | Transition ทำงานจริง + permission safe |
| **Sprint C** (Q3)         | OSM-FE-06,07 + OSM-BE-03 + OSM-INT-02    | ~8-12 วัน    | Webhook live, UI ครบ, audit trail      |
| **Phase 2**               | OSM-INT-03,04 + OSM-BE-04,05 + OSM-FE-08 | ~3-4 สัปดาห์ | Multi-tenant, SLA, PWA                 |

---

## [P1] Force Password Change Flow

> Spec: `docs/specs/auth-flow.md` → section "Force Password Change"
> Approach: Option B (post-auth) — nav guard after full auth, /change-password page
> Branch: `feat/s2-fpc`

### S2-FPC-01: `[BE]` DB Migration — `users.must_change_password` ❌

**Effort:** XS | **Priority:** P1 — blocks ทุก task ใน feature นี้

### S2-FPC-02: `[BE]` InviteUser + UpdateUser — set MustChangePassword flag ❌

**Effort:** S | **Priority:** P1 | **Depends on:** S2-FPC-01

### S2-FPC-03: `[BE]` GET /me — เพิ่ม `mustChangePassword` field ❌

**Effort:** XS | **Priority:** P1 | **Depends on:** S2-FPC-01

### S2-FPC-04: `[BE]` POST `/api/identity/auth/change-password` endpoint ❌

**Effort:** M | **Priority:** P1 | **Depends on:** S2-FPC-01, S2-FPC-02, S2-FPC-03
**Steps:**

1. Request: `ChangePasswordRequest(string NewPassword, string ConfirmPassword)`
2. Handler: validate → `GeneratePasswordResetTokenAsync` + `ResetPasswordAsync` → `MustChangePassword = false` → `UpdateSecurityStampAsync`
3. Endpoint: `POST /api/identity/auth/change-password` — Bearer auth
4. Unit test: happy path + mismatch + too_weak + already false

### S2-FPC-05: `[FE]` AuthUser type + MSW mock + Navigation Guard ❌

**Effort:** S | **Priority:** P1 | **Depends on:** S2-FPC-03

### S2-FPC-06: `[FE]` `/change-password` page + ChangePasswordForm + wire API ❌

**Effort:** M | **Priority:** P1 | **Depends on:** S2-FPC-04, S2-FPC-05
**Steps:**

1. `features/auth/components/ChangePasswordForm.tsx` — RHF + Zod schema
2. Zod: password policy validation (min 8, upper, lower, digit, special)
3. `app/(auth)/change-password/page.tsx`
4. `POST /api/identity/auth/change-password` → success: clear session → `/login` + toast
5. MSW handler

---

## GROUP — Order SKU Match `P1` `[srattha]` `claimed: 2026-05-11`

> **Spec:** `docs/specs/order-sku-match.md` v1.0 ✅
> **Mockup:** `docs/mockups/sku_match_modal_improved.html` + `docs/mockups/order-list-full-improve.html`
> **Sprint ref:** `tasks/current-sprint.md` § GROUP — Order SKU Match
> **Master plan:** `tasks/plans/MATCH-SPEC-01/plan.md`
> **Type:** BE Foundation + BE Domain + BE App + Sync Worker + FE
> **Effort:** ~5.5-6.5 dev-days (7 tasks)
> **Problem:** order status 100 + matchStatus unmatched → user ติด dead-end เพราะ (1) FE ไม่มี entry point เปิด match UI, (2) BE ไม่มี endpoint manual match, (3) sync worker ไม่ทำ auto-match → `MatchStatus = null` ทุก order, (4) ไม่มี `Product` / `SkuMapping` / `Shop` tables
> **Dependency sequence:** SPEC-01 → BE-00 → BE-01 → (BE-02 || WORKER-01) → FE-01 → FE-02

### [P1] MATCH-SPEC-01 — Master spec
- **Effort:** S · ~2h
- **Branch:** `feat/match-spec-01`
- **Plan:** `tasks/plans/MATCH-SPEC-01/plan.md`
- เขียน `docs/specs/order-sku-match.md` v1.0 — data model, API contract, flows, error codes, permission matrix, E2E test cases
- Scope decisions confirmed: Option B (auto+manual); Shop entity full; Shop 1:N IntegrationCredential; no Shop CRUD UI (defer); default-shop backfill; keep `Order.ShopCredId`

### [P1] MATCH-BE-00 — Shop entity foundation
- **Effort:** M · ~1d | **Depends:** SPEC-01
- **Branch:** `feat/match-be-00-shop-entity`
- **Plan:** `tasks/plans/MATCH-BE-00/plan.md`
- `Shop` entity (Id, TenantId, Name, Code, Status, audit, RowVersion)
- `IntegrationCredential.ShopId` + `Order.ShopId` FK (nullable → backfill → NOT NULL)
- Migration backfill: 1 Default Shop per Tenant
- `GET /api/oms/shops` endpoint (read-only)

### [P1] MATCH-BE-01 — Product + SkuMapping
- **Effort:** M · ~1d | **Depends:** BE-00
- **Branch:** `feat/match-be-01-product-skumapping`
- **Plan:** `tasks/plans/MATCH-BE-01/plan.md`
- `Product` entity (TenantId, Code unique, Name, Barcode, Status)
- `SkuMapping` entity (TenantId, ShopId, PlatformSkuCode, InternalProductId) — UNIQUE `(TenantId, ShopId, PlatformSkuCode)`
- `GET /api/oms/products?q=` typeahead + `GET /api/oms/sku-mappings`

### [P1] MATCH-BE-02 — Manual match endpoint + perm
- **Effort:** M · ~1d | **Depends:** BE-01
- **Branch:** `feat/match-be-02-match-endpoint`
- **Plan:** `tasks/plans/MATCH-BE-02/plan.md`
- `POST /api/oms/orders/{id}/items/match` + `MatchOrderItemsHandler`
- `Order.SetMatchStatus(...)` public method (replace test-only)
- `OrderItem.MatchToProduct(...)` + columns `MatchedProductId`, `MatchedAt`
- `MatchStatusCalculator`: all → matched; some → partial; none → unmatched
- Auto-upsert `SkuMapping` ตอน match (teaching pattern)
- Seed permission `orders.match` (FfManager, BrandAdmin, SuperAdmin)

### [P1] MATCH-WORKER-01 — Auto-match in sync worker
- **Effort:** M · ~1d | **Depends:** BE-01 (parallel กับ BE-02)
- **Branch:** `feat/match-worker-01-auto-match`
- **Plan:** `tasks/plans/MATCH-WORKER-01/plan.md`
- `SkuMatchService` resolve `(ShopId, PlatformSkuCode) → InternalProductId`
- Wire เข้า `MarketplaceOrderSyncService.UpsertOrderAsync()`
- Preload SkuMapping per shop (no N+1)
- Manual takes precedence: skip items ที่ `MatchedProductId != null`
- Log `auto_match_result` event

### [P1] MATCH-FE-01 — SkuMatchDialog
- **Effort:** M · ~1d | **Depends:** BE-02
- **Branch:** `feat/match-fe-01-sku-match-dialog`
- **Plan:** `tasks/plans/MATCH-FE-01/plan.md`
- `SkuMatchDialog.tsx` ตาม mockup (header + item rows + footer)
- Typeahead product search (debounce 300ms) — `useProductSearch(query)`
- `useSkuMatch()` mutation → POST match endpoint
- Progress `N/total`, error handling (403, 404, 409)
- MSW handlers + i18n th/en
- E2E scaffold (test.skip → un-skip ใน FE-02)
- Visual fidelity check vs `sku_match_modal_improved.html` (BLOCKING)

### [P1] MATCH-FE-02 — Wire entry points
- **Effort:** S · ~0.5d | **Depends:** FE-01
- **Branch:** `feat/match-fe-02-wire-entry-points`
- **Plan:** `tasks/plans/MATCH-FE-02/plan.md`
- `OrderActionButton` dynamic label by matchStatus:
  - 100 + `unmatched`/`partial` → `🔗 Match SKU` (open SkuMatchDialog)
  - 100 + `matched`/`null` → `✓ ยืนยัน` (เดิม)
- OAD banner `isBlocked` เพิ่มลิงก์ `→ เปิด SKU Match`
- Permission gate `orders.match` (hide button if denied)
- Un-skip E2E ของ FE-01 + FE-02 → green

---

## GROUP — My Profile Dialog `P2`

> **Spec:** `docs/specs/my-profile-dialog-spec.md` ✅ Ready
> **Type:** BE + FE
> **Effort:** ~8h (5 tasks)
> **Decisions:** Stats scope by role | Edit → `/settings/profile` | 2FA CTA | Relative time + tooltip | Session-only sign out

### [P2] BE-PROFILE-01 — Extend `/api/identity/auth/me` response `[1h]`

**Type:** BE
**Spec Ref:** `docs/specs/my-profile-dialog-spec.md` § 13 Task BE-PROFILE-01
**Goal:** เพิ่ม fields ที่ขาดใน `MeResponse` ให้ FE ใช้ได้ทันที
**Files:**

- `Application/Identity/GetCurrentUserQuery.cs` — เพิ่ม `TwoFAEnabled`, `Tenant`, `LastLoginAt` ใน result
- `Infrastructure/Services/IdentityService.cs` — map จาก `AppUser.TwoFactorEnabled`, `UserProfile.LastLoginAtUtc`, tenant name
- `Api/Endpoints/LoginEndpoints.cs` — map response fields
  **Acceptance Criteria:**

- [ ] Response มี `twoFAEnabled` (bool), `tenant` (string), `lastLoginAt` (ISO 8601 UTC)
- [ ] `lastLoginAt` null → return `""` ไม่ crash
- [ ] unit test: happy path + null lastLoginAt
  **Effort:** 1h | **Priority:** P2 | **Tags:** `[BE]`

---

### [P2] BE-PROFILE-02 — `GET /api/v1/users/me/stats` `[2h]`

**Type:** BE
**Spec Ref:** `docs/specs/my-profile-dialog-spec.md` § 13 Task BE-PROFILE-02
**Goal:** Endpoint คืน stats (orders today, SLA rate, avg fulfill time) แบบ role-based scope
**Files:**

- `Application/User/GetUserStatsQuery.cs` — query + handler interface
- `Infrastructure/Services/UserStatsService.cs` — handler implementation
- `Api/Endpoints/UserEndpoints.cs` — endpoint registration
  **Scope logic:**
- `ff-member`, `wh-worker` → `WHERE assigned_to = currentUserId`
- `ff-manager`, `wh-admin`, `brand-admin` → `WHERE tenant_id = currentTenantId`
- `sys-admin` → ทั้ง system
  **Acceptance Criteria:**

- [ ] Role-based scope ทำงานถูกต้องทุก role
- [ ] `slaRate` = orders fulfilled ภายใน SLA / total orders (format "98.3%")
- [ ] `avgFulfillHours` = avg `fulfilledAt - createdAt` เฉพาะ completed orders (format "4.2h")
- [ ] ไม่มี orders → `{ ordersToday: 0, slaRate: "N/A", avgFulfillHours: "N/A" }`
- [ ] unit test + integration test ครอบคลุม scope แต่ละ role
  **Effort:** 2h | **Priority:** P2 | **Tags:** `[BE]`

---

### [P3] BE-PROFILE-03 — `GET /api/v1/auth/sessions/current` `[1h]`

**Type:** BE
**Spec Ref:** `docs/specs/my-profile-dialog-spec.md` § 13 Task BE-PROFILE-03
**Goal:** Expose current session detail (lastLoginAt, device, ip) ผ่าน API
**Files:**

- `Application/Identity/GetCurrentSessionQuery.cs` — query + handler interface
- `Infrastructure/Services/IdentityService.cs` — resolve session จาก `IdentitySessionRecord` by session claim
- `Api/Endpoints/LoginEndpoints.cs` หรือ `SessionEndpoints.cs`
  **Acceptance Criteria:**

- [ ] Return session ของ token ปัจจุบันเท่านั้น (via `jti` / session claim)
- [ ] 401 ถ้า session ถูก revoke แล้ว
- [ ] unit test: active session / revoked session / session ไม่พบ
  **Effort:** 1h | **Priority:** P3 | **Tags:** `[BE]` | **Note:** FE ใช้ `lastLoginAt` จาก `/me` แทนได้ชั่วคราว

---

### [P2] FE-PROFILE-01 — `UserProfileDialog` component `[3h]`

**Type:** FE
**Spec Ref:** `docs/specs/my-profile-dialog-spec.md` § 3–9
**Goal:** Implement modal dialog ตาม spec ครบทุก section
**Dependencies:** BE-PROFILE-01 (หรือ MSW mock)
**Files:**

- `features/users/components/UserProfileDialog/` — component + CSS
- `features/users/hooks/useUserProfile.ts` — fetch `/api/identity/auth/me` + `/api/v1/users/me/stats`
- MSW handlers สำหรับ `/api/identity/auth/me` และ `/api/v1/users/me/stats`
  **Sections ที่ต้อง implement:**
- Hero (gradient + blob animation × 3)
- Avatar (role-based gradient + initials จาก `AVATAR_ROLE_CONFIG`)
- Name Row (email, role badge, tenant badge, online chip)
- Stats Row (ordersToday, slaRate, avgFulfillHours)
- Info Grid 2×2 (email, tenant, 2FA with CTA, last login relative + tooltip)
- Action buttons (Edit Profile → `/settings/profile`, Sign out)
  **Acceptance Criteria:**

- [ ] Avatar gradient + initials ถูกต้องทุก role (6 roles)
- [ ] 2FA disabled → แสดง `⚠ Not enabled · Enable →` (orange, clickable)
- [ ] Last login แสดง relative time + tooltip absolute
- [ ] Click backdrop / ✕ → close modal
- [ ] Click Edit Profile → navigate `/settings/profile`
- [ ] Click Sign out → `doLogout()` + close modal
- [ ] Light mode override ทำงานถูกต้อง
- [ ] unit test: hook fetch states + avatar role mapping
- [ ] e2e scaffold: `e2e/user-profile/user-profile.spec.ts`
  **Effort:** 3h | **Priority:** P2 | **Tags:** `[FE]`

---

## Issues Backlog

> Done issues archived → tasks/archive/2026-04-backlog.md

### ✅ [IMPROVE][P2] FE-OD-IMPROVE-102 — Order Detail · Main scroll move to outermost container `[srattha]` `merged: 2026-05-26`

**Found:** 2026-05-26 | **Type:** Improvement | **Repo:** `web`
**Source:** GitHub issue [#102](https://github.com/B1DXDev/b1dx-fulfillment-workspace/issues/102) (parent #97 — Order Detail UI/i18n match mockup 100%)
**Plan:** `tasks/plans/FE-OD-IMPROVE-102/plan.md` (Done — PR #249, FE main `20f8d05`)
**Mockup ref (BLOCKING):** `docs/mockups/order-detail-improve.html` § `.demo-shell` / `.demo-body` / `.od-layout` (no inner overflow rules)
**Test Cases:** `docs/testing/order-module/tc-order-detail.md` § FE-OD-IMPROVE-102 — `TC_OMS-DETAIL_100`..`_105` (6 cases, 12-col)
**Effort:** XS · ~1h

#### Symptom
- `OrderDetailView.tsx` (L417-446) มี nested scroll: outer wrapper ใส่ `md:overflow-hidden h-full` + main column `flex-1 md:overflow-y-auto` + sidebar `w-72 md:overflow-y-auto` → **2 scroll tracks แยกกันบน desktop**
- UX เพี้ยน: sticky header/footer ไม่ทำงาน, mobile scroll lag, sidebar กับ main ไม่ scroll พร้อมกัน

#### Root cause
- Component สร้างตอนแรกใช้ pattern "fixed-height columns" — ไม่ตรงกับ mockup intent (`.demo-shell` + `.demo-body` page-level scroll, `.od-layout {align-items:start}` ไม่ sticky)

#### Fix
- Outer wrapper เปลี่ยนเป็น single scroll container: `flex-1 overflow-y-auto` + `data-testid="order-detail-scroll"`
- เพิ่ม inner layout wrapper: `flex flex-col md:flex-row min-h-full`
- ลบ `md:overflow-y-auto` ออกจาก main + sidebar columns
- ไม่แตะ AppShell (cross-cutting), ไม่แตะ flow bar internal scroll (overflow-x ภายใน)

#### Test
- E2E `od-improve-102-scroll-outermost.spec.ts` (6 cases — E1-E6 ครอบคลุม outer overflow, no nested scroll main/sidebar, synced scroll, mobile stack, mobile scroll smooth)
- Regression: `od-fidelity-01d-timeline-sidebar.spec.ts` + `od-fidelity-01b-status-flow.spec.ts`

---

### ✅ [IMPROVE][P2] FE-OD-IMPROVE-100 — Order Detail · Order info section theme/i18n `[srattha]` `merged: 2026-05-25`

**Found:** 2026-05-25 | **Type:** Improvement | **Repo:** `web`
**Source:** GitHub issue [#100](https://github.com/B1DXDev/b1dx-fulfillment-workspace/issues/100) (parent #97 — Order Detail UI/i18n match mockup 100%)
**Plan:** `tasks/plans/FE-OD-IMPROVE-100/plan.md` (Done — PR #247, FE main `2ddcd52`)
**Mockup ref (BLOCKING):** `docs/mockups/order-detail-improve.html` § `.od-card → ORDER INFO` (`renderInfoGrid`)
**Effort:** S~M · ~3h

#### Symptom
- `OrderInfoSection.tsx` ใช้ Tailwind raw + cell structure 7 cells (Order No · Platform · Service Policy · SLA · Payment · Tracking · [COD]) ไม่ตรง mockup ที่ใช้ 6 cells (Order ID · Platform · Service Policy · **Courier** · Payment · **COD เสมอ**)
- Hardcoded strings: `✏ แก้ไข` (L116), `toLocaleString("th-TH")` (L211), `toLocaleString("en-GB")` formatSla (L71–80)
- `PAYMENT_STATUS_LABELS` hardcoded Thai dict (L21–30) — ไม่รองรับ 2 ภาษา
- Payment method sub ใช้ `normalizeEnum().replace(/_/g, " ")` (L181) — ไม่ใช่ i18n

#### Root cause
- Component สร้างก่อน i18n infra พร้อม + ไม่มี locale-aware date/currency formatter ส่วนกลาง

#### Fix
- Rebuild Order info section ตาม mockup (6 cells, COD always-shown)
- เอา hardcoded strings ออกทั้งหมด → ใช้ `t()` + `formatDateTime`/`formatCurrency` ผ่าน `useI18nFormatter()` ใหม่
- เพิ่ม i18n keys: `order_detail.{edit, courier, no_courier, no_tracking, no_cod, cod_yes, payment_confirmed, payment_pending, sla_24h, sla_6h}` + `orders.payment.{paid, pending, partial, cod_pending, cod_collected, refunded, failed, waived}` + `orders.payment_method.{cod, bank_transfer, ...}`

#### Test
- Unit (OrderInfoSection cell render + i18n key + formatter call) + Unit (`formatters.test.ts`) + E2E `od-improve-100-order-info.spec.ts` (8 cases incl. screenshot vs mockup)

---

### ✅ [IMPROVE][P2] FE-OD-IMPROVE-99B — Order Status Labels · i18n support (100–900) `[srattha]` `merged: 2026-05-25`

> **Note:** เดิม claim เป็น FE-OD-IMPROVE-100 — rename เป็น 99B เพราะเป็น follow-up ของ -99 (status labels i18n out-of-scope ใน -99) bundled ใน PR #246 เดียวกัน. GitHub issue #100 จริงๆ คือ Order info theme/i18n (entry ด้านบน)

**Found:** 2026-05-25 | **Type:** Improvement | **Repo:** `web`
**Source:** Manual report — `/orders/order-osm-st700` (status node labels Thai hardcoded, ไม่รองรับ 2 ภาษา)
**Plan:** `tasks/plans/FE-OD-IMPROVE-99B-status-i18n/plan.md` (Done)
**Parent:** Follow-up of FE-OD-IMPROVE-99 (line 98 of -99 plan: status labels i18n out-of-scope, addressed here)
**Effort:** S · ~2h

#### Symptom
- `ORDER_STATUS_LABELS` ใน [`orderStatus.ts`](../web-repo/apps/b1dx-oms-fulfillment/src/features/orders/types/domain/orderStatus.ts#L24-L38) เป็น Thai hardcoded `Record<number, string>` — ไม่เปลี่ยนตาม `i18n.language`
- ขาด 3 codes: 550 (OutForDelivery), 750 (CancelRequested), 900 (Returned) — render เป็น number fallback
- 8 consumer files ใช้ `ORDER_STATUS_LABELS[code]` โดยตรง

#### Root cause
- Labels defined inline ใน TS file ไม่ได้ resolve ผ่าน i18n resources

#### Fix
- Convert `ORDER_STATUS_LABELS` → `ORDER_STATUS_LABEL_KEYS` (key paths) + `useOrderStatusLabel(code)` hook
- Add `orders.status.*` namespace ใน th.json + en.json (16 keys)
- Migrate 8 consumer files

#### Test
- Unit (key coverage symmetry) + component (update existing) + E2E `od-improve-100-status-labels.spec.ts` (5 cases)

---

### 🆕 [IMPROVE][P2] FE-OD-IMPROVE-99 — Order Detail State Machine connector logic + theme/i18n `[srattha]`

**Found:** 2026-05-25 | **Type:** Improvement | **Repo:** `web`
**Source:** GitHub issue [#99](https://github.com/B1DXDev/b1dx-fulfillment-workspace/issues/99) (parent #97)
**Plan:** `tasks/plans/FE-OD-IMPROVE-99/plan.md`
**Mockup:** `docs/mockups/order-detail-improve.html` § `.od-flow`
**Effort:** S · ~2h

#### Symptom
- Connector ของ node ปัจจุบันใน Order State Machine ไม่แสดง active ทางขวา (เช่น status=100 → ขวาเป็นเส้นจาง ทั้งที่ควร active)
- Header `"Order State Machine"` + `"… steps · Complete"` ยัง hardcoded EN

#### Root cause
- [`OrderStatusFlowBar.tsx:185-206`](../web-repo/apps/b1dx-oms-fulfillment/src/features/orders/components/order-detail/OrderStatusFlowBar.tsx#L185-L206) ใช้ `isDone` (klass === "completed" ของ node ซ้าย) → ถ้า node ซ้ายเป็น `current` → connector ทางขวาของ current จะถูก mark false ผิดเจตนา

#### Fix
- เปลี่ยน rule: connector "done" เมื่อ `klass(left) !== "future"` (completed **หรือ** current)
- i18n: `order_detail.state_machine_title` / `steps` / `complete_suffix` (en + th)

#### Test
- Component unit + E2E ครอบคลุม 3 cases (status 100 / 400 / 700) + i18n + regression `od-fidelity-01b`

---

### 🎯 WH-CAP Initiative — Peak Load Readiness (11/11 / 12.12) `[sompon-benz]`

**Found:** 2026-05-19 | **Type:** Capacity initiative | **Repo:** `webhook`
**Source:** code review 2026-05-19 — peak-load capacity analysis (full hot-path trace HTTP→RabbitMQ→worker→DB)
**Trigger events:** 11/11 (2026-11-11), 12.12 (2026-12-12)
**Status:** Tickets opened, no work started

#### TL;DR — รับไม่ไหวสำหรับ promotion peak
- **Ingestion (HTTP→RabbitMQ):** ~500-2000 req/sec ก่อนติด publisher channel mutex
- **Worker (RabbitMQ→DB):** **~1-3 msg/sec** (Lazada/TikTok), **~0.1-0.5 msg/sec** (Shopee!)
- 500 ออร์เดอร์/วินาที = queue โต **~500x** เร็วกว่า consume → broker disk เต็มภายใน 5-10 นาที

#### Failure timeline (500 orders/sec sustained, NO fix)
| T | สิ่งที่เกิด |
|---|---|
| T+0s | Gin spawn goroutines รับได้, publisher serialize ผ่าน channel mutex |
| T+5s | publisher contention → 202 latency พุ่งเป็นวินาที → platform timeout → retry ซ้ำ |
| T+30s | Queue `webhook.ingest` ~15k msgs, โต **497 msg/s** (in 500 / out ~1-3) |
| T+5-10m | Broker memory alarm → flow control → publishes block → Gin handlers hang → connection pile-up → **OOM api process** |
| ตลอด | Worker ยัง Shopee `SyncRecentOrders` ต่อ msg → ชน Shopee rate limit (~1000/min/shop) → 429 ทุก request |

**ผลลัพธ์:** ออร์เดอร์หาย, ซ้ำจาก retry, sellers เห็น lag เป็นชั่วโมง

#### Bottleneck ranking (worst → least) → ticket
1. Worker `Qos(1)` + 1 goroutine → **WH-CAP-01**
2. Shopee `SyncRecentOrders(1h)` per webhook → **WH-CAP-01**
3. Single shared `amqp.Channel` (no Confirm/reconnect/timeout) → **WH-CAP-02**
4. ไม่มี queue length limit / DLX → **WH-CAP-04**
5. HTTP server ไม่มี timeouts → **WH-CAP-03**
6. WebhookEvent.Create ไม่ idempotent → **WH-CAP-05**
7. Unbounded fan-out fallback → **WH-CAP-06**

#### 🔥 Critical pair — must-ship ก่อน 11/11
> **WH-CAP-01 + WH-CAP-04** (~1d + 1h) เปลี่ยนจาก "พังแน่" เป็น "ช้าแต่ไม่หาย"

#### Tickets summary
| ID | P | Effort | ผลที่ได้ | Depends on |
|---|---|---|---|---|
| WH-CAP-01 | P1 | 1d | 50-100x worker throughput | — |
| WH-CAP-04 | P1 | 1h | กัน broker เต็ม | — |
| WH-CAP-02 | P2 | 1-2d | durability + ingress 5-10k/s | — |
| WH-CAP-05 | P2 | 2h | กัน row ซ้ำจาก retry | — |
| WH-CAP-03 | P2 | 2h | กัน slow-loris + OOM | — |
| WH-CAP-06 | P2 | 2h | กัน goroutine ระเบิด | — |
| WH-CAP-07 | P2 | 1d | ไม่ชน marketplace rate limit | WH-CAP-01 |
| WH-CAP-08 | P3 | 1h | horizontal scale | WH-CAP-01 |

**Recommended execution order:**
WH-CAP-04 (1h) → WH-CAP-01 (1d) → WH-CAP-03/05/06 parallel (2h each) → WH-CAP-02 (1-2d) → WH-CAP-07 → WH-CAP-08

---

### 🆕 [P1] WH-CAP-01 — Worker concurrency + Shopee per-webhook full-resync rewrite `[sompon-benz]`

**Found:** 2026-05-19 | **Type:** Capacity / performance | **Repo:** `webhook`
**Source:** WH-CAP Initiative (bottleneck #1+#2)
**Plan:** `tasks/plans/WH-CAP-01/plan.md` (TBD)
**Mockup:** n/a
**Effort:** ~1d
**Depends on:** — (foundational; blocks WH-CAP-07 + WH-CAP-08)

#### Symptom
- **Current:** ~1-3 msg/sec (Lazada/TikTok), ~0.1-0.5 msg/sec (Shopee)
- **Target:** 50-100x throughput (~100+ msg/sec aggregate, p99 < 2s)
- 11/11 peak ~500 ออร์เดอร์/วินาที queue โต ~500x → broker disk เต็มใน 5-10 นาที

#### Root cause
- [internal/adapter/queue/rabbitmq_consumer.go:50](D:/Project/Project_2026/b1dx/b1dx-workspace/webhook/internal/adapter/queue/rabbitmq_consumer.go#L50) ใช้ `ch.Qos(1, 0, false)` + ประมวลผล 1 goroutine sequentially (`processDelivery` loop) — comment ระบุชัดว่า "Process one message at a time"
- [rabbitmq_consumer.go:155-162](D:/Project/Project_2026/b1dx/b1dx-workspace/webhook/internal/adapter/queue/rabbitmq_consumer.go#L155) เรียก Shopee `SyncRecentOrders(1 hour)` ต่อ webhook 1 event = paginated `GetOrderList` + `GetOrderDetail` หลายสิบ batch → 2-10 วินาที/webhook + ชน rate limit Shopee (~1000/min/shop)

#### Fix
- `ch.Qos(50, 0, false)` + spawn N worker goroutines อ่าน `msgs` channel เดียวกัน (ack ใน goroutine นั้น ๆ)
- เขียน `Shopee.SyncOrderByID(ordersn)` ใหม่ — เรียก `GetOrderDetail` แค่ออร์เดอร์เดียวจาก webhook payload
- ใช้ `SyncOrderByID` ใน hot path (เหมือน Lazada/TikTok) — ส่วน `SyncRecentOrders` เก็บไว้สำหรับ reconciliation/backfill job แยก

#### Test
- Unit: consumer ประมวลผลพร้อมกัน N msgs ไม่ duplicate ack
- Integration: TestContainers RabbitMQ + publish 100 msgs → wall-clock < (100 / N) × avg_latency + slack
- Shopee adapter: `SyncOrderByID` happy path + 404 + rate-limit retry
- Soak: 1000 msgs sustained → throughput ≥ 100 msg/s, p99 < 2s, no goroutine leak

#### Out of Scope
- Per-platform rate limiter (WH-CAP-07)
- Channel pool (WH-CAP-02)
- Horizontal scale (WH-CAP-08)

---

### 🆕 [P1] WH-CAP-04 — Queue safety: x-max-length + DLX + message TTL `[sompon-benz]`

**Found:** 2026-05-19 | **Type:** Resiliency | **Repo:** `webhook`
**Source:** WH-CAP Initiative (bottleneck #4)
**Plan:** `tasks/plans/WH-CAP-04/plan.md` (TBD)
**Mockup:** n/a
**Effort:** ~1h
**Depends on:** —

#### Symptom
- **Current:** queue ไม่มี cap → ถ้า worker ตามไม่ทัน (WH-CAP-01) ระบบล้มทั้งหมด
- **Target:** queue overflow → DLX (publish ยังคืน 202), broker disk ไม่เต็ม
- ลำดับเหตุการณ์ตอน peak: queue โต → RabbitMQ memory alarm → flow control → publish block → Gin handler hang → OOM api process

#### Root cause
- [internal/adapter/queue/rabbitmq_publisher.go:90-94](D:/Project/Project_2026/b1dx/b1dx-workspace/webhook/internal/adapter/queue/rabbitmq_publisher.go#L90) `QueueDeclare` ไม่ตั้ง `x-max-length`, ไม่ตั้ง `x-overflow`, ไม่มี DLX, ไม่มี TTL

#### Fix
- ตั้ง args ตอน declare main queue:
  - `x-max-length: 100000` (หรือค่าเหมาะกับ infra)
  - `x-overflow: reject-publish-dlx`
  - `x-dead-letter-exchange: webhook.dlx`
- Declare DLX + DLQ + TTL บน DLQ (เช่น 7 วัน) — กัน loop ระหว่าง main↔DLQ
- เพิ่ม metric `rabbitmq_dlx_messages_total` (ถ้ามี Prometheus)

#### Test
- Integration: publish > max-length → ตัวเกินไป DLQ + handler ได้ 202 (ไม่ block)
- Restart consumer → DLQ ไม่เคลียร์เอง (manual replay)
- Verify: main queue ที่ existing rabbit cluster declare ซ้ำได้โดยไม่ error (idempotent)

#### Out of Scope
- DLQ replay UI/CLI
- Metric/alerting setup (แยก task)

---

### 🆕 [P2] WH-CAP-02 — Publisher channel pool + Confirm mode + reconnect logic `[sompon-benz]`

**Found:** 2026-05-19 | **Type:** Durability / throughput | **Repo:** `webhook`
**Source:** WH-CAP Initiative (bottleneck #3)
**Plan:** `tasks/plans/WH-CAP-02/plan.md` (TBD)
**Mockup:** n/a
**Effort:** ~1-2d
**Depends on:** —

#### Symptom
- **Current:** ~1-5k publishes/sec single channel, ลดเหลือหลักร้อยภายใต้ contention; broker restart = ข้อความหาย; connection drop = ทุก publish fail จนกว่าจะ restart process
- **Target:** ingress ceiling 5-10k publishes/sec, durability ผ่าน Confirm ACK, auto-reconnect

#### Root cause
- [internal/adapter/queue/rabbitmq_publisher.go:18,50](D:/Project/Project_2026/b1dx/b1dx-workspace/webhook/internal/adapter/queue/rabbitmq_publisher.go#L18) ใช้ shared `*amqp.Channel` ตัวเดียว — channel ไม่ thread-safe
- ไม่เรียก `channel.Confirm(false)` + ไม่ subscribe `NotifyPublish`
- ไม่ subscribe `NotifyClose` + ไม่มี recreate channel/connection
- ไม่มี publish timeout

#### Fix
- Channel pool ขนาด `runtime.NumCPU()*2` (config ได้) — round-robin หรือ `sync.Pool`
- เปิด Confirm mode + รอ ack ผ่าน `NotifyPublish` พร้อม timeout 5s
- Subscribe `NotifyClose` → recreate connection+channels พร้อม exponential backoff
- เพิ่ม `Publish(ctx, ..., timeout)` API + return error ที่แยก timeout vs nack

#### Test
- Unit: pool round-robin + recreate after close
- Integration: kill broker mid-publish → reconnect + ไม่มี nil-deref + publish ต่อได้
- Soak: 1000 concurrent publish ผ่าน → ไม่มี deadlock, latency p99 < 50ms

#### Out of Scope
- Mandatory flag / return handler
- Persistent message flag (ตรวจสถานะปัจจุบันก่อน)

---

### 🆕 [P2] WH-CAP-05 — WebhookEvent idempotency (unique index + Upsert) `[sompon-benz]`

**Found:** 2026-05-19 | **Type:** Correctness | **Repo:** `webhook`
**Source:** WH-CAP Initiative (bottleneck #6)
**Plan:** `tasks/plans/WH-CAP-05/plan.md` (TBD)
**Mockup:** n/a
**Effort:** ~2h
**Depends on:** —

#### Symptom
- **Current:** marketplace retry บ่อยช่วง peak (timeout/202 ช้า) → row ใน `webhook_events` ซ้ำหลายเท่า → audit queries เพี้ยน + DB bloat
- **Target:** at-least-once delivery + DB-level dedup, 1 webhook event = 1 row regardless of retry count

#### Root cause
- [internal/adapter/repository/gorm_webhook_event.go:19-28](D:/Project/Project_2026/b1dx/b1dx-workspace/webhook/internal/adapter/repository/gorm_webhook_event.go#L19) `Create` เป็น plain INSERT ไม่ใช่ Upsert
- ไม่มี unique index บน `(platform, external_event_id)` หรือ equivalent ที่ platform ส่งมาเป็น dedup key

#### Fix
- เพิ่ม migration: unique index `(platform, external_event_id)` (snake_case ตาม project rule) — column `external_event_id` มีอยู่หรือต้องเพิ่ม ต้องตรวจ schema ก่อน
- เปลี่ยน `Create` → `clause.OnConflict.DoNothing()` + return early ถ้า duplicate (เพื่อ skip downstream)

#### Test
- Unit: publish event เดียวซ้ำ 5 ครั้ง → row เดียวใน DB
- Integration: ทดสอบ schema migration บน fresh DB + existing DB ที่มี duplicate (ต้อง dedupe ก่อน apply unique index)

#### Out of Scope
- Idempotency บน downstream order/product (มี Upsert อยู่แล้ว)
- Retention/archival ของ webhook_events เก่า

---

### 🆕 [P2] WH-CAP-03 — HTTP server timeouts + body size cap `[sompon-benz]`

**Found:** 2026-05-19 | **Type:** Security / resiliency | **Repo:** `webhook`
**Source:** WH-CAP Initiative (bottleneck #5)
**Plan:** `tasks/plans/WH-CAP-03/plan.md` (TBD)
**Mockup:** n/a
**Effort:** ~2h
**Depends on:** —

#### Symptom
- **Current:** unlimited timeouts (Go default = 0), no body-size cap → slow-loris connections เปิดทิ้งไว้ได้ไม่จำกัด
- **Target:** ReadTimeout 5s / WriteTimeout 10s / IdleTimeout 60s, MaxBodyBytes 1MB → connection cleanup เร็ว

#### Root cause
- [cmd/api/main.go:424](D:/Project/Project_2026/b1dx/b1dx-workspace/webhook/cmd/api/main.go#L424) ใช้ `r.Run(":port")` — Go default timeouts = 0 (unlimited), no `MaxHeaderBytes` override
- ไม่มี middleware `http.MaxBytesReader`

#### Fix
- Replace `r.Run(":port")` ด้วย:
  ```go
  srv := &http.Server{
    Addr: ":" + port, Handler: r,
    ReadHeaderTimeout: 2 * time.Second,
    ReadTimeout:       5 * time.Second,
    WriteTimeout:      10 * time.Second,
    IdleTimeout:       60 * time.Second,
    MaxHeaderBytes:    16 << 10, // 16KB
  }
  ```
- เพิ่ม graceful shutdown ผ่าน `srv.Shutdown(ctx)` ใน existing signal handler
- เพิ่ม middleware `c.Request.Body = http.MaxBytesReader(c.Writer, c.Request.Body, 1<<20)` (1MB)

#### Test
- Unit: middleware reject body > 1MB ด้วย 413
- Integration: slow-loris (write byte ละ 6s) → server close connection ภายใน timeout

#### Out of Scope
- Per-route timeout override
- Rate limiting (WH-CAP-07)

---

### 🆕 [P2] WH-CAP-06 — Bounded fan-out fallback (replace unbounded goroutines) `[sompon-benz]`

**Found:** 2026-05-19 | **Type:** Resource safety | **Repo:** `webhook`
**Source:** WH-CAP Initiative (bottleneck #7)
**Plan:** `tasks/plans/WH-CAP-06/plan.md` (TBD)
**Mockup:** n/a
**Effort:** ~2h
**Depends on:** —

#### Symptom
- **Current:** ตอน `seller_id` ว่างจาก webhook payload (legacy/empty) consumer fan-out sync ไปยัง **ทุก** tenant/account → goroutine ระเบิด + DB pool หมด + ชน rate limit marketplace ทุก shop พร้อมกัน + ขัดเจตนาของ 1-shop-1-tenant
- **Target:** max 8 concurrent fan-out goroutines (errgroup-bounded) หรือ skip + log WARN เมื่อ resolve ไม่ได้

#### Root cause
- [rabbitmq_consumer.go:193-202](D:/Project/Project_2026/b1dx/b1dx-workspace/webhook/internal/adapter/queue/rabbitmq_consumer.go#L193) `fanOutLazada` spawn `go func()` per entry ไม่จำกัด
- [rabbitmq_consumer.go:223-232](D:/Project/Project_2026/b1dx/b1dx-workspace/webhook/internal/adapter/queue/rabbitmq_consumer.go#L223) `fanOutTikTok` เช่นเดียวกัน

#### Fix
- ใช้ `golang.org/x/sync/errgroup` + `SetLimit(8)` หรือ semaphore
- พิจารณา skip fan-out + log WARN เมื่อ `seller_id` resolve ไม่ได้ (ดีกว่ายิงทุก tenant)
- เพิ่ม metric `webhook_fanout_fallback_total` (ถ้ามี Prometheus)

#### Test
- Unit: 100 entries + limit 8 → max 8 concurrent goroutines
- Integration: simulate empty seller_id → ตรวจว่าไม่ fan-out (ตามนโยบายใหม่)

#### Out of Scope
- Rewrite extractor (ของอื่น)

---

### 🆕 [P2] WH-CAP-07 — Per-platform / per-shop rate limiter (token bucket) `[sompon-benz]`

**Found:** 2026-05-19 | **Type:** Reliability | **Repo:** `webhook`
**Source:** WH-CAP Initiative (consequence of WH-CAP-01 — high concurrency จะชน marketplace rate limit)
**Plan:** `tasks/plans/WH-CAP-07/plan.md` (TBD)
**Mockup:** n/a
**Effort:** ~1d
**Depends on:** WH-CAP-01 (ก่อน worker scale ขึ้น rate limiter ไม่จำเป็น)

#### Symptom
- **Current:** sync service ไม่มี rate-limit gate → หลังเพิ่ม worker concurrency จะยิง marketplace API พร้อมกันเยอะ → 429 → ออร์เดอร์หายหรือ requeue loop
- **Target:** sustained API calls ≤ 80% ของ marketplace limit per shop (Shopee ~1000/min, Lazada ~500/min, TikTok configurable)

#### Root cause
- Sync service ของแต่ละ marketplace เรียก client API ตรง ๆ ไม่มี rate-limit gate
- Client retry นับ 429 เป็น generic error (ไม่ backoff ตาม `Retry-After` header)

#### Fix
- Token bucket per `(platform, shop_id)` — `golang.org/x/time/rate.NewLimiter(rate.Limit, burst)`
- ค่าจาก config: `lazada.rps`, `shopee.rps`, `tiktok.rps`
- Client adapter wrap call ผ่าน limiter
- Handle 429 → respect `Retry-After` + exponential backoff
- เพิ่ม metric `marketplace_rate_limited_total`

#### Test
- Unit: limiter block call ครบ burst + รอ refill
- Integration: mock platform return 429 → client retry ด้วย backoff + ไม่ exceed N attempts
- Soak: 100 concurrent SyncOrderByID → ไม่ exceed configured RPS

#### Out of Scope
- Global rate limit (cross-shop) ของ marketplace
- Adaptive rate (ปรับตาม 429 rate)

---

### [x] [P3] WH-CAP-08 — Horizontal scale worker pods + queue contention test `[sompon-benz]` ✅ Done 2026-05-25 (PR webhook#26)

**Found:** 2026-05-19 | **Type:** Infra / scaling | **Repo:** `webhook` + deploy
**Source:** WH-CAP Initiative (post-WH-CAP-01 scaling option)
**Plan:** [`tasks/plans/WH-CAP-08/plan.md`](plans/WH-CAP-08/plan.md) (Status: Done — merged `ba86b04`)
**Mockup:** n/a
**Effort:** ~1h infra + test
**Depends on:** WH-CAP-01 (vertical scale ต้องทำก่อน horizontal)

#### Symptom
- **Current:** 1 worker pod, ยังไม่ทดสอบ competing-consumers pattern
- **Target:** N pods กิน queue เดียวกัน (RabbitMQ load-balance) + verify exact-once processing

#### Root cause
- Worker deploy ปัจจุบัน 1 replica
- ยังไม่มี test ครอบคลุม `competing consumers` pattern

#### Fix (depends on WH-CAP-01 done first)
- ตั้ง `replicas: N` ใน docker-compose / k8s manifest
- ตรวจว่า consumer ใช้ `manual ack` + `prefetch` ทำงานถูก เมื่อมีหลาย consumer (RabbitMQ load-balance อัตโนมัติ)
- Integration test: 3 consumers + 100 msgs → แต่ละ msg ประมวลผลครั้งเดียว (ไม่ duplicate)

#### Test
- Integration: TestContainers RabbitMQ + 3 consumer instances → ตรวจ exact-once ผ่าน DB row count
- Production smoke: scale up 2 replicas ใน staging → metric `processed_total` รวมไม่เกิน publish count

#### Out of Scope
- Leader election สำหรับ scheduler/cron (cron ใช้ replica เดียวพอ)
- DB connection pool sizing per-replica (depend on infra)

---

### 🆕 [P2] WH-CAP-09 — Async admin sync endpoints (move /sync HTTP → queue-backed jobs) `[sompon-benz]`

**Found:** 2026-05-22 | **Type:** Refactor / API ergonomics | **Repo:** `webhook`
**Source:** WH-CAP-03 review — `WriteTimeout=10s` ใหม่จะ timeout endpoints ที่ทำ paginated sync
**Plan:** `tasks/plans/WH-CAP-09/plan.md` (TBD)
**Mockup:** n/a
**Effort:** ~3-4h
**Depends on:** WH-CAP-03 (creates the WriteTimeout constraint), WH-CAP-04 (queue safety baseline)

#### Symptom
- หลัง WH-CAP-03 deploy: POST /shopee/sync, /shopee/products/sync, /lazada/sync, /lazada/products/sync จะคืน timeout (~10s) เพราะ handler ทำ paginated marketplace fetch ที่ใช้เวลา 30s-2min
- Scheduler ปกติ (เรียกผ่าน service ตรง) ไม่กระทบ — เฉพาะ manual admin trigger ผ่าน HTTP

#### Root cause
- 4 admin sync handlers ทำงาน synchronous: `c.JSON()` รอจน `SyncRecentOrders` / `SyncProducts` วน paginate marketplace API จนจบ
- WriteTimeout 10s ของ WH-CAP-03 จำเป็นต่อ security ปิดช่อง slow-loris — ปรับขึ้นจะลด safety

#### Fix
- เปลี่ยน 4 admin sync handlers เป็น **async job pattern**:
  1. POST /sync → enqueue job (RabbitMQ message หรือ DB row `sync_jobs` table) → คืน 202 + `{"job_id": "..."}` ทันที (<10s)
  2. New endpoint GET /sync/jobs/{id} → คืน status (pending/running/done/failed) + count + error
  3. Worker consume job → run sync ใน background (no HTTP timeout)
- หรือทางง่ายกว่า: spawn goroutine + return 202 พร้อม progress endpoint (ไม่ persist, hi เริ่ม fresh ทุก deploy แต่ scope แคบกว่า)

#### Test
- Unit: handler returns 202 + job_id immediately
- Integration: job_id status transitions running → done; sync result == direct call result
- Manual: trigger /sync ด้วย body ที่ทำให้ sync วน > 30s → handler คืน 202 ภายใน 1s + GET /sync/jobs/{id} เห็น running → done

#### Out of Scope
- Sync job retry policy (ใช้ RabbitMQ DLX จาก WH-CAP-04 ครอบ)
- Web UI สำหรับ admin trigger + monitor (CLI / Postman พอสำหรับ ops)

---

### 🆕 [P3] FIX-FE-VITEST-FAILURES-01 — Fix 56 failing vitest tests across 11 files (mock drift) `[wat]`

**Found:** 2026-05-18 | **Type:** FE test maintenance | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** internal — `pnpm test --run` ใน `apps/b1dx-oms-fulfillment` แสดง 56 failed / 11 files / 2 failed suites; ส่วนใหญ่เป็น mock drift หลัง component/i18n เปลี่ยน
**Plan:** `tasks/plans/FIX-FE-VITEST-FAILURES-01/plan.md`
**Mockup:** n/a (test-only)

#### Failing files (cause summary)
| File | Fails | Cause |
|---|---|---|
| `OrderFilterBar.search.test.tsx` | 20 | mock `@b1dx/ui` ขาด `Input` (component switched `SimpleInputField` → `Input`) |
| `useOrderListViewModel.test.ts` | 9 | mock `react-i18next` ขาด `initReactI18next` export |
| `OrdersTable.test.tsx` | 10 | likely same i18n / text drift |
| `LoginForm.test.tsx` | 4 | `useAuth must be used within AuthProvider`; label via i18n key not matched |
| `ResetPasswordForm.test.tsx` | 5 | router invariant + auth provider missing |
| `ForgotPasswordForm.test.tsx` | 3 | `invariant expected app router to be mounted` |
| `SyncJobBanner.test.tsx` | 3 | `🛒` not in textContent; `banner-job-id` testid missing; `SHOPEE` text drift |
| `RoleOverviewCards.test.tsx` | 1 | `1 users` text not found (likely i18n key) |
| `BulkPrintTicketDialog.test.tsx` | 1 | text assertion drift (`✓ Shopee✗ Lazada — Network error`) |
| `ProfileSettingsContainer.test.tsx` | suite | file-load error |
| `Tab0General.test.tsx` | suite | file-load error |

#### Scope
- แก้ test/mock เท่านั้น — ไม่แก้ production code ยกเว้น restore testid ที่ component drift ลบไป
- 8 commits แยกตาม cause (ดู plan § Commit Plan)
- Fix file-load errors (2 failed suites) ก่อน — block อ่าน suite อื่น

#### Test
- `pnpm test --run` ผ่าน 0 fail / 0 failed suites
- `pnpm lint && pnpm typecheck` ผ่าน
- Manual spot-check orders list + login ใน browser → behavior unchanged
- ไม่มี new test case — restore existing assertions only

#### Out of Scope
- E2E suite (separate ticket if drift)
- Refactor shared mock helpers (defer)
- Re-architect AuthProvider/router test setup (use minimal wrapper)
- BE/Sync Worker tests
- Production bug discovered along the way → log new backlog + STOP, ไม่ bundle ใน PR

---

### 🆕 [P0] CRSET-02 Mockup Fidelity Audit (7 tickets) — Courier Drawer 5 tabs ไม่ตรง mockup `[srattha]`

**Found:** 2026-05-19 | **Type:** UI fidelity + BE contract gap | **Audit:** chat 2026-05-19 vs `docs/mockups/courier-settings.html`
**Total effort:** ~26h (1 fullstack dev, ~3.5 days)
**Decisions (2026-05-19):** ทำครบ 7 tickets · แยก `PUT /pickup` endpoint · เก็บ `weightFrom` column

#### Symptom
- **General Tab** coverage ~15% — ขาด 11 fields (code, mode, phone, website, note, supports*, has*, auto*), connection status row, 2 section toggles iOS-style, stats grid, view mode
- **Credentials Tab** coverage ~80% — ขาด view mode, manual-mode info-box, Test Connection wiring, apiSecret copy
- **Pricing Tab** coverage ~60% — ขาด minCharge/perKgExtra (DB มี), zone enum select, view mode, overlap validation
- **Pickup Tab** coverage ~70% — ขาด SLA field, warehouses hook (hardcoded `[]`), view mode, **endpoint รวมกับ general → concurrency risk**
- **Danger Tab** coverage ~75% — ขาด block for new courier, icon polish
- **Cross-cutting:** ไม่มี view mode (default = edit), `UpdateCourierRequest` ไม่ครบ field ตาม DB schema, Test Connection ไม่มี FE wiring, services shape inconsistency (`flags` vs `string[]`)

#### Root cause
- CRSET-02 รอบแรก (2026-05-15) merge แบบ MVP — defer fidelity work 9 items แตกเป็น CRSET-02A..D
- CRSET-02C-E2E + CRSET-02D ปิด test/audit-log gaps แล้ว แต่ visual fidelity ที่เหลือยัง defer
- BE schema (`couriers` table § 4.1) มี column ครบหมด — แต่ Application + Contract layer ไม่ expose

#### Tasks (ดูรายละเอียดใน plan.md ของแต่ละ ticket)

| # | Ticket | Layer | Effort | Priority | Branch |
|---|---|---|---|---|---|
| 1 | ~~**CRSET-02J**~~ ✅ merged 2026-05-19 BE contract: code+mode + `PUT /pickup` split + domain validation | BE | S ~1.5h | P0 | `feat/crset-02j-courier-be-contract` |
| 2 | **CRSET-02K** Drawer shell (view/edit, services flags, i18n) | FE | M ~4h | P0 | `feat/crset-02k-courier-drawer-shell` |
| 3 | ~~**CRSET-02E**~~ ✅ merged 2026-05-19 General Tab full rebuild | FE | L ~7h | P0 | `feat/crset-02e-courier-general-tab` |
| 4 | **CRSET-02F** Credentials Tab gaps | FE | M ~3h | P1 | `feat/crset-02f-courier-credentials-tab` |
| 5 | **CRSET-02G** Pricing Tab gaps + validation | FE | M ~3h | P1 | `feat/crset-02g-courier-pricing-tab` |
| 6 | ~~**CRSET-02H**~~ ✅ merged 2026-05-20 Pickup Tab + endpoint split | FE+BE | M ~4h | P1 | `feat/crset-02h-courier-pickup-tab` |
| 7 | ~~**CRSET-02I**~~ ✅ merged 2026-05-20 Danger Tab polish (new-mode block + icons) | FE | XS ~1h | P2 | `fix/crset-02i-courier-danger-tab` |
| 8 | ~~**CRSET-02H-FIX**~~ ✅ merged 2026-05-20 Pickup persist SLA + warehouse Guid + FE format validation (follow-up CRSET-02H) | FE+BE | L ~6h | P1 | `fix/crset-02h-pickup-persist-sla` |

#### Build order
- **Phase 1 (parallel):** CRSET-02J + CRSET-02K — blockers ของ tab tickets
- **Phase 2 (parallel หลัง Phase 1):** CRSET-02E + 02F + 02G + 02H + 02I
- **Phase 3:** Visual fidelity audit รวม 5 tabs + screenshot vs mockup

#### Test Cases (ครอบทุก ticket)
- [ ] General: 15 unit + 8 e2e cases (sections render, toggles, connection row, stats grid, view/edit transition)
- [ ] Credentials: 11 unit + 4 e2e (view mode mask, manual info-box, test connection, copy)
- [ ] Pricing: 12 unit + 4 e2e (minCharge, zone select, view mode, overlap validation)
- [ ] Pickup: 7 unit + 5 e2e (SLA, warehouses, endpoint split via network assertion)
- [ ] Danger: 7 unit + 2 e2e (new mode block, regression)
- [ ] BE: 13 unit + 7 integration (TestContainers)
- [ ] Shell: 11 unit + 5 e2e (view/edit transition, unsaved indicator, services shape)
- [ ] Visual fidelity: screenshot vs mockup 5 tabs (BLOCKING per CLAUDE.md)

---

### 🆕 [P3] WS-WEBHOOK-SUBMODULE-01 — Add webhook repo as submodule + webhook-coder expert agent + docs `[wat]` (reassigned: wat → sompon-benz → wat on 2026-05-19)

**Found:** 2026-05-18 | **Type:** Workspace infra / docs | **Repo:** `b1dx-fulfillment-workspace`
**Source:** internal — integrate `B1DXDev/b1dx-marketplace-webhook` (Go service) into 4-repo workspace pattern
**Plan:** `tasks/plans/WS-WEBHOOK-SUBMODULE-01/plan.md`
**Mockup:** n/a (no UI)

#### Scope
- Submodule: `b1dx-marketplace-webhook/` → `https://github.com/B1DXDev/b1dx-marketplace-webhook.git`
- Config: `workspace.local.example.json` + `workspace.local.json` → `webhook_repo_path`
- Doc: `CLAUDE.md` repos table + folder ref + agent table + ห้ามเขียน submodule list
- Agent: NEW `agents/webhook-coder.md` (Go + Docker/Caddy expert; pattern แบบ `sync-worker-coder.md`)
- Guide: NEW `docs/guides/webhook-coder-trigger-flow.md` (10-section trigger flow)
- Guide updates: `docs/guides/{agent-guide,git-submodules,github-branch-protection,progress-guide}.md` → 4-repo coverage
- Regen: `tasks/progress.html` via `node scripts/gen-progress.mjs`

#### Test
- `git submodule status` → 4 submodules (be / web / sync-worker / webhook)
- `b1dx-marketplace-webhook/go.mod` + `Dockerfile` + `Caddyfile` exist
- `workspace.local.json` parse + `webhook_repo_path` resolves to real dir
- `agents/webhook-coder.md` sections: Required Context / Verify Gate / Stack / Patterns / Done Gate
- `node scripts/gen-progress.mjs` exit 0

#### Out of Scope
- ไม่แตะ webhook repo code
- ไม่ migrate / สร้าง webhook feature task ใน backlog
- ไม่ setup CI สำหรับ webhook ใน workspace
- ไม่สร้าง orchestrator integration (จะทำเมื่อมี first webhook feature)

---

### ✅ [P2] FE-OD-FIDELITY-01a — Order Detail: wire missing sections (Customer / Payment / Shipping) `[srattha]` `Done 2026-05-22`

**Found:** 2026-05-17 | **Type:** FE visual fidelity gap | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user audit 2026-05-17 — "order detail ไม่เหมือน mock; expected เหมือน mockup 100%"
**Plan:** `tasks/plans/FE-OD-FIDELITY-01a/plan.md`
**Mockup:** `docs/mockups/order-detail-improve.html`

#### Gap (audit summary)
- `OrderCustomerSection`, `OrderPaymentSection`, `OrderShippingSection` มีอยู่จริงแต่ `OrderDetailView` ไม่ compose → user มองไม่เห็น

#### Goals
1. Wire 3 sections เข้า `OrderDetailView` ตามลำดับ mockup (Hero → Customer → Info → Payment → Shipping → Items → Timeline)
2. Verify internal rendering ของแต่ละ section ตรง mockup
3. Extend MSW handler / type ถ้า field ขาด

#### Test
- Unit: 3 section testid present in DOM, section order matches mockup, sub-blocks render per data
- E2E: open detail → 3 sections visible, screenshot vs mockup

#### Out of Scope
- Status flow bar (01b), Hero/info polish (01c), Timeline/sidebar polish (01d)
- EditOrderDialog, BE schema change (flag follow-up if field gap)

---

### ✅ [P2] FE-OD-FIDELITY-01b — Order Detail: Status Flow Bar (horizontal state machine) `[srattha]` `merged: 2026-05-22`

**Found:** 2026-05-17 | **Type:** FE missing component | **Repo:** `b1dx-oms-fulfillment-web`
**Depends on:** FE-OD-FIDELITY-01a (merged)
**Plan:** `tasks/plans/FE-OD-FIDELITY-01b/plan.md`
**Mockup:** `docs/mockups/order-detail-improve.html` (`.od-flow` block)

#### Gap
- Mockup มี horizontal state flow bar ที่ top ของ main column — app ไม่มี (มีแค่ vertical `OrderTimeline`)

#### Goals
1. NEW `OrderStatusFlowBar` — horizontal scrollable nodes (completed / current / future + connectors)
2. Wire above Customer section
3. Reuse workflow state list (single source of truth)

#### Test
- Unit: all states render, current/completed/future classes, connectors n-1, overflow scroll, helper unit
- E2E: state X → node `[data-state="current"]` matches X, mobile horizontal scroll works

#### Out of Scope
- Status transition logic (existing `OrderStatusDropdown`)
- Workflow editor

---

### ✅ [P2] FE-OD-FIDELITY-01c — Order Detail: Hero + Info section polish `[srattha]` `merged: 2026-05-22`

**Found:** 2026-05-17 | **Type:** FE visual fidelity polish | **Repo:** `b1dx-oms-fulfillment-web`
**Depends on:** FE-OD-FIDELITY-01a (merged)
**Plan:** `tasks/plans/FE-OD-FIDELITY-01c/plan.md`
**Mockup:** `docs/mockups/order-detail-improve.html`

#### Gap
- Hero: missing breadcrumb, static gradient (mockup = dynamic per status), no external order ID tag
- Info grid: 2-col (mockup = 3-col desktop), missing Service Policy + SLA, COD always shown, platform text only (mockup = logo)

#### Goals
1. Hero: breadcrumb, dynamic gradient via `getStatusGradient(status)`, external ID tag
2. Info: 3-col desktop, Service Policy + SLA, conditional COD, platform logo
3. Add i18n keys (th + en)

#### Test
- Unit: breadcrumb render, gradient per status, ext-ID conditional, 3-col grid, Service Policy + SLA, COD conditional, platform `<img>`, helper unit
- E2E: breadcrumb nav, desktop 3-col, tablet 2-col, COD conditional, i18n switch no Thai leak

#### Out of Scope
- Section wiring (01a), Status flow bar (01b), Timeline+sidebar (01d), status transition logic

---

### 🆕 [P2] FE-OD-FIDELITY-01d — Order Detail: Timeline + Sidebar polish `[srattha]`

**Found:** 2026-05-17 | **Type:** FE visual fidelity polish | **Repo:** `b1dx-oms-fulfillment-web`
**Depends on:** FE-OD-FIDELITY-01a (merged)
**Plan:** `tasks/plans/FE-OD-FIDELITY-01d/plan.md`
**Mockup:** `docs/mockups/order-detail-improve.html`

#### Gap
- Timeline dot 2.5px (mockup = 20px + pulse on latest + issue badges)
- Sidebar width 300px (mockup = 288px)
- Mobile responsive: not verified for full page

#### Goals
1. Timeline: 20px dot, pulse on latest event, issue badge per event severity
2. Sidebar: 288px width, mobile stack (collapse below main < md)
3. Verify full-page responsive across breakpoints

#### Test
- Unit: dot 20px class, latest has `animate-pulse`, severity badge conditional, sidebar 288 class
- E2E: desktop sidebar right 288, mobile stack below, latest dot pulses, severity badge visible

#### Out of Scope
- Sidebar internal sections (already OK per audit), Customer/Payment/Shipping wiring (01a), Status flow bar (01b), Hero/info polish (01c)

---

### 🆕 [P2] FE-OD-FIDELITY-01e — Order Detail: Items Section od-card fidelity `[srattha]`

**Found:** 2026-05-22 (user chat report) | **Type:** FE visual fidelity polish | **Repo:** `b1dx-oms-fulfillment-web`
**Depends on:** FE-OD-FIDELITY-01a (merged)
**Plan:** `tasks/plans/FE-OD-FIDELITY-01e/plan.md`
**Mockup:** `docs/mockups/order-detail-improve.html` §`.od-card` items (lines 177–254, 1168–1216)

#### Gap
- Card uses `shadow-sm` + `rounded-xl` (mockup = no shadow + `--radius` + `--bg2`)
- Card title `text-body font-semibold` (mockup = mono 10.5px bold tracking-.06em)
- Table th no bg, `border-b-2` (mockup = `--bg3` fill + border 1px)
- Item icon 34×34 (mockup = 30×30 with `--bdr2`)
- **Missing match-status pill** ใต้ SKU (`✓ Matched` / `⟳ กำลังจับคู่` / `✕ ไม่พบ SKU`)
- Gift indicator = "ของแถม" pill (mockup = mono 8px `GIFT` tag)
- Totals layout: `justify-end` mini-cols ฝั่งขวา (mockup = 4 full-width equal-flex cols + indigo TOTAL bg + 15px value)
- Discount =0 → `−฿0` (mockup = `—` em-dash)

#### Goals
1. Rebuild card chrome to use `--bg2`/`--bg3`/`--bdr`/`--bdr2`/`--dim`/`--muted`/`--radius`
2. Card header/title/badge typography per mockup
3. Table th/td padding 9px 12px + th `--bg3` background
4. Item icon 30×30 + rounded-7px + `--bdr2` border
5. Add match-status pill (3 states) — derive from `orderStatus===100` cascade + optional per-item override
6. Gift tag mono 8px green-bg border + "GIFT" label
7. 4-col full-width totals row + indigo TOTAL col + 15px value
8. Discount em-dash when 0/undefined

#### Test
- Unit: 14 cases — card chrome, title/badge style, 3 item rows, icon 30px, gift tag, gift total, match pill default/warn/err, 4 totals cols, TOTAL indigo+15px, discount em-dash, th bg
- E2E: 4 cases (items visible, 4 th columns, first row icon+name+sku+match pill, totals 4 cols TOTAL 15px); 2 skipped pending mock data (gift item + status=100 order)

#### Out of Scope
- BE schema change for per-item match status / gift flag
- Real SKU matching logic (FE derive จาก orderStatus เหมือน mockup)
- Other sections (01a/b/c/d done)
- Items price/total calculation logic

---

### ✅ [P2] FE-OD-FIDELITY-01f — Order Timeline (Status History) 100% mockup fidelity `[srattha]` `Done 2026-05-22`

**Found:** 2026-05-22 (user chat report) | **Type:** FE visual fidelity polish | **Repo:** `b1dx-oms-fulfillment-web`
**Depends on:** FE-OD-FIDELITY-01d (merged — partial timeline polish done, fidelity gaps remain)
**Plan:** `tasks/plans/FE-OD-FIDELITY-01f/plan.md`
**Mockup:** `docs/mockups/order-detail-improve.html` § `renderTimeline()` (lines 1218–1261) + CSS `.od-card-*` (177–192), `.od-tl-*` (256–287), `.spill.s*` (289–308)

#### Gap (current vs mockup)
- Card title `"Status History"` text-sm bold foreground (mockup = `"STATUS HISTORY"` mono 10.5px bold muted tracking-.06em)
- Missing `{n} events` badge (mockup mono 9.5px dim)
- Has `⟶ Flow` button in header (mockup ไม่มี)
- Sort = desc newest top (mockup = asc oldest top → newest bottom per TIMELINE_DATA)
- Dot.done: bg-current/20 circle ภายใน 1.5px (mockup = bg currentColor solid + ✓ checkmark SVG polyline)
- Dot.current: bg-transparent + circle 2px ภายใน (mockup = transparent + animate-pulse + 8px solid inner dot)
- Line = 1px bg-border (mockup = 2px bg-`--bdr`)
- Status row: ไม่มี `● ปัจจุบัน` indicator (mockup = amber 10px อยู่ข้าง pill ของ latest)
- Pill: `bg-current/10` (mockup = `spill.s###` token bg per status — amber-bg/ind-bg/teal-bg/red-bg/etc.)
- Meta row: ขาด `✓` ตอนท้าย row ของ done items
- Note: ใช้ severity-driven warn/error tint (mockup = `issue` flag → amber border+bg+text)
- Severity badge `⚠ warn` / `✕ error` pill แยก (mockup ไม่มี — รวมเป็น note style แทน)

#### Goals
1. Card chrome `bg-[var(--bg2)] border-[var(--bdr)] rounded-[var(--radius)]` no shadow
2. Mono header "STATUS HISTORY" + mono `{n} events` badge
3. Asc sort entries (oldest → newest)
4. Dot.done bg-current/13 + ✓ checkmark SVG polyline (1.5,4.5 → 3.5,7 → 7.5,1.5)
5. Dot.current bg-transparent + animate-pulse + 8px solid inner dot
6. 2px vertical line `bg-[var(--bdr)]`
7. Status row: colored label + `spill.s###` 8.5px mono pill + amber `● ปัจจุบัน` for latest
8. Meta row 10.5px dim: actor + mono ts + `✓` for done
9. Note default `bg-[var(--bg3)]` border `--bdr`; issue (severity warn/error OR toStatus∈{600,620,800,900}) → amber bg+border+text
10. ลบปุ่ม `⟶ Flow` (mockup ไม่มี)
11. ลบ severity-badge pill (รวมเข้า issue note)

#### Test
- Unit: 14 cases — asc sort, label+pill+actor+ts, note text, empty state, header text + events badge, no Flow button, latest dot pulse single instance, done dots ✓ SVG, ● ปัจจุบัน, ✓ in meta, issue note for severity warn/error/toStatus∈issue-codes
- E2E: 7 cases — header "STATUS HISTORY", `{n} events` badge regex, no Flow button, asc sort by data-to-status, ● ปัจจุบัน indicator, latest dot pulse, screenshot artifact
- Regression: 01d E2E severity-badge test → issue-note test (pass ✓)

#### Out of Scope
- BE/DTO change — `severity?: 'warn' | 'error'` field คงอยู่ใน schema
- `OrderDetailView` orchestration (props signature ไม่เปลี่ยน)
- Sidebar / Order Info / Items / Hero (01a/b/c/d/e done or separate)
- Status color palette/token rework (ใช้ tokens ที่มีใน globals.css)

#### Status (2026-05-22)
- Plan Draft → Confirmed → In Progress → In Review → ✅ Done
- PR #242 merged manual squash on `b1dx-oms-fulfillment-web` (main `6d11ad0`)
- Unit 14/14 ✅, E2E 01f 7/7 ✅, E2E 01d 5/5 ✅
- Post-review fix-up: dropped `orange` token (750 → red per mockup priority), `mt-1.5` → `mt-[5px]` on note, removed redundant `isLast`, E04 e2e strict monotonic via `data-occurred-at`

---

### ✅ [P3] FONT-SIZE-STD-PREF-01 — Standardize font-size scale + improve user preference UI `[wat]` ✅ Done 2026-05-17 (PR #201)

**Found:** 2026-05-16 | **Type:** FE design-system + UX improvement + minor bug | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user request 2026-05-16 — "Improve font size: สรุป default font size แต่ละส่วน เพื่อให้ user สามารถปรับเองได้ เพื่อกำหนดเป็นมาตรฐานทั้ง app"
**Plan:** `tasks/plans/FONT-SIZE-STD-PREF-01/plan.md`

#### Goals
1. กำหนด 5 semantic font-size tokens (`--text-display/title/heading/body/label`) เป็นมาตรฐานทั้ง app
2. ย้าย font-size preference จาก slider (90-120%) ใน "การเข้าถึง" → 4 discrete radio (sm/md/lg/xl) ใน "การแสดงผล" ใต้ Density
3. Fix bug: `applyFontSize` set `#app-root style.fontSize` แต่ token Tailwind classes ใช้ rem → preference ไม่ scale → fix ด้วย `--text-scale` CSS var multiplier บน `<html>`

#### Scope
- `apps/.../app/globals.css` — add 5 `--text-*` token vars + `--text-scale` multiplier + `html[data-text-scale="*"]`
- `apps/.../tailwind.config.ts` — extend `fontSize` map backed by token vars
- `apps/.../lib/prefEffects.ts` — rewrite `applyFontSize` (set `--text-scale` on `<html>`, not `#app-root` inline px)
- `apps/.../types/prefs.ts` — add `textScale` enum field
- `apps/.../components/preferences/PanelAppearance.tsx` — add Text Size radio group under Density
- `apps/.../components/preferences/PanelAccessibility.tsx` — remove font-size slider, add helper link
- `apps/.../lib/i18n/locales/{th,en}.json` — `prefs.appearance.text_size.*` keys
- `packages/ui/.../button.tsx`, `badge.tsx`, `dialog.tsx` — replace hardcoded `text-*` with semantic token classes
- `apps/.../scripts/codemod-text-sizes.mjs` (NEW) — replace ~600 `text-[Npx]` arbitrary classes
- ~50 files under `apps/.../features/**` — codemod sweep (commit per directory)

#### Test
- Unit: `applyFontSize` writes `--text-scale` to `<html>` not `#app-root` (regression); `usePreferences` default + persist `textScale`; `<PanelAppearance>` 4 radio + change + reset; `<PanelAccessibility>` slider removed + helper link
- Visual/Integration: computed font-size for page title (20px @ md, 25px @ xl), dialog title (18px), table cell (12px), button (14px), pagination (12px)
- E2E: open settings dialog → Appearance → select lg → reload → persisted; reset → defaults; Accessibility tab no slider

#### Out of Scope
- BE migration / endpoint change (reuse existing `preferences_font_size` int column)
- Density option changes (keep compact/default/comfortable)
- Theme / accent color / font family
- Print/PDF layout tuning
- Mockup HTML regeneration

**Effort:** M, ~6-8h | **Priority:** P3 (design-system + UX improvement, no blocking flow)
**Branch:** `feat/font-size-std-pref-01`

---

### 🆕 [P3] IMPROVE-FE-FILTER-01 — `/orders` Filter Bar improvements (remove pageSize · i18n · datepicker dark+format · reset always-visible) `[wat]`

**Found:** 2026-05-16 | **Type:** FE improvement | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user request 2026-05-16 — visit `/orders` → improve filter bar
**Plan:** `tasks/plans/IMPROVE-FE-FILTER-01/plan.md`

#### Goals
- ลบ `<select>` pageSize dropdown ออกจาก filter bar (pager handle size เอง)
- i18n: เติม keys ที่ยังเป็น inline fallback ลง `en.json` + `th.json`
- Datepicker: dark theme รองรับ + format ตาม `preferences.dateFormat` ของ user
- Reset button visible เสมอ (ปัจจุบัน gate ด้วย `hasActiveFilters`)

#### Scope
- `apps/.../order-list/OrderFilterBar.tsx` — remove pageSize block + always-visible reset + pass `displayFormat`
- `packages/ui/src/components/forms/inputs/RHFDatePicker.tsx` — dark theme + `displayFormat?: string` prop
- `apps/.../OrderListContainer.tsx` (or `useOrdersQuery`) — drop `pageSize` from form values → hard default `10`
- `apps/.../lib/i18n/locales/{en,th}.json` — add missing `orders.filter.*` keys

#### Test
- Unit (vitest): pageSize select gone; Reset always visible; Reset click → defaults; `displayFormat` forwarded from prefs; dark class applied on RHFDatePicker
- E2E (Playwright): no pageSize dropdown; reset visible+functional; dark theme readable; TH→EN no Thai leak; date format from prefs renders

**Effort:** S, ~2-3h | **Priority:** P3 (UX improvement, not blocking — pager already paginates)
**Branch:** `feat/improve-fe-filter-01`

---

### 🆕 [P2] STATUS-CHIP-01 — `/orders` table status column ไม่มี chip สำหรับ status 550/750/900 `[wat]`

**Found:** 2026-05-16 | **Type:** FE display bug | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user report 2026-05-16 — visit `/orders` → orders status 550/750/900 ใน table list
**Plan:** `tasks/plans/STATUS-CHIP-01/plan.md`

#### Symptom
- Status column ของ orders ที่มี status 550 (OutForDelivery), 750 (CancelRequested), 900 (Returned) แสดงเป็น muted code-only span (เช่น `550`) ไม่มี chip background / dot / label เหมือน status อื่น (100/200/.../800)

#### Root cause
- `OrderStatusChip.tsx` `ORDER_STATE_MAP` ขาด entries สำหรับ 550, 750, 900 → fall through `unknown` fallback branch ที่ return muted code-only span
- `STATUS_COLORS` และ `OrderStatusBadge` มีครบทุก code แต่ `OrderStatusChip` (component ที่ใช้ใน table) ขาด

#### Fix
- เพิ่ม 3 entries ใน `ORDER_STATE_MAP`:
  - `550: { label: 'ออกส่ง', color: 'teal' }` (last-mile family, distinct from 500 "กำลังส่ง")
  - `750: { label: 'ขอยกเลิก', color: 'amber' }` (pending action; distinct from 800 Cancelled red)
  - `900: { label: 'คืนสินค้า', color: 'red' }` (terminal return)

#### Test
- Unit (vitest): regression 3 cases — `state=550/750/900` → render full chip + dot + label + correct `data-color`
- Manual smoke: `/orders` chip visible matching 100/500/800 style

**Effort:** XS, ~1h | **Priority:** P2 (display fidelity; data + actions ยังถูกต้อง)
**Branch:** `fix/status-chip-01`

---

### 🆕 [P3] WORKFLOW-GH-SYNC-01 — GitHub Issue ↔ plan.md ↔ Project board auto-sync `[wat]` `[ACTIVE — In Progress]`

**Found:** 2026-05-26 | **Type:** Workflow tooling (no app code change) | **Repo:** `b1dx-fulfillment-workspace` (scripts + docs)
**Source:** user request 2026-05-26 — GH Project board ค้าง Todo แม้งานเริ่มแล้ว · agent + dev ต้อง sync state ทั้ง 3 จุด (plan.md / GH issue / Project board) ด้วยมือ → automate
**Plan:** [`tasks/plans/WORKFLOW-GH-SYNC-01/plan.md`](plans/WORKFLOW-GH-SYNC-01/plan.md)
**Issue:** [#123](https://github.com/B1DXDev/b1dx-fulfillment-workspace/issues/123) (auto-created by script during build trigger)
**Branch:** `chore/workflow-gh-sync-01`
**Effort:** M, ~6h | **Priority:** P3 (foundational — unlocks Slice B/C/D + dovetails with QA-TC-* epic)
**Scope:** core `scripts/gh-issue-sync.mjs` + CLAUDE.md + plan template + assignments README · Slice B/C/D deferred

---

### 🆕 [P3] QA-TC-FORMAT-01 — Standardized 12-col Test Case Template + Tooling (epic — 8 slices) `[wat]` `[ACTIVE — Block 1A first]`

**Found:** 2026-05-24 | **Type:** Documentation + tooling (no app code change) | **Repo:** `b1dx-fulfillment-workspace` (docs + scripts) + cross-repo loaders (FE/BE/Webhook)
**Source:** user request 2026-05-24 — post Block 0 done; devs (wat/srattha/nuchit) ต้องการ template เดียวกัน + test cases สำหรับ task ใหม่ทุก feature/bug
**Plan:** `tasks/plans/QA-TC-FORMAT-01/plan.md` (parent epic)
**Priority Doc:** `tasks/assignments/2026-05-wat-priority.md` § Block 1A (revised — split from full Block 1)

#### Goal
Standardize test case format ทั่ว 4 repos → devs/reviewers/loaders ใช้ template เดียวกัน · Phase 2 unlock auto-spec generation (Playwright/xUnit/Go).

#### Decision Record (locked 2026-05-24)
**12-col schema** (vs 15-col / 10-col / 8-col): TC ID, Module, Role, Test Type, Priority, Pre-Conditions, Scenario, Test Steps, Test Data, Expected Result, Status (manual), Defect/Notes (manual). Drop Sub-module (= H2 section), Severity (= Priority), Actual Result (manual Excel), Executed By/Date (= git blame).

#### Slices (8) — Recommended order ตาม dependency

| # | ID | Title | Effort | Depends |
|---|---|---|---|---|
| 1 | ✅ **QA-TC-01** | Template (12-col) + CSV + workflow guide — **Done PR #110, 2026-05-25** | 0.5d | — |
| 2 | **QA-TC-02** | CLAUDE.md + plan template + PR template wire — 🟡 **In Progress** (issue #111, PR #119) | 0.5d | ✅ #1 |
| 3 | **QA-TC-06** | Playwright loader (FE TS) — 🟡 **In Progress** (issue #113, branch `docs/qa-tc-06-pw-loader`) | 1d | ✅ #1 |
| 4 | QA-TC-04 | Generator script (permutation matrix) | 1d | #1, #3 |
| 5 | QA-TC-05 | Validator script (lint + sync) | 1d | #4 |
| 6 | QA-TC-07 | xUnit loader (BE + Worker) | 1d | #1, #3 |
| 7 | QA-TC-08 | Go test loader (Webhook) | 0.5d | #1, #3 |
| 8 | QA-TC-03 | Agent prompts update (10 agents) | 1d | #1, #2 |

**Critical path:** #1 → #2/#3 parallel → #4/#6/#7 → #5/#8 → #8 (agents last)

#### Scope
- 8 slice deliverables (per-slice plan.md ภายใต้ epic)
- Cover 4 repos: FE (Playwright/Vitest) + BE (xUnit) + Sync (xUnit) + Webhook (Go test)
- Pattern: `docs/testing/_templates/` directory ตั้ง standard

#### Out of Scope
- ❌ Bulk migrate legacy 243 TCs (Order Module) — forward-looking template · lazy migrate on touch
- ❌ Jira/Xray/TestRail import — internal team, no external tool
- ❌ App code change (FE/BE/Sync/Webhook) — pure docs + tooling
- ❌ Performance/load test template — functional only

#### Test
- Per-slice plan.md ระบุ verify steps
- Adoption check: 1 TC ใน Block 2 P1 bug (BUG-FE-MOCK-UUID-01) ใช้ template ผ่าน reviewer
- Loader smoke: parse 1 known TC → emit spec (Playwright/xUnit/Go)
- Validator: detect missing col / invalid Priority value / TC ID dup

**Effort:** XL, ~5.5d total (revised from 6.5d under 12-col decision) | **Priority:** P3 (process standardization; block Phase 2 auto-spec)
**Branch:** `docs/qa-tc-format-01` (parent) + per-slice branches (`docs/qa-tc-0{1..8}-{slug}`)

---

### ✅ [P0] QA-ORDER-SM-01 — Order Module Test Artifacts (state machine HTML + matrix + step-by-step cases) `[wat]` `[DONE 2026-05-24 — PR #91]`

**Found:** 2026-05-24 | **Done:** 2026-05-24 (same day) | **PR:** #91 (merged to main) | **Type:** Documentation / QA artifacts (no Order Module code change) | **Repo:** `b1dx-fulfillment-workspace` (docs only)
**Source:** user request 2026-05-24 — เช็ค state machine ที่หน้า Order List + Order Detail ก่อน รวม role permission ครบทุกเคส
**Plan:** `tasks/plans/QA-ORDER-SM-01/plan.md`
**Priority Doc:** `tasks/assignments/2026-05-wat-priority.md` (Block 0 — TOP)

#### Goal
Self-served QA test artifacts for Order Module — HTML state machine diagram (visual) + matrix doc (state×role×action) + step-by-step manual test cases ครบทุก scenario (Order List + Order Detail).

#### Slices (4)
| ID | Title | Effort | Output |
|---|---|---|---|
| **SM-01a** | HTML mermaid state machine flow doc | 1d | `docs/guides/9.order-state-machine.html` |
| **SM-01b** | Test matrix verified vs code (5 matrices) | 0.5d | `docs/testing/order-module/test-matrix.md` |
| **SM-01c** | Order List step-by-step TCs (~80-100 cases) | 2d | `docs/testing/order-module/tc-order-list.md` |
| **SM-01d** | Order Detail step-by-step TCs (~80-120 cases) | 2d | `docs/testing/order-module/tc-order-detail.md` |

#### Scope
- 4 deliverables ใน `docs/guides/` + `docs/testing/order-module/`
- 8-col Excel-friendly format (NOT 15-col QA-TC-FORMAT template — deferred)
- Cross-check vs `OrderStatusTransitionService.cs`, `order-action-permissions.ts`, `bulkActions.ts`, `OrderContextMenu.tsx`, `OrderActionPanel.tsx`

#### Out of Scope
- ❌ Modify Order Module code (pure docs)
- ❌ Playwright `.spec.ts` codegen (manual first)
- ❌ BE / Sync Worker / Webhook test cases
- ❌ 15-col template adoption (QA-TC-FORMAT-01 deferred)

**Effort:** XL, ~5.5d total (44h) | **Priority:** P0 (user pivot — block 0)
**Branch:** `docs/qa-order-sm-01` (parent) + per-slice branches

---

### 🆕 [P1] BUG-FE-PRINT-LABEL-01 `[POSTPONED — QA-ORDER-SM-01 prioritized]` — Print Label ไม่ปริ้นจริง (2 flows: BulkPrintLabelDialog + OrderActionDialog 480→490) `[wat]`

**Found:** 2026-05-16 | **Type:** FE behavior bug | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user report 2026-05-16 — `/orders` → `order-osm-st480` → ⋯ → "Print Shipping Label" (Flow A) หรือ "🏷 Label" transition 480→490 (Flow B) → submit → dialog แสดง complete + toast แต่ "ไม่ปริ้น label จริง"
**Plan:** `tasks/plans/BUG-FE-PRINT-LABEL-01/plan.md`

#### Symptom
- **Flow A** (Tools › "Print Shipping Label" → `BulkPrintLabelDialog`): submit → dialog ปิด + (บางครั้ง) error toast `เบราว์เซอร์บล็อกการพิมพ์...` แต่ไม่มี success toast และ **ไม่มี print dialog**
- **Flow B** (transition "🏷 Label" 480→490 → `OrderActionDialog`): submit → state เปลี่ยน + success result screen 1.2s → ปิด dialog → **ไม่ปริ้น PDF** เลย

#### Root cause
- **Flow A:** `lib/printPdf.ts:openAndPrint` ใช้ hidden iframe + `iframe.contentWindow.print()` — Chrome PDF viewer plugin loaded ใน iframe ไม่ expose `print()` ผ่าน contentWindow consistent → silently fail หรือ throw → onBlocked toast แสดง แต่ไม่ปริ้น. ไม่มี success toast ใน `useBulkPrintLabel.submit` หลัง print fires
- **Flow B:** `OrderActionDialog.handleSubmit` toState=490 transition success → `setScreen('success')` แล้ว `onSuccess()` แค่ปิด dialog — **ไม่ trigger print PDF**. Mockup/spec expect ว่า 480→490 = "Print Label + assign courier" ควรปริ้น label หลัง state change

#### Fix
- **printPdf.ts:** สลับเป็น `window.open(blobUrl)` + onload `win.print()` — เปิดใน new tab + browser native print toolbar; pre-open window ใน user-gesture (click handler) ก่อน async fetch กัน popup blocker
- **useBulkPrintLabel.ts:** pre-open window + เพิ่ม `toast.success('พิมพ์ Label สำเร็จ')` หลัง print fires
- **useBulkPrintTicket.ts:** collateral fix (ใช้ openAndPrint ตัวเดียวกัน) + success toast non-pdf
- **OrderActionDialog.tsx:** toState=490 onSuccess → chain `/orders/print-pdf?ids={id}&jobType=label&format={labelSize}` → trigger print + toast
- **NEW** `useTransitionPrintLabel.ts` — wraps `useOrderTransition` + post-success print fetch

#### Test
- Unit (vitest): `openAndPrint` window.open variant (3 cases); `useBulkPrintLabel` regression + success toast; `useBulkPrintTicket` non-pdf success; `OrderActionDialog` toState=490 chains print + อื่นๆ ไม่ chain
- E2E (Playwright): Flow A new tab + toast; Flow B state change + new tab; popup-blocked → onBlocked toast; status=700 regression (ไม่มี print item)

**Effort:** S, ~2-3h | **Priority:** P1 (broken feature ทั้งสอง print flow ใน /orders — blocks demo/UAT)
**Branch:** `fix/bug-fe-print-label-01`

---

### 🆕 [P2] BUG-FE-PACK-REQUIRED-01 — "Pack เสร็จ" (450→480) empty submit แสดง required แค่ weight + message ยาว `[wat]`

**Found:** 2026-05-16 | **Type:** FE validation UX bug | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user report 2026-05-16 — `/orders` → `order-osm-st450` → ⋯ → "Pack เสร็จ" → click confirm without filling info
**Plan:** `tasks/plans/BUG-FE-PACK-REQUIRED-01/plan.md`

#### Symptom
- empty submit → required error แสดงเฉพาะ weight (W/L/H ไม่ขึ้น)
- error message ยาว `"Expected number, received string"` (Zod default) แทน Thai สั้น

#### Root cause
- `RHFDecimalInput` field value type = `string` (empty = `""`). Schema field = `z.number().min(0.01).max(999.99).optional()` — empty `""` fails `z.number()` ก่อน superRefine → Zod คืน default `"Expected number, received string"`. `optional()` allow undefined อย่างเดียวไม่ allow ""
- `order-transition.schema.ts:104-107` case 480 require เฉพาะ `weightKg` ไม่ require W/L/H
- `OrderActionDialog.tsx:484-488` W/L/H ไม่ได้ใส่ `required` prop

#### Fix
- Schema case 480: preprocess `weightKg`/`widthCm`/`lengthCm`/`heightCm` ("" → undefined) + require ทั้ง 4 + Thai short msgs (`ระบุน้ำหนัก` / `ระบุกว้าง` / `ระบุยาว` / `ระบุสูง`)
- UI: `required` prop ที่ W/L/H RHFDecimalInput

#### Test
- Unit (vitest): regression `createTransitionSchema(450, 480)` + empty all 4 → 4 issues + Thai short
- E2E + manual: st450 → "Pack เสร็จ" → empty submit → 4 inline errors

**Effort:** XS, ~1h | **Priority:** P2 (validation UX gap; flow ทำงานได้ถ้ากรอกครบ)
**Branch:** `fix/bug-fe-pack-required-01`

---

### 🆕 [P1] BUG-FE-MOCK-UUID-01 `[POSTPONED — QA-ORDER-SM-01 prioritized]` — MSW mock user IDs ไม่ใช่ UUID → "เริ่มแพ็ค" (400→450) submit fail "Invalid uuid" `[wat]`

**Found:** 2026-05-16 | **Type:** FE mock data bug | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user report 2026-05-16 — `/orders` → `order-osm-st400` → ⋯ → select "มอบหมาย WH Worker" → click "เริ่มแพ็ค" → field error "Invalid uuid"
**Plan:** `tasks/plans/BUG-FE-MOCK-UUID-01/plan.md`

#### Symptom
- คลิก "เริ่มแพ็ค" ใน OrderActionDialog (400→450 transition) → submit ถูก block, แสดง `Invalid uuid` ใต้ฟิลด์ assignedWorkerId
- เกิดเฉพาะ dev mock mode (MSW เปิด)

#### Root cause
- FE schema `apps/b1dx-oms-fulfillment/src/features/orders/schemas/order-transition.schema.ts:14,79` ใช้ `assignedWorkerId: z.string().uuid().optional()` (ถูกต้องตาม BE Guid?)
- MSW `apps/b1dx-oms-fulfillment/src/mocks/msw/db/index.ts:65` — `nextId('user')` คืน `user_${counter}` (เช่น `user_1`, `user_14`) — ไม่ใช่ UUID
- `/api/user/users?roleKey=wh-worker` คืน `items[].id = user_15..18` → `useWorkerOptions` map เป็น select value
- `OrderActionDialog.tsx:146-148` default `assignedWorkerId = user.userId` = current session user (`user_N`)
- zodResolver run on submit → fail `uuid()` validation → field error rendered ในภาษา EN ของ Zod ("Invalid uuid")

#### Fix
- เปลี่ยน mock user id generator → UUID-shaped (e.g. `00000000-0000-0000-0000-000000000NNN`) ตรงกับ format ของ roles/tenants ที่ใช้ในไฟล์เดียวกัน
- update test fixtures / assertions ที่ hardcode `user_N`
- ไม่แก้ schema (ถูกต้องตาม BE Guid)

#### Test
- Unit (vitest): regression assert `useWorkerOptions` ids ทุกตัว pass `z.string().uuid()`, mock users id pass UUID regex
- E2E + manual smoke: 400→450 submit ผ่าน (no "Invalid uuid"), 100→400 (assign worker optional) ผ่าน
- existing rbac/auth e2e suites still green

**Effort:** XS, ~1h | **Priority:** P1 (broken pack flow in dev mock — blocks demo/UAT)
**Branch:** `fix/bug-fe-mock-uuid-01`

---

### ✅ [P1] BUG-FE-EXPORT-01 — Export Order (xlsx/csv/json/pdf) ไฟล์ที่ดาวน์โหลดเปิดไม่ได้ `[wat]` — **Done — PR #195 merged 2026-05-16**

**Found:** 2026-05-16 | **Type:** FE wiring bug | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user report 2026-05-16 — Order list → context-menu → "Export Order นี้" → submit → ไฟล์ดาวน์โหลดแต่เปิดไม่ได้ (ทั้ง 4 formats)
**Plan:** `tasks/plans/BUG-FE-EXPORT-01/plan.md`

#### Symptom
- เลือก Order → ⋯ → "Export Order นี้" → BulkExportDialog → submit → ไฟล์ `.xlsx`/`.csv`/`.json`/`.pdf` ดาวน์โหลด แต่เปิดไม่ได้ (corrupt / not the expected format)
- เกิดกับทั้ง 4 formats และทั้ง single-order export + bulk export (ใช้ `BulkExportDialog` ตัวเดียวกัน)

#### Root cause
- `exportOrders()` ใน `features/orders/services/ordersApi.ts:263` เรียก raw `fetch('/api/oms/orders/export')` direct — ไม่ผ่าน gateway proxy
- API call อื่นใน app ใช้ `omsRequest()` → routes ผ่าน `oms()` helper → `/gateway/proxy/core/api/oms/...` → Next.js gateway handler → forwards ไป BE
- `/api/oms/orders/export` ไม่ตรงกับ Next.js route ใดๆ และไม่ตรงกับ MSW handler → Next.js dev / prod คืน 404 HTML → blob ถูก save เป็นไฟล์ → เปิดไม่ได้

#### Fix
- เปลี่ยน `exportOrders` ให้สร้าง URL ด้วย `oms('/api/oms/orders/export')` (= `/gateway/proxy/core/api/oms/orders/export`) — ยังคง raw fetch ไว้เพราะ response เป็น blob ไม่ใช่ JSON
- เก็บ auth headers (Bearer token + `x-tenant-id`) ไว้เหมือนเดิม
- เพิ่ม MSW handler `POST /gateway/proxy/core/api/oms/orders/export` สำหรับ dev mock mode + e2e

#### Test
- Unit (vitest): regression test assert fetch URL = `/gateway/proxy/core/api/oms/orders/export` + Accept header ต่อ 4 formats + error throw
- Manual smoke: download xlsx + csv + json + pdf เปิดได้ทั้ง 4 (Excel / VSCode / PDF reader)

**Effort:** S, ~1.5-2h | **Priority:** P1 (broken feature in production path)
**Branch:** `fix/bug-fe-export-01`

---

### [x] [P1] PERM-FE-ORDERS-01 — `/orders` FE permission gates ขาดทั้ง bulk action bar + Actions column (primary + kebab) — render ครบทุก role `[wat]` ✅ Done 2026-05-24 (PR #244)

**Found:** 2026-05-16 | **Type:** FE authorization gap (security UX) | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user request 2026-05-16 — code audit `/orders` "check permission of each role that can action bulkaction and actions in Actions column"
**Plan:** `tasks/plans/PERM-FE-ORDERS-01/plan.md`

#### Symptom
- Bulk action bar ([OrderListView.tsx:191-230](b1dx-oms-fulfillment-web/apps/b1dx-oms-fulfillment/src/features/orders/components/order-list/OrderListView.tsx#L191-L230)) render 6 buttons (Confirm/Print Ticket/Print Label/Track/Export/Cancel) เหมือนกันทุก role ไม่กรองตาม `user.permissions`
- Actions column primary button ([OrderActionButton.tsx:71-89](b1dx-oms-fulfillment-web/apps/b1dx-oms-fulfillment/src/features/orders/components/order-list/OrderActionButton.tsx#L71-L89)) — status-based เท่านั้น (ยกเว้น Match SKU เช็ค `orders.match`); transitions อื่น (Pack/Label/Confirm/Cancel/Approve) render ทุก role
- Kebab menu Actions/Tools/General sections ([OrderContextMenu.tsx:141-236](b1dx-oms-fulfillment-web/apps/b1dx-oms-fulfillment/src/features/orders/components/order-list/OrderContextMenu.tsx#L141-L236)) — list ทุก transition + Print/Video/Track/Match SKU manual + Export ไม่ filter by perm
- Match SKU manual inconsistency: primary button gate ([OrderActionButton.tsx:69](b1dx-oms-fulfillment-web/apps/b1dx-oms-fulfillment/src/features/orders/components/order-list/OrderActionButton.tsx#L69)) เช็ค `orders.match`, kebab tool item ([OrderContextMenu.tsx:197-204](b1dx-oms-fulfillment-web/apps/b1dx-oms-fulfillment/src/features/orders/components/order-list/OrderContextMenu.tsx#L197-L204)) ไม่เช็ค → user ไม่มี perm กดผ่าน kebab ได้

#### Root cause
- `useOrderListViewModel.ts:201` `computeBulkStats(items)` คำนวณ `enabled` จาก status เท่านั้น ไม่รับ `permissions`
- `OrderContextMenu` อ่าน `caps = getOrderCapabilities(status, matchStatus)` ([order-capabilities.ts:34-46](b1dx-oms-fulfillment-web/apps/b1dx-oms-fulfillment/src/features/orders/lib/order-capabilities.ts#L34-L46)) — status-based pure function, ไม่ผูก perm
- MSW handler `recomputeAvailableTransitions` ไม่ filter transitions by user perm ([order-transition.ts:134-158](b1dx-oms-fulfillment-web/apps/b1dx-oms-fulfillment/src/mocks/msw/helpers/order-transition.ts#L134-L158)) — ส่งครบทุก toState ที่ state machine อนุญาต
- FE ไม่มี helper สำหรับ map action → permission code

#### Fix
- NEW `features/orders/lib/order-action-permissions.ts` — `ACTION_PERMISSION_MAP` + helpers (`canExecuteBulkAction`, `canExecuteTransition`, `canExecuteTool`, `filterBulkActions`, `filterTransitions`) ใช้ DB-style perm keys (`orders.edit`, `orders.cancel`, `orders.export`, `orders.match`, `wh.pickpack`, `wh.returns`, `orders.view`)
- MODIFY `computeBulkStats(items, permissions)` — เพิ่ม `permitted: boolean`, `enabled = (count > 0) && permitted`
- MODIFY `useOrderListViewModel` ส่ง `user.permissions` จาก `useAuth()` เข้า `computeBulkStats`
- MODIFY `OrderListView` bulk bar hide button ถ้า `!permitted`
- MODIFY `OrderContextMenu` filter Actions/Tools/General per `canExecuteTransition/Tool` + `usePermission`; hide section header ถ้า items ว่าง
- MODIFY `OrderActionButton` เพิ่ม `canExecuteTransition(primary.toState, perms)` check
- FIX `OrderContextMenu` Match SKU manual tool → ผูก `usePermission('orders.match')` parity กับ primary button

#### Test
- Unit (vitest): 19 cases (`order-action-permissions.test.ts` 16 + `bulkActions.test.ts` 3 update)
- Component (RTL): 8 cases (4 bulk bar per role + 4 kebab per role)
- E2E (Playwright): 3 scaffold tests (`e2e/order-list/permissions.spec.ts`, test.skip)
- Manual smoke per role (6 logins): admin / ff-manager / ff-member / wh-admin / wh-worker / brand-admin

#### Out of Scope
- BE/server enforcement ของ transitions (return 403 ถ้า perm missing) → ticket แยก
- Permission naming alignment `orders:confirm/pack/admin` (colon) ↔ DB keys (dot) → ticket แยก `RBAC-ORDERS-PERM-NAMING-01`
- Order Detail (`/orders/:id`) action panel + Bulk Ops page

**Effort:** M, ~3-5h | **Priority:** P1 (authorization gap; client-side defense-in-depth)
**Branch:** `feat/perm-fe-orders-01`

---

### [x] [P1] BUG-FE-RETURN-600 — "ตีกลับ" (500→600) submit นิ่ง ไม่มี toast — schema/form ขาด returnReason → silent 422 `[wat]` ✅ Done 2026-05-17 (PR #196)

**Found:** 2026-05-16 | **Type:** FE form gap (spec mismatch) | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user report 2026-05-16 — `/orders` → `order-osm-st490` → (Courier รับแล้ว → status=500) → ⋯ → "ตีกลับ" → click confirm "ตีกลับ"
**Plan:** `tasks/plans/BUG-FE-RETURN-600/plan.md`

#### Symptom
- คลิกปุ่ม "ตีกลับ" ใน OrderActionDialog (500→600 transition) → dialog freeze, ไม่มี toast, ไม่มี field error, mutation ดูเหมือนไม่ทำงาน
- Network: MSW คืน `422 validation_failed` field=`returnReason` แต่ user ไม่เห็น feedback

#### Root cause
- Spec (`docs/specs/order-list-full-improve.md:131,408,690`): `returnReason` **required** for toState=600
- MSW backend (`apps/.../mocks/msw/helpers/order-transition.ts:215-218`) enforce ถูก → คืน 422
- FE form (`apps/.../components/order-list/OrderActionDialog.tsx:422-607` `TransitionFields`): ไม่มี block สำหรับ `toState===600` → submit empty body
- FE zod (`apps/.../schemas/order-transition.schema.ts:104-122` `createTransitionSchema`): ไม่มี `requirePresent('returnReason')` case 600 → FE pass validation ก่อนยิง
- `onError` 422 handler (`OrderActionDialog.tsx:172-178`): `form.setError('returnReason', ...)` บน field ที่ไม่ render → silent fail
- Mockup (`order-list-full-improve.html:1675-1789`) ขาด toState=600 block; spec เป็น SoT ตาม CLAUDE.md

#### Fix
- เพิ่ม block `if (toState === 600)` ใน `TransitionFields`: RHFSelect `returnReason` (required, 5 options: wrong_address/recipient_absent/refused/damaged/other) + RHFTextarea `note` (optional)
- `createTransitionSchema` switch case 600: `requirePresent('returnReason', 'Return reason is required')`
- Defensive toast: ใน `onError` 422 ถ้า field errors ไม่ map กับ form input ที่ render → `toast.error(...)`

#### Test
- Unit (vitest): regression `OrderActionDialog` toState=600 empty submit → RHF error visible, mutation NOT called; toState=600 + select → mutation called with `{toState:600, returnReason:'damaged'}`; schema `createTransitionSchema(500, 600)` empty → issue path=`returnReason`; 422 unmapped field → toast.error called
- E2E + manual: `/orders` → st490 → "Courier รับแล้ว" → 500 → ⋯ → "ตีกลับ" → select → confirm → success screen + row → 900

**Effort:** XS, ~45min | **Priority:** P1 (broken transition flow visible to user; silent failure UX bug)
**Branch:** `fix/bug-fe-return-600`

---

### [x] [P2] DEPLOY-DASHBOARD — Public mirror dashboard via GitHub Pages `[nuchit]` ✅ Done 2026-05-16 (PR #17 + #19 + #20)

**Found:** 2026-05-16 | **Type:** Chore (infra/devops) | **Repo:** `b1dx-fulfillment-workspace`
**Plan:** `tasks/plans/DEPLOY-DASHBOARD/plan.md`
**Live URL:** <https://b1dxdev.github.io/b1dx-fulfillment-dashboard/>

#### Delivered

- `scripts/build-site.mjs` → produces `_site/` + sanitizer for secret patterns
- `.github/workflows/deploy-dashboard.yml` → pushes HTML-only to `B1DXDev/b1dx-fulfillment-dashboard` on push to main
- Landing page auto-redirects to executive-summary.html
- 16 files deployed: executive-summary, progress, backlog, roadmap, standup/*, reports/*

#### Test cases — all passed

- [x] `node scripts/build-site.mjs` local — 16 files, no secrets
- [x] Push to main → workflow success → URL reachable (HTTP 200)
- [x] First-deploy edge case: empty mirror repo handled via fallback `git init -b main`

#### Notes

- 3 fix PRs needed: #17 (initial), #19 (YAML multi-line commit bug), #20 (empty-repo checkout)
- Final blocker: fine-grained PAT was pending org approval → switched to Classic PAT + SSO authorize
### [x] [P1] OAD-FE-09a — Action column ขาดปุ่ม ⋯ + ContextMenu ไม่ได้ wire `[wat]` ✅ Done 2026-05-15 (PR #180)

**Found:** 2026-05-14 | **Type:** FE wiring gap (improvement) | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user audit 2026-05-14 — เทียบ column action ของ app vs `docs/mockups/order-list-full-improve.html` พบ 3 gaps
**Plan:** `tasks/plans/OAD-FE-09a/plan.md`
**Parent:** OAD-FE-09 (Action+SLA+Flags+Courier) — แยก Action col ออกมา เพราะ FE-07/08 ยังไม่เสร็จ

#### Symptom
- `ActionsCell` มีปุ่ม primary action ปุ่มเดียว — mockup กำหนด `[primary] [⋯]` คู่กัน
- `OrderContextMenu.tsx` เขียนเสร็จใน OAD-FE-05 (merged PR #144) แต่ไม่ถูก import ที่ไหนเลยนอกจาก test → user เปิด context menu ไม่ได้
- Context menu ขาด 2 items: `📄 ดู Invoice` (status=700) และ `🔍 Match SKU Manual` (status=100 unmatched/partial)

#### Root cause
- OAD-FE-09 (Action+SLA+Flags+Courier wiring) ยังไม่เริ่ม — dep รอ FE-07/08
- FE-05 implement `OrderContextMenu` standalone แต่ไม่ได้ wire เพราะแยก task

#### Fix
- เพิ่ม ⋯ button + portal-mount `OrderContextMenu` ใน `ActionsCell`
- เพิ่ม `canViewInvoice` + `canMatchSkuManual` capabilities + 2 menu items
- เพิ่ม i18n keys `orders.action.view_invoice` + `orders.action.match_sku_manual`

#### Test
- Unit: 14 cases (ActionsCell + OrderContextMenu) ครอบคลุม render / portal / dismissal / capability gating / dialog open
- E2E: 5 cases (open menu, invoice item, manual match → SkuMatchDialog, copy id, click outside)

**Effort:** M, ~2-3h | **Priority:** P1 (visual gap จาก mockup, blocks FE-09 wiring)
**Branch:** `feat/oad-fe-09a-action-context-menu`

---

### [x] [P2] PIM-I18N-RETROFIT — PIM module FE-02..08 use literal strings; retrofit `useTranslation` + `pim.*` locale keys `[nuchit]` ✅ Done 2026-05-15

> Plan: [`tasks/plans/PIM-I18N-RETROFIT/plan.md`](plans/PIM-I18N-RETROFIT/plan.md) (Status: Done)
> FE PR: [b1dx-oms-fulfillment-web#186](https://github.com/B1DXDev/b1dx-oms-fulfillment-web/pull/186) + FE-04 wave PR [#187](https://github.com/B1DXDev/b1dx-oms-fulfillment-web/pull/187) · Workspace PR: [#13](https://github.com/B1DXDev/b1dx-fulfillment-workspace/pull/13)
> Effort: L · ~5-6h (actual) — across 2 PRs (#186 main retrofit + #187 follow-up for FE-04)

**Found:** 2026-05-15 | **Type:** FE refactor (i18n debt) | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** PIM-FE-08 review 2026-05-15 — plan claimed i18n in scope; impl used literal strings consistent with FE-02/03/06 already merged

#### Symptom
- All 4 PIM tabs (Variants/Pricing/Channels/Categories) + Categories CRUD view use English literal strings
- No `useTranslation` hook used anywhere under `src/features/pim/`
- `en.json` / `th.json` have **0** `pim.categories.*` / `pim.pricing.*` / `pim.channels.*` keys
- Original plans (FE-02..08) all listed i18n as in-scope but implementations consistently skipped — inconsistency between plan and reality

#### Fix
- Audit all literal strings under `src/features/pim/{components,views}/`
- Introduce `pim.*` namespace in `src/lib/i18n/locales/{en,th}.json`
- Wrap user-visible strings with `useTranslation` or equivalent project hook
- Update plan templates so future PIM tickets either declare i18n out-of-scope OR enforce it pre-merge

#### Test
- Unit: `tsc` clean after replacement; existing snapshot/render tests still pass
- E2E: TH locale switch shows Thai labels in PIM pages

**Effort:** L, ~4-6h (4 tabs + Categories CRUD + ~80-100 keys × 2 locales)
**Priority:** P2 (consistency + future-proofing for non-EN customers)
**Branch:** `refactor/pim-i18n-retrofit`

---

### 🆕 [P1] BUG-PIM-BE-01B-MERGE-LOSS — BE main broken: merge commit 5bf5c9a deleted BE-01b's DbSet wirings ✅ **In Review (PR #80)**

**Found:** 2026-05-17 | **Type:** BE regression from merge conflict | **Repo:** `b1dx-oms-fulfillment-api`
**Source:** PIM-BE-01c-1 build attempt — `dotnet build` failed with ~10 CS1061 errors

#### Symptom
- `_db.PimBrands`, `_db.PimCategories`, `_db.PimAttributes`, `_db.PimCategoryAttributeBindings` referenced in services but DbSets gone from AppDbContext
- Merge commit `5bf5c9a` shows `488 deletions(-)` and 0 insertions across 4 files — classic "keep one side" conflict resolution discarding the other side entirely

#### Fix
- PR [#80](https://github.com/B1DXDev/b1dx-oms-fulfillment-api/pull/80): `git checkout 070dc84 -- <4 files>` restores deleted content byte-for-byte
- Verified: `dotnet build` 0 errors, `dotnet test` unit 526/526 pass

#### Blocks
- **PIM-BE-01c** (and its split BE-01c-1/2/3) — extends the same DbContext, can't proceed until #80 merges

**Effort:** XS, 15min | **Priority:** P1 (blocks all PIM BE work) | **Branch:** `fix/pim-be-01b-restore-dbcontext-wirings`

---

### 🆕 [P2] TEST-PIM-DRIFT-AUDIT-PROP — AuditTab not wired with productId in ProductDetailView ✅ **In Review (PR #198, stacked on #197)**

**Found:** 2026-05-17 | **Type:** FE impl bug (regression discovered by E2E) | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** PIM-E2E-01 implementation 2026-05-17 — 3 audit tests fixmed (E2E-AUDIT-01/02/03)
**Fix:** PR [#198](https://github.com/B1DXDev/b1dx-oms-fulfillment-web/pull/198) — stacked on #197, commit `112f70b`. Verified 3/3 audit tests pass.

#### Symptom
- `apps/b1dx-oms-fulfillment/src/features/pim/components/ProductDetailView.tsx:137` renders `<AuditTab />` with NO `productId` prop
- `AuditTab` early-returns its own empty state when `productId` is falsy → `AuditList` + all `audit-*` testids never mount on product-detail pages
- Audit tab in product detail = permanently empty regardless of mock data

#### Fix
- Pass `productId={product.id}` (or equivalent) to `<AuditTab />` in ProductDetailView:137 — single-line fix
- Un-fixme E2E-AUDIT-01/02/03 in `e2e/pim/audit.spec.ts`
- Verify 3 audit tests pass: `pnpm playwright test e2e/pim/audit.spec.ts`

#### Test
- E2E-AUDIT-01 [happy]: audit tab → entries list renders
- E2E-AUDIT-02 [happy]: open diff view → before/after rows visible
- E2E-AUDIT-03 [edge]: revert button → confirm dialog opens

**Effort:** XS, ~15min (one prop fix + un-fixme)
**Priority:** P2 (audit feature non-functional on product detail; tests already written, just fixmed)
**Branch:** `fix/audit-tab-product-id-prop`

---

### 🆕 [P3] TEST-PIM-FE-08-MISSING — Missing component tests for AttributeBinder + ChannelMappingPanel + TreeNode ✅ **In Review (PR #199)**

**Found:** 2026-05-15 | **Type:** FE test coverage gap | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** PIM-FE-08 self-review 2026-05-15 — 7 components shipped, 3 component test files (Tree, Form, Delete)

#### Symptom
- `CategoryAttributeBinder` has local state (`pickerId`), side-effect callbacks — no test
- `CategoryChannelMappingPanel` has per-channel draft state map, save clears draft — no test
- `CategoryTreeNode` tested indirectly via `CategoryTree` (acceptable, but actions like add/edit/delete hover-button visibility not asserted directly)

#### Fix
- Add `CategoryAttributeBinder.test.tsx` — picker filters bound attrs, bind clears picker, required toggle fires, unbind fires
- Add `CategoryChannelMappingPanel.test.tsx` — empty row saves + clears draft, existing row removes, save disabled until id present
- Optionally `CategoryTreeNode.test.tsx` — hover actions render, status badge for inactive

#### Test
- vitest count should increase from 40 (FE-08 contributes) to ~52 (+~12 new)
- All pass with `pnpm vitest run src/features/pim`

**Effort:** S, ~1-1.5h
**Priority:** P3 (hook coverage already strong; components are mostly presentational)
**Branch:** `test/pim-fe-08-component-coverage`

---

### [x] [P3] FIX-PIM-FE-03-FOOTER-BUTTONS — Pricing tab "Save pricing" + "Price history" footer buttons ✅ **Resolved: not a bug (already implemented)** — 2026-05-17

**Found:** 2026-05-15 | **Type:** FE visual gap (cosmetic) | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** PIM-FE-03 visual fidelity re-check 2026-05-15 — screenshot `tasks/screenshots/pim-wave1-verify/03-product-detail-pricing.png` vs mockup `docs/mockups/pim/beone_pim_crud_mock.html` line 1959-1961

#### Outcome (2026-05-17)
Footer buttons จริงๆ **มีตั้งแต่ FE-03 merge `7f9712a`** (`PricingTab.tsx:74-98`):
- `data-testid="pim-pricing-save"` — primary button with 💾 icon, disabled with title "Coming in PIM-BE-01c integration" (correct — wire to BE later)
- `data-testid="pim-pricing-history-open"` — outline button with History icon + entry count, opens `PriceHistoryDrawer`

E2E test `E2E-PRICING-04` exercises the history-open button and passes on chromium. The 2026-05-15 visual check screenshot likely was captured before the tab scrolled to the footer region (or referenced an earlier component state). No code change needed; closing as resolved.

#### Original Symptom (now obsolete)
- ~~Mockup ระบุ 2 buttons ที่ footer ของ Pricing tab: `[💾 Save pricing] [Price history]`~~
- ~~FE จริงไม่มี buttons เหล่านี้ — PriceHistoryDrawer มีแล้วแต่ไม่มี trigger button~~

**Effort:** S, ~30min | **Priority:** P3 (cosmetic — drawer accessible via other paths, no functional impact)
**Branch:** (none — no code change needed)

---

### [x] [P1] BUG-PRINT-LABEL-PDFLIB — Missing `pdf-lib` + `@pdf-lib/fontkit` deps break FE dev server `[srattha]` ✅ Done 2026-05-13

**Found:** 2026-05-13 | **Type:** FE dep/build bug (dev mode blocker) | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** PIM-FE-03 visual fidelity check 2026-05-13 — `pnpm dev` boot fails
**Resolution:** Fixed by srattha in commit `3509210` (`fix(print-label): wire bulk-print to mock PDF + render mockup-faithful labels`) — added `pdf-lib@^1.17.1` + `@pdf-lib/fontkit@^1.1.1` to `apps/b1dx-oms-fulfillment/package.json`. Unblocks FE dev server and PIM-FE-03 visual fidelity verification.

#### Symptom
- `pnpm dev` build fails with `Module not found: Can't resolve 'pdf-lib'` at `apps/b1dx-oms-fulfillment/src/mocks/msw/lib/buildLabelPdf.ts:1`
- Import trace: `MockProvider → msw/index → handlers/index → orders → buildLabelPdf`
- Blocks **all** `localhost:3000` development including PIM-FE-03 visual fidelity verification

#### Root cause (suspected)
- PRINT-LABEL-REAL-FE work landed `buildLabelPdf.ts` referencing `pdf-lib` + `@pdf-lib/fontkit` but `pnpm add pdf-lib @pdf-lib/fontkit` step was not committed to `apps/b1dx-oms-fulfillment/package.json` / lockfile

#### Fix
- `pnpm add pdf-lib @pdf-lib/fontkit` in `apps/b1dx-oms-fulfillment`
- Commit `package.json` + `pnpm-lock.yaml`
- Owner: srattha (PRINT-LABEL feature owner)

#### Test
- `pnpm dev` boots without module-not-found error
- Can navigate to `http://localhost:3000/pim/products/1` and load PIM module

**Effort:** XS, ~10min | **Priority:** P1 (high — blocks all dev server work and PIM-FE-03 visual verification)
**Discovered:** 2026-05-13 during PIM-FE-03 dev server visual check

---

### ✅ [P1] FIX-LABEL-490 — Bulk Print Label status 490 silent skip + missing skippedIds warning `[srattha]` `Done 2026-05-14`

**Found:** 2026-05-14 | **Type:** BE+FE quality bug (post PRINT-LABEL-REAL release) | **Repo:** `b1dx-oms-fulfillment-api` + `b1dx-oms-fulfillment-web`
**Source:** user QA 2026-05-14 — orderlist เลือก `ORD-OSM-ST480` + `ORD-OSM-ST490` → bulk print label → PDF ออกแค่ 1 หน้า (ST480 only), ST490 ถูก skip เงียบ ไม่มี warning
**Depends on:** _(none — fix on top of PRINT-LABEL-REAL-BE/FE merged 2026-05-13)_

#### Symptom
- เลือกหลาย orders ที่มี status รวม 490 + status อื่น → PDF ขาด label ของ order ที่ 490
- ไม่มี toast/warning แจ้ง user → user คิดว่าระบบพิมพ์ครบแต่จริงๆ ขาดไปเงียบๆ

#### Root cause
1. **BE** [`PrintOrdersPdfQueryHandler.cs:20-25`](../../be-repo/src/B1dx.OmsFulfillment.Infrastructure/Services/PrintOrdersPdfQueryHandler.cs) — `PrintAllowedStates = {400, 450, 480}` ไม่รวม `490` (LabelPrinted/ReadyToShip)
2. **FE** [`useBulkPrintLabel.ts:25-29`](../../web-repo/apps/b1dx-oms-fulfillment/src/features/orders/hooks/useBulkPrintLabel.ts) — `fetch(url).blob()` ตรงๆ ไม่ parse `skippedIds` จาก response → ไม่แจ้ง user
3. **FE** [`order-capabilities.ts:12`](../../web-repo/apps/b1dx-oms-fulfillment/src/features/orders/lib/order-capabilities.ts) — `PRINT_LABEL_STATUSES = {480}` กันการเลือก 490 ฝั่ง FE ไม่ครบ flow

#### Fix
1. **BE:** เพิ่ม `FulfillmentSubStatus.LabelPrinted` ใน `PrintAllowedStates` (use case: reprint label ที่ถูกพิมพ์ไปแล้ว — read-only ปลอดภัย เพราะ shipment + tracking มีอยู่แล้ว)
2. **FE:** เพิ่ม `490` ใน `PRINT_LABEL_STATUSES`
3. **FE:** แก้ `useBulkPrintLabel` ให้อ่าน `skippedIds` (response header `X-Skipped-Ids` หรือ response JSON) → `toast.warning` พร้อมจำนวน orders ที่ skip

#### Test
- Unit: BE — eligible state matrix ครอบคลุม 490, FE — skippedIds parsing + toast trigger
- E2E: เลือก 2 orders ต่าง status (480+490) → ทั้งสองพิมพ์ได้ + ไม่มี skip
- E2E: เลือก orders ที่มี status 5xx รวม → skip + toast แสดง

**Files:** `PrintOrdersPdfQueryHandler.cs`, `OrderEndpoints.cs` (อาจต้อง expose skippedIds ใน response), `order-capabilities.ts`, `useBulkPrintLabel.ts` + tests
**Effort:** S, ~1.5h | **Priority:** P1 (data หาย — user ไม่รู้ว่า label หายไปจาก batch)
**Branch:** `fix/label-status-490`

---

### ✅ [P2] FIX-COURIER-ACTIONCOL — Courier Settings table Action column ขาด View+Edit icons `[srattha]` `Done 2026-05-15`

**Found:** 2026-05-14 | **Type:** FE visual fidelity bug | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user review 2026-05-14 — เปิด `http://localhost:3000/settings/couriers` Action column มีแค่ปุ่ม Delete ตัวเดียว ไม่ตรง mockup
**Mockup:** [`docs/mockups/courier-settings.html`](../../docs/mockups/courier-settings.html) lines 1228–1240 (`.row-act` 3 ปุ่ม view+edit+delete)
**Resolution:** PR #181 merged 2026-05-15 (`e843c9a`) — CourierTable.tsx Action column 1 ปุ่ม → 3 ปุ่ม (view+edit+delete); view/edit disabled + "Coming soon" รอ CRSET-02; hover colors per mockup; 28×28; i18n + tests 10/10

#### Symptom
- หน้า Couriers ตาราง column Action แสดงแค่ icon `Trash2` ตัวเดียว
- Mockup กำหนด 3 ปุ่ม: 👁 View (eye) → ✏️ Edit (pencil) → 🗑 Delete (trash) เรียงแนวนอน + hover สีต่างกัน (teal/indigo/red)

#### Root cause
- [`CourierTable.tsx:231-243`](../../web-repo/apps/b1dx-oms-fulfillment/src/features/couriers/components/CourierTable.tsx#L231-L243) — render เฉพาะปุ่ม delete (PermissionGate `courier.delete`); ไม่มี view/edit เพราะ drawer ของ existing courier วางไว้ใน CRSET-02

#### Fix
- เพิ่ม 2 ปุ่ม `<button>` (View, Edit) — `disabled` + tooltip "Coming soon" (CRSET-02 จะ wire จริง)
- Style ตาม `.ra-btn` mockup: 28×28 rounded-md hover เฉพาะสี (view=teal, edit=indigo, del=red)
- ปรับ delete button ให้ size + hover ตรง `.ra-del` mockup
- เพิ่ม i18n keys: `couriers.action.view`, `couriers.action.edit`, `couriers.action.coming_soon`

#### Test
- Unit: ทุก row render 3 ปุ่ม; view+edit `disabled`; delete clickable + `stopPropagation`; PermissionGate ยังครอบเฉพาะ delete (regression)
- Visual fidelity: screenshot vs mockup — 3 ปุ่ม + 28×28 + hover สี

**Files:** `apps/b1dx-oms-fulfillment/src/features/couriers/components/CourierTable.tsx` + `CourierTable.test.tsx` + `src/lib/i18n/locales/{en,th}.json`
**Effort:** S, ~1.5h | **Priority:** P2 (visual fidelity — ไม่บล็อก workflow แต่ผิดจาก mockup)
**Branch:** `fix/courier-action-col-fidelity` | **Plan:** [`tasks/plans/FIX-COURIER-ACTIONCOL/plan.md`](plans/FIX-COURIER-ACTIONCOL/plan.md)

---

### ✅ [P1] FIX-LABEL-ADDR-WRAP — 10x15 recipient address ถูกตัด "..." `[srattha]` Done 2026-05-14

**Found:** 2026-05-14 | **Type:** BE PDF render bug | **Repo:** `b1dx-oms-fulfillment-api`
**Source:** user QA 2026-05-14 — พิมพ์ label 10x15 → preview เห็น `คลองเตย คลองเตย กรุงเทพมหานคร 1...` (truncated)

#### Symptom
- ที่อยู่ผู้รับยาวๆ ถูกตัดท้ายด้วย ellipsis แทนการ wrap หลายบรรทัด
- กระทบการจัดส่งจริง — courier อาจส่งผิดที่ถ้าอ่าน label จากที่ตัดแล้ว

#### Root cause (สมมติฐาน)
- [`LabelDocument.cs:62`](../../be-repo/src/B1dx.OmsFulfillment.Infrastructure/Services/Pdf/Templates/LabelDocument.cs) — `Text($"{order.RecipientProvince ?? "-"}  {order.RecipientZip ?? "-"}").FontSize(8)` รวมบรรทัดเดียว เกิน width 10cm → QuestPDF default ตัด ellipsis แทน wrap
- Width container = 283.46pt - margin 16 - padding 8 = ~260pt = ~9.2cm
- Province ภาษาไทยยาวๆ + 2 spaces + zip อาจเกิน 9.2cm ที่ fontSize 8

#### Fix
- แยก `RecipientProvince` กับ `RecipientZip` คนละบรรทัด ลด width pressure
- หรือ enable explicit wrap ใน QuestPDF text style
- หรือลด fontSize ของบรรทัด address (fallback)

#### Test
- Unit test: render label ด้วย address ยาวมาก (`คลองเตย คลองเตย กรุงเทพมหานคร`) → assert ไม่มี ellipsis ใน PDF text content
- Visual: เปรียบเทียบ 10x10 / 10x15 / Thermal100mm ด้วย long address — แสดงครบทุกขนาด

**Files:** `LabelDocument.cs` + new unit test in `PrintOrdersPdfQueryHandler_Label_Tests.cs` หรือ test ใหม่
**Effort:** XS, ~30min | **Priority:** P1 (กระทบการจัดส่งจริง — address ตัดทำให้ส่งผิดที่)
**Branch:** `fix/label-addr-wrap`

---

### ✅ [P2] FIX-LABEL-THERMAL — Thermal100mm ใช้ขนาด+layout เดียวกับ 10x15 `[srattha]` `Done 2026-05-14` (no-op)

> Decision เปลี่ยน 2026-05-14: user สั่งให้ใช้ขนาด+layout เดียวกับ 10x15 ไปเลย (ไม่แยก thermal-optimized layout) → code ปัจจุบันเข้าเงื่อนไขนี้อยู่แล้ว ([LabelDocument.cs:17-21](../../be-repo/src/B1dx.OmsFulfillment.Infrastructure/Services/Pdf/Templates/LabelDocument.cs#L17-L21) — `Thermal100mm` ใช้ PageSize เดียวกับ `Label10x15` และ `Compose()` ไม่มี branching ตาม format) → ปิด task ไม่ต้องแก้โค้ด. Plan: `tasks/plans/FIX-LABEL-THERMAL/plan.md`.

<details><summary>Original scope (ก่อน decision change)</summary>

**Found:** 2026-05-14 | **Type:** BE feature gap (post PRINT-LABEL-REAL) | **Repo:** `b1dx-oms-fulfillment-api`
**Source:** user QA 2026-05-14 — เลือก Thermal 100mm vs 10x15 cm → PDF byte-identical (page size + layout เหมือนกันเปี๊ยบ)

#### Symptom
- User เลือก Thermal100mm = ไม่ได้ประโยชน์อะไรนอกจากชื่อ format
- Output ที่ได้ไม่ optimize สำหรับ thermal printer (มีสี, font เล็ก, contrast ต่ำ)

#### Root cause
- [`LabelDocument.cs:18-19`](../../be-repo/src/B1dx.OmsFulfillment.Infrastructure/Services/Pdf/Templates/LabelDocument.cs):
  ```csharp
  LabelFormat.Label10x15   => new PageSize(283.46f, 425.20f),
  LabelFormat.Thermal100mm => new PageSize(283.46f, 425.20f),  // ← เท่ากับ 10x15
  ```
- Comment ใน code: "spec has no fixed roll length" — ใช้ค่าเดียวกับ 10x15 เป็น fallback
- ใช้ template/layout เดียวกันทั้งหมด

#### Fix (Decision: ขนาด 100×150mm + แยก layout)
- คงขนาด 100×150mm (= 10×15) เพราะเป็นมาตรฐาน shipping label ที่ courier ใช้
- **แยก thermal-optimized layout:**
  - ตัดสีออกหมด → ขาวดำเท่านั้น (thermal พิมพ์สีไม่ได้)
  - Font ใหญ่ขึ้น (thermal 203 DPI vs laser 600 DPI)
  - Barcode + tracking number ใหญ่ขึ้น (scanner-friendly)
  - Border หนา + contrast สูง (ทน thermal degradation)

#### Test
- Unit: render Thermal100mm → assert PDF bytes ต่างจาก 10x15
- Unit: render Thermal100mm → assert ไม่มี colored elements
- Visual review: เปรียบเทียบ Thermal100mm vs 10x15 vs mockup ใหม่ (ขอ design ช่วย review)

**Files:** `LabelDocument.cs` (อาจแยกเป็น `ThermalLabelDocument.cs`), `QuestPdfLabelRenderer.cs` (route format → renderer), tests
**Effort:** S, ~1.5h | **Priority:** P2 (ใช้งานได้ แต่ output ไม่ optimize สำหรับ thermal)
**Branch:** `fix/label-thermal-layout`

</details>

---

### ✅ [P2] FIX-LABEL-PRINTFLOW — Print dialog default A4+Fit ทำให้ทุกขนาดเท่ากันบนกระดาษ `[srattha]` `Done 2026-05-14`

**Found:** 2026-05-14 | **Type:** FE UX bug | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user QA 2026-05-14 — กดพิมพ์ label/ticket → browser print dialog ต้องเลือก A4 + Fit to paper เองทุกครั้ง → confused ว่า feature เสีย
**Plan:** `tasks/plans/FIX-LABEL-PRINTFLOW/plan.md` · **Branch:** `fix/label-printflow-a4-fit` · **PR merged manually 2026-05-14**

#### Resolution (Decision change — PDF-side fix แทน UI-side)
- `buildLabelPdf.ts` + `buildTicketPdf.ts` — render native-size label/ticket ใน scratch PDFDocument → `embedPages` → drawPage scaled-fit + centered บน A4 (595×842pt) ทุก format
- ผล: browser print dialog default = A4 อยู่แล้ว, ไม่ต้องเลือก Scale=Fit (PDF page = paper size อยู่แล้ว) → ทุก format (10x10/10x15/thermal_100mm/a4_2up/a5/thermal_80mm/pdf) ออกขนาดเท่ากันบนกระดาษ
- **Tests:** unit 16/16 ✅ (label 6 + ticket 7 + helpers) · E2E scaffold `e2e/print-flow/print-a4-fit.spec.ts` (test.skip)
- **Risk noted:** thermal printer driver users — ต้องพึ่ง shrink-to-fit (deferred — future task ถ้ามี complaint)

---

### 🆕 [P1] FIX-MSW-RACE — MSW worker start race → POST/PATCH หลุดไป BE → 404 `[srattha]`

**Found:** 2026-05-13 | **Type:** FE infra bug (dev mode) | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user 2026-05-13 — bulk print ticket (status 400) ได้ 404 จาก `POST /api/oms/orders/order-osm-st400/print` แม้ MSW handler มี
**Plan:** `tasks/plans/FIX-MSW-RACE/plan.md`

#### Root cause
- `worker.start()` resolve เมื่อ SW register แต่ยังไม่ active/controlling page → MockProvider setReady → children render → first POST หลุดไป BE จริง
- BE มี route constraint `{id:guid}` ที่ทุก order endpoint → mock id (`order-osm-st400`) ไม่ match GUID → 404 (silent — debug ยาก)

#### Fix
- เพิ่ม `await navigator.serviceWorker.ready` + รอ `controllerchange` ถ้า controller===null ใน `src/mocks/msw/index.ts`

**Files:** `apps/b1dx-oms-fulfillment/src/mocks/msw/index.ts` + new test
**Effort:** XS, ~30min | **Priority:** P1 (blocker — กระทบ POST/PATCH/DELETE ทั้งหมดใน dev mode)

---

### ✅ [P1] PRINT-TICKET-REAL — Bulk print ticket ไม่ออกกระดาษจริง (queue เปล่า) `[srattha]` — Done 2026-05-14

**Found:** 2026-05-13 | **Type:** FE behavior gap | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user 2026-05-13 — กด "สั่งพิมพ์ A4 (2 up)" แล้วไม่มีอะไรออก (เพราะปัจจุบันแค่ POST /print = queue เปล่า)
**Mockup:** `docs/mockups/order-list-full-improve.html` line 1244-1266
**Spec:** `docs/specs/order-bulk-actions.md` § WF-02
**Plan:** `tasks/plans/PRINT-TICKET-REAL/plan.md`
**Depends on:** FIX-MSW-RACE (ทดสอบไม่ได้ถ้า MSW ยัง race)

#### Issue
- `a4_2up` / `a5` / `thermal_80mm` → ยิง `POST /orders/{id}/print` = สร้าง PrintJob queue เท่านั้น ไม่มีกระดาษ
- `pdf` format → ใช้ `/orders/print-pdf` แล้ว ดาวน์โหลด PDF ได้ (แต่ user ต้องเปิดเอง + กดพิมพ์)

#### Fix
- BE `GET /api/oms/orders/print-pdf?format=...` รองรับทุก format ผ่าน QuestPDF อยู่แล้ว ([OrderEndpoints.cs:395](../../be-repo/src/B1dx.OmsFulfillment.Api/Endpoints/OrderEndpoints.cs))
- เปลี่ยน `useBulkPrintTicket` ให้ทุก format ไปยัง `/print-pdf` → open blob URL ใน new window → `window.print()` อัตโนมัติ
- `pdf` format คงเดิม (save as file)
- handle popup blocker ด้วย toast

**Files:** `useBulkPrintTicket.ts` + test, MSW handler `orders.ts` verify, i18n keys `print.popup_blocked`
**Effort:** S, ~1.5-2h | **Priority:** P1 (feature ใช้ไม่ได้จริง)

---

### ✅ [P1] PRINT-LABEL-REAL-BE — Bulk print Label PDF renderer (BE) `[srattha]`

**Found:** 2026-05-13 | **Type:** BE feature gap | **Repo:** `b1dx-oms-fulfillment-api`
**Source:** user 2026-05-13 — status 480 กด "พิมพ์ Label" → แค่ toast (ไม่มี BE renderer สำหรับ label PDF)
**Mockup:** `docs/mockups/print_label_480.html`
**Spec:** `docs/specs/order-bulk-actions.md` § WF-03
**Plan:** `tasks/plans/PRINT-LABEL-REAL-BE/plan.md`
**Depends on:** _(none)_

#### Issue
- `/api/oms/orders/print-pdf` ปัจจุบันรับเฉพาะ `jobType=ticket` + ticket formats (`a4_2up`/`a5`/`thermal_80mm`) ([OrderEndpoints.cs:435](../../be-repo/src/B1dx.OmsFulfillment.Api/Endpoints/OrderEndpoints.cs))
- ไม่มี `LabelPdfRenderer` — label formats (`10x10`/`10x15`/`thermal_100mm`) ยังไม่ implement

#### Fix
- ขยาย `/print-pdf` รับ `jobType=label` + label formats
- เพิ่ม `LabelPdfRenderer` (QuestPDF) — layout ตาม mockup
- `PrintOrdersPdfQueryHandler` branch ticket vs label
- จัดกลุ่ม order ตาม courierCode ใน PDF เดียว

**Files:** `OrderEndpoints.cs`, `PrintOrdersPdfQueryHandler.cs`, `LabelPdfRenderer.cs` (NEW), test
**Effort:** M, ~4-6h | **Priority:** P1 (feature ใช้ไม่ได้จริง)

---

### ✅ [P1] PRINT-LABEL-REAL-FE — Bulk print Label ออกกระดาษจริง (FE) `[srattha]` — Done 2026-05-13

**Found:** 2026-05-13 | **Type:** FE behavior gap | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user 2026-05-13 — กด "พิมพ์ Label" ใน bulk action (status=480) ได้แค่ toast ไม่มีกระดาษ
**Mockup:** `docs/mockups/print_label_480.html`
**Spec:** `docs/specs/order-bulk-actions.md` § WF-03
**Plan:** `tasks/plans/PRINT-LABEL-REAL-FE/plan.md`
**Depends on:** PRINT-TICKET-REAL (reuse helper) · PRINT-LABEL-REAL-BE (real env) · FIX-MSW-RACE (dev)

#### Issue
- `useBulkPrintLabel` loop ยิง `POST /orders/{id}/print` (queue เปล่า) → toast `"Label N ใบ กำลังพิมพ์"` แต่ไม่มีกระดาษออก ([useBulkPrintLabel.ts:18](../../web-repo/apps/b1dx-oms-fulfillment/src/features/orders/hooks/useBulkPrintLabel.ts))
- MSW handler `/print-pdf` ปัจจุบัน hardcode 400 `jobType=label` ([orders.ts:769](../../web-repo/apps/b1dx-oms-fulfillment/src/mocks/msw/handlers/orders.ts))

#### Fix
- rewrite `useBulkPrintLabel` → `GET /print-pdf?jobType=label&format=...&ids=...` → blob → `window.open` → auto `window.print()`
- reuse `openAndPrint` helper จาก PRINT-TICKET-REAL
- MSW handler รับ `jobType=label` + ทุก label format → fake PDF
- popup blocker handle ด้วย toast `print.popup_blocked`

**Files:** `useBulkPrintLabel.ts` + test, MSW `orders.ts`, `lib/printPdf.ts` (reuse)
**Effort:** S, ~1.5-2h | **Priority:** P1 (feature ใช้ไม่ได้จริง)

---

### ✅ [P2] IMPROVE-MATCH-FIDELITY — SkuMatchDialog visual ไม่ตรง mockup `[srattha]` — Done 2026-05-13

**Found:** 2026-05-13 | **Type:** FE visual fidelity | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user review 2026-05-13 — เปิด SkuMatchDialog เทียบ mockup `sku_match_modal_improved.html` แล้วพบ gap หลายจุด
**Mockup:** `docs/mockups/sku_match_modal_improved.html`
**Spec:** `docs/specs/order-sku-match.md` § 6 Manual-match Flow
**Plan:** `tasks/plans/IMPROVE-MATCH-FIDELITY/plan.md`

#### Issues
1. **Header subtitle ขาด** — ปัจจุบันรวม shop + StatusPill + progress ใน row เดียว ไม่มี subtitle row `{shop} · {channel} · {N} รายการรอจับคู่`
2. **Section header `รายการจาก Platform` ขาด** + progress `จับคู่แล้ว N/M` ทางขวา
3. **Item row ไม่เปลี่ยน background/border ตาม state** — ปัจจุบันใช้ border-b ธรรมดา ไม่ตรง mockup ที่เปลี่ยน amber/green/red
4. **Badge ไม่ครบ** — มีแค่ `จับคู่แล้ว` (matched) — ขาด `รอจับคู่` (amber) และ `ไม่พบ SKU` (red)
5. **Validation/Confirm bar ขาด** — mockup มี amber "ยังจับคู่ไม่ครบ..." / green "จับคู่ครบ N รายการแล้ว — พร้อมยืนยัน"

**Deferred (out of scope ของ task นี้):**
- Stock count ใน dropdown — ต้อง BE Inventory module ก่อน (Product type ยังไม่มี `stockOnHand`)
- AI Suggest per-row + "แนะนำทั้งหมด" footer — spec gap, ต้องคุย product + BE `/suggest` endpoint
- Dropdown style เปลี่ยนเป็น `<select>` — คง combobox typeahead

**Files (modify only):**
- `apps/b1dx-oms-fulfillment/src/features/orders/components/order-list/SkuMatchDialog.tsx`
- `apps/b1dx-oms-fulfillment/src/features/orders/components/order-list/SkuMatchDialogItemRow.tsx`
- `apps/b1dx-oms-fulfillment/src/lib/i18n/locales/{th,en}/orders.json`
- `__tests__/SkuMatchDialog.test.tsx` + `__tests__/SkuMatchDialogItemRow.test.tsx`

**Test Cases:**
- [ ] (regression) typeahead search ยังทำงาน + clear (`✕`) ยังเอา selection ออก
- [ ] section header `รายการจาก Platform` + progress `จับคู่แล้ว N/M` render
- [ ] badge `รอจับคู่` (amber) เมื่อ unmatched + ยังไม่ select
- [ ] badge + row color เปลี่ยนเป็น green เมื่อ select product
- [ ] validation bar amber → green เมื่อ matched ครบทุก row
- [ ] header subtitle render `{shop} · {channel} · {N} รายการรอจับคู่`
- [ ] E2E: เปิด dialog → เลือก SKU ครบ → validation green + ปุ่ม `ยืนยัน Match` enable

**Effort:** S, ~2-2.5h | **Priority:** P2 (visual fidelity, not functional)

---

### 🆕 [P2] IMPROVE-OL-TABLE-FIDELITY — Order List table visual ไม่ตรง mockup (4 issues) `[Wat]`

**Found:** 2026-05-02 | **Type:** FE visual regression + improvement | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** user review 2026-05-02 — เปิด order list จริงแล้วเห็น table ไม่ตรง mockup หลัง IMPROVE-UI-SIMPLE-TABLE + IMPROVE-OL-FILTER-DG-FIDELITY merge
**Mockup:** `docs/mockups/order-list-full-improve.html`

#### Issues

1. **Header column styling ไม่ตรง mockup** — font weight, background color, border ใน `<th>` ไม่ตรงกับ `.tbl-head` ใน mockup
2. **Sticky/freeze header regression** — scroll แล้ว header ไม่ freeze — regression จาก IMPROVE-OL-FILTER-DG-FIDELITY (PR #109); `position: sticky; top: 0` ใน `DataGrid.tsx` อาจหาย/ถูก override
3. **Action column ขาด vertical divider** — ไม่มีเส้น border-left กั้น Action column จาก column อื่น ตาม mockup `border-l` style
4. **Footer & pagination ไม่ตรง mockup** — layout, background, page-size selector, page indicator ไม่ตรงกับ `.tbl-footer` ใน mockup

**Root cause (preliminary):** ปัญหา 2 น่าเป็น CSS specificity หรือ overflow container ที่ทำให้ sticky ไม่ทำงาน; ปัญหา 1/3/4 คือ missing class/style ที่ยังไม่ migrate จาก mockup

**Files (likely):**
- `packages/ui/src/components/ui/DataGrid.tsx` — sticky header, column divider logic
- `apps/b1dx-oms-fulfillment/src/features/orders/components/order-list/OrdersTable.tsx` — column defs, Action column config
- Footer/pagination component (TBD — อยู่ใน DataGrid หรือแยก)

**Test Cases:**
- [ ] regression: scroll order table ลง > 1 viewport — header row ยังมองเห็นที่ top
- [ ] regression: header `<th>` มี background-color ตรงกับ mockup `.tbl-head` (snapshot test)
- [ ] regression: Action column มี border-left แยกจาก column ข้างๆ (visual assertion หรือ CSS class check)
- [ ] regression: footer แสดง page-size selector + page indicator ตาม mockup layout

---

### 🆕 [P1] IMPROVE-UI-SHARED-COMPONENTS — Wrap mockup-style table/state-bar/status-chip → `@b1dx/ui` `[Wat]`

**Found:** 2026-04-28 | **Type:** FE refactor — shared component extraction | **Repo:** `b1dx-oms-fulfillment-web` | **Spec:** based on `docs/mockups/order-list-full-improve.html`
**Source:** user request 2026-04-28 — extract reusable UI primitives จาก mockup ให้ใช้ทั่วทั้ง app (ทุก list page) — ป้องกัน duplicate logic + enforce consistent visual

> **Sequence:** ทำหลัง `IMPROVE-ORDER-LIST-VISUAL-SHELL` merge — แล้ว migrate Wave 0 components ไปใช้ shared
> **Naming:** all components generic, exported from `@b1dx/ui` root (no deep import)

---

### 🆕 [P2] EOD-BE-04 — Add `auditTrail[]` to GET /api/orders/{id} response `[unassigned]`

**Found:** 2026-04-28 | **Type:** BE feature — read-side enrichment | **Branch:** `feat/eod-be-04-audit-trail-field` (TBD) | **Spec:** `docs/specs/spec-order-improve.md` (extend GET response)
**Source:** BUG-EOD-DIALOG-MOCKUP-MISMATCH ต้องแสดง audit trail ใน Edit dialog (mockup line 2651-2656); FE จะ ship UI พร้อม optional field, BE ตามมาเสริม data

#### [P2] EOD-BE-04 `[S]` `[ ]`

**Type:** BE GET endpoint enrichment | **Effort:** 1-2h

- **Symptom:** `EditOrderResponse` / `GET /api/orders/{id}` ไม่มี `auditTrail[]` field — FE ต้อง render `—` (empty state) จนกว่า BE จะ wire data
- **Root cause:** EOD-BE-03 (GET endpoint) ไม่ได้รวม audit log ใน response — น่าจะ join จาก `OrderAuditLog` หรือ `OrderStatusHistory` table ที่มีอยู่
- **Scope:**
  - เพิ่ม `auditTrail: AuditTrailEntry[]` ใน `EditOrderResponse` (Contracts)
  - `AuditTrailEntry`: `{ actor: string, timestamp: DateTime, action: string }`
  - Query existing audit log table → map → return rows ล่าสุด N rows (เสนอ N=10, sorted desc)
  - หรือ derive จาก `order_status_transitions` (ตาม OSM-BE-01) ถ้ายังไม่มี audit log table แยก
- **Test cases:**
  - `[BE]` integration test: GET /api/orders/{id} response มี `auditTrail` array
  - `[BE]` integration test: `auditTrail[]` populate จาก audit log table (3+ rows ใน fixture)
  - `[BE]` integration test: order ที่ไม่มี history → `auditTrail: []` (empty array, ไม่ใช่ null)
  - `[BE]` integration test: tenant isolation — audit trail filtered by tenant_id
- **Dependencies:** EOD-BE-03 ✅ (GET endpoint exists), OSM-BE-01 ✅ (audit log table)
- **Reference:**
  - `docs/mockups/order-detail-improve.html:2651-2656` — audit trail rows visual
  - SaaS principle: audit trail = compliance artifact (SOC2, ISO27001) — ต้อง real data, ห้าม fake

---

### 🆕 [P2] IMPROVE-PERM-ORDERS-EDIT-INTERNAL-NOTE — เพิ่ม fine-grained permission สำหรับ Internal Note `[Wat]`

**Found:** 2026-04-28 | **Type:** BE+FE permission registry extension | **Branch:** `improve/perm-orders-edit-internal-note` (TBD) | **Spec:** spec follow-up
**Source:** discussion ใน BUG-EOD-DIALOG-MOCKUP-MISMATCH (2026-04-28) — ปัจจุบัน Internal Note ใน Edit dialog ใช้ role-derived flag (`currentUserRole === 'wh-*'`) ใน ViewModel hook ซึ่ง pragmatic แต่ไม่ใช่ SaaS-correct long-term เพราะ permission ไม่ได้อยู่ใน registry → ABAC/conditional access engine ตรวจไม่เจอ, audit ไม่ได้

### 🆕 [P2] EOD-BE-AUDIT — audit_logs infrastructure for order edits (BR-08)

**Found:** 2026-04-26 | **Type:** BE feature | **Effort:** M (3-4h)

- **Context:** Spec `spec-order-improve.md` § BR-08 mandates audit_log row in same transaction for every successful PATCH /api/oms/orders/{id}/edit. EOD-BE-02 shipped without this — no `audit_logs` table currently exists in the codebase.
- **Scope:**
  - Create `AuditLog` Domain entity + EF config + migration (`oms.audit_logs`)
  - Columns per spec: `tenant_id`, `actor_user_id`, `action`, `entity`, `entity_id`, `meta_json` (jsonb with `changed_fields[]` + `diff{}` + `reprint_triggered`)
  - Wire into `EditOrderCommandService` — write row in same `SaveChangesAsync` (rollback ทั้งหมดถ้า fail)
  - Capture before/after diff for every recipient/shipping/meta field
- **Test cases:**
  - Successful PATCH → 1 audit_log row exists with correct diff JSON
  - audit_log fail → entire transaction rolled back (Order unchanged)
- **Why deferred:** EOD-BE-02 task scope was edit endpoint logic; audit table is cross-cutting infrastructure that other features will share — better as its own task
- **Follow-up to:** EOD-BE-02 (PR #44, merged 2026-04-26)

### 💡 [P3] IMPR-RBAC-02 — แยก Reports group ออกจาก Admin ใน PERMISSION_GROUPS

**Found:** 2026-04-10 | **Type:** FE improvement | **Effort:** XS

- **Fix:** เพิ่ม group `{ id: 'reports', label: 'Reports', permissions: ['reports.view', 'reports.export'] }` และย้ายออกจาก `admin` group

### 💡 [P3] IMPR-RBAC-01 — Permission Editor: tab-switching navigation guard

**Found:** 2026-04-03 | **Type:** FE improvement | **Effort:** XS

- **Expected:** แสดง confirm dialog "ยกเลิกการเปลี่ยนแปลง?" ก่อน leave
- **Depends on:** ไม่มี (BUG-RBAC-PERM-ALIGN ✅ done)

### 💡 [P3] IMPR-CR-01 — CreateRoleDrawer: Base Role picker ควรแสดง system roles ด้วย

**Found:** 2026-04-10 | **Effort:** XS | **Branch:** —

- **Pending:** ต้องการ spec clarification

### 💡 [P3] IMPR-CR-02 — CreateRoleDrawer: group header labels ไม่รองรับ i18n

**Found:** 2026-04-10 | **Effort:** XS

### 💡 [P3] IMPR-CR-03 — CreateRoleDrawer: reset() ก่อน drawer animation จบอาจทำให้ form กระพริบ

**Found:** 2026-04-10 | **Effort:** XS

### 💡 [P3] IMPR-CR-04 — CreateRoleDrawer: i18n keys `scope_*` ที่เพิ่มไว้ยังไม่ถูกใช้งาน

**Found:** 2026-04-10 | **Effort:** XS

### 🐛 [P2] BUG-E2E-01 — auth e2e 3 tests fail

**Found:** 2026-04-03 | **Assigned:** wat | **Branch:** `fix/bug-e2e-01-auth-test-failures`

- **Failure 1** `ForgotPasswordPage.submitButton` locator ไม่ตรง — button text จริงคือ "📧 ตรวจสอบ"
- **Failure 2** ไม่มี inline error "รหัสผ่านไม่ตรงกัน" — ต้องตรวจสอบ `ResetPasswordForm.tsx`
- **Failure 3** MSW refresh fallback สร้าง session อัตโนมัติ → redirect /login ไป dashboard
- **Fix:** อัปเดต `e2e/auth/auth.po.ts` locators + mock refresh ใน `LoginPage.goto()`

### 🐛 [P2] ISSUE-002: Rename DB Columns to snake_case via Npgsql Naming Convention

- **Status:** 📋 Backlog | **Assigned:** Nuchit | **Reported:** 2026-04-03
- **Root Cause:** EF Core ไม่มี naming convention ตั้งต้น — สร้าง column ชื่อ PascalCase
- **Decision:** Enable `UseSnakeCaseNamingConvention()` + EF Core migration rename columns

**Implementation Plan:**

1. Enable `options.UseNpgsql(conn).UseSnakeCaseNamingConvention()`
2. สร้าง EF Core migration `RenameColumnsToSnakeCase`
3. Verify migration SQL — ต้องเป็น `RenameColumn` ไม่ใช่ `DropColumn` + `AddColumn`
4. ทดสอบบน staging ก่อน

**Acceptance Criteria:**

- [ ] `UseSnakeCaseNamingConvention()` enabled ใน DbContext
- [ ] Migration สร้างด้วย `RenameColumn` (ไม่ใช่ drop+add)
- [ ] `dotnet test` ผ่านทั้งหมด
- [ ] Query ด้วยมือโดยไม่ใส่ quotes ทำงานได้
- [ ] Login / Auth flow ยังทำงานปกติ

---

## GROUP — Preferences Dialog `P3`

> **Spec:** `docs/specs/preferences-dialog-spec.md` ✅ Ready
> **Type:** FE + BE — preferences persist ข้ามเครื่องผ่าน API
> **Effort:** ~13h (6 tasks)
> **Constraints:**
> 
> - Persistence: localStorage ก่อน — abstract ด้วย `PrefsStorage` interface เพื่อ swap API ได้ทีหลัง (YAGNI)
> - Auth tab: dev-only (`NODE_ENV === 'development'`) — security by default
> - Notifications tab: client-only (localStorage) ก่อน — backend wiring เป็น separate task
> - Density: ใช้ CSS variables (`--row-h`, `--cell-py`) ไม่ hardcode per-component
> - Font size: scope ใน `#app-root` ไม่ใช่ `document.documentElement` เพื่อไม่ทำลาย rem-base ของ `@b1dx/ui`

---

### [~] FE-PREF-01 — PreferencesDialog shell + tab routing `[2h]` `[srattha]`

**Goal:** สร้าง modal shell พร้อม sidebar tabs + routing ระหว่าง panel
**Type:** FE — component
**Spec Ref:** `docs/specs/preferences-dialog-spec.md` § 1, 3, 6

**Files to Create:**

- `components/preferences/PreferencesDialog.tsx` — modal container, split layout
- `components/preferences/DialogSidebar.tsx` — 5 tab buttons + active state
- `components/preferences/DialogContent.tsx` — header (title + close) + panel slot + footer (Reset/Done/SaveStatus)
- `hooks/usePreferences.ts` — `Prefs` state + `prefSave()` + `resetPrefs()` + localStorage persistence via `PrefsStorage` interface
- `types/prefs.ts` — `Prefs` interface + `PrefsStorage` interface + `localStorageStorage` impl

**Constraints:**

- `PrefsStorage` interface: `{ load(): Prefs, save(p: Prefs): void }` — swap-ready
- Footer SaveStatus: "✓ Saved" fade out หลัง 2s
- Layout: `max-width: 780px`, `height: 90vh`, sidebar 200px fixed

**Acceptance Criteria:**

- [ ] Modal เปิด/ปิดได้จาก `openPrefs()` call
- [ ] Tab switching เปลี่ยน active panel + header title ถูกต้อง
- [ ] `usePreferences` load จาก localStorage on mount, save on change
- [ ] Reset ปุ่มคืนค่า default ครบทุก field
- [ ] Layout ตรง mockup (sidebar + content split)
- [ ] Unit test: `usePreferences` — load/save/reset

**Dependencies:** None
**Effort estimate:** 2h

---

---

## GROUP — Auth Login Improvements `P2`

> **Spec:** `docs/specs/auth-login-improvements.md`
> **Goal:** ลด login friction โดยไม่ลด security — Default Role, Dev Bypass, Silent Re-auth, Device Trust, Shared Device
> **Effort:** Phase 1 ~4h | Phase 2 ~6h | Phase 3 ~13h | Phase 4 TBD

---

### [P2] AUTH-LOGIN-01 — Default Role: apply skipRoleSelect + defaultRole ใน OtpScreen `[2h]` `[FE]`

**Goal:** user ที่ตั้ง `defaultRole` ใน Preferences → login แล้วข้าม role-select screen อัตโนมัติ
**Type:** FE — Feature (Phase 1)
**Spec Ref:** `docs/specs/auth-login-improvements.md` § Phase 1

**Root Cause:** `skipRoleSelect` + `defaultRole` ถูกบันทึกใน Prefs แต่ไม่ถูกอ่านใน `OtpScreen.handleVerify()` เลย

**Logic ที่ต้องเพิ่มใน `handleVerify()` (หลัง verifyOtp ได้ memberships[]):**
```ts
if (prefs.skipRoleSelect && prefs.defaultRole) {
  const match = memberships.find(m => m.roleKey === prefs.defaultRole)
  if (match) {
    await issueToken(match)
    return
  }
  toast.warning(t('auth.default_role_not_available'))
  // fallback: แสดง role-select screen ปกติ
}
```

**Files:**
- `src/features/auth/components/OtpScreen.tsx` — เพิ่ม prefs check ใน `handleVerify()`
- `src/hooks/usePreferences.ts` — expose `prefs` ให้ OtpScreen ใช้ (ถ้ายังไม่ได้)
- `src/lib/i18n/locales/en.json` + `th.json` — เพิ่ม key `auth.default_role_not_available`

**Test Cases:**
- [ ] P1-TC-01: `skipRoleSelect=true` + `defaultRole='ff-manager'` → ข้าม role-select screen → เข้าแอปทันที
- [ ] P1-TC-02: `skipRoleSelect=true` + `defaultRole=''` → แสดง role-select screen ปกติ (ไม่ crash)
- [ ] P1-TC-03: user มี role เดียว → auto-select เสมอ (existing behavior ไม่เปลี่ยน)
- [ ] P1-TC-04: `defaultRole='ff-manager'` แต่ role ถูกลบ → fallback role-select + toast warning
- [ ] P1-TC-05: `defaultRole='wh-admin'` แต่ user มีแค่ `ff-manager` → พฤติกรรมเดียวกับ TC-04
- [ ] P1-TC-06: `skipRoleSelect=false` → role-select screen เสมอ แม้มี `defaultRole` set
- [ ] P1-TC-11: user ไม่เคยตั้ง prefs → login ปกติ ไม่มี regression
- [ ] P1-TC-12: `defaultMembershipId` จาก BE ยังทำงาน (existing logic ไม่เสีย)

**Acceptance Criteria:**
- [ ] `skipRoleSelect=true` + `defaultRole` ตรงกับ membership → ข้าม role-select
- [ ] `defaultRole` ไม่พบ → toast + fallback (ไม่ crash)
- [ ] `skipRoleSelect=false` → ไม่ข้ามเสมอ
- [ ] existing auto-select (single membership, defaultMembershipId) ยังทำงาน

**Dependencies:** FE-PREF-04 (done)
**Effort estimate:** 2h

---

### [P2] AUTH-LOGIN-02 — Default Role: Preferences UI ออกจาก dev-only + dropdown จาก memberships `[2h]` `[FE]`

**Goal:** `skipRoleSelect` + `defaultRole` controls ให้ทุก user ใช้ได้ และ dropdown แสดง roles จริงของ user
**Type:** FE — Feature (Phase 1)
**Spec Ref:** `docs/specs/auth-login-improvements.md` § Phase 1 — UI Changes

**Changes:**
- `PanelAuth.tsx`: ย้าย `skipRoleSelect` + `defaultRole` ออกจาก dev-only guard (คง `skipOtp` ไว้ก่อน)
- `PanelAuth.tsx`: `defaultRole` dropdown เปลี่ยนจาก hardcode ROLE_OPTIONS → memberships จาก `authStore.user`
- `DialogSidebar.tsx`: Rename tab "Auth & Login" → "Login"
- `en.json` / `th.json`: อัพเดท i18n keys

**Files:**
- `src/components/preferences/PanelAuth.tsx`
- `src/components/preferences/DialogSidebar.tsx`
- `src/lib/i18n/locales/en.json` + `th.json`

**Test Cases:**
- [ ] P1-TC-07: `skipRoleSelect` + `defaultRole` controls แสดงสำหรับทุก user (ไม่ใช่แค่ dev)
- [ ] P1-TC-08: `defaultRole` dropdown แสดง roles จาก user's memberships จริง ณ ขณะนั้น
- [ ] P1-TC-09: ตั้ง `defaultRole` → reopen Preferences → dropdown แสดงค่าที่เลือกไว้
- [ ] P1-TC-10: เปลี่ยน `defaultRole` → Logout → Login ใหม่ → ใช้ค่าใหม่

**Acceptance Criteria:**
- [ ] controls แสดงในทุก environment
- [ ] dropdown options = memberships ของ user จริง
- [ ] Tab ใน sidebar ชื่อ "Login"

**Dependencies:** AUTH-LOGIN-01
**Effort estimate:** 2h

---

### [P2] AUTH-LOGIN-03 — Dev OTP Bypass via env var (แทน UI toggle) `[3h]` `[FE + BE]`

**Goal:** Dev/QA ข้าม OTP ได้ผ่าน env var + BE magic code — ลบ `skipOtp` UI toggle ออก
**Type:** FE + BE — Improvement (Phase 2)
**Spec Ref:** `docs/specs/auth-login-improvements.md` § Phase 2A

**Root Cause:** `skipOtp` toggle ใน UI = user-controlled security bypass = ไม่ควรมีใน production
**Solution:** BE ยอมรับ magic code `000000` เฉพาะ dev env, FE auto-fill จาก env var

**BE Changes:**
- `Application/Auth/VerifyOtpHandler.cs`: ถ้า `env=Development` และ code ตรงกับ `Auth:MagicOtpCode` → verify สำเร็จ
- `appsettings.Development.json`: `"Auth": { "MagicOtpCode": "000000" }`

**FE Changes:**
- `OtpScreen.tsx`: dev auto-fill จาก `NEXT_PUBLIC_DEV_OTP_CODE` env var (guard ด้วย `NODE_ENV=development`)
- `PanelAuth.tsx`: ลบ `skipOtp` toggle
- `types/prefs.ts`: ลบ `skipOtp` field
- `.env.local.example`: เพิ่ม `# NEXT_PUBLIC_DEV_OTP_CODE=000000`

**Test Cases:**
- [ ] P2-TC-01: `NEXT_PUBLIC_DEV_OTP_CODE=000000` + BE dev mode → OTP screen auto-submit ทันที
- [ ] P2-TC-02: production build → dev auto-fill ไม่ทำงาน
- [ ] P2-TC-03: `skipOtp` toggle ไม่แสดงใน Preferences ทุก environment
- [ ] P2-TC-BE-01: BE dev + code `000000` → verify สำเร็จ
- [ ] P2-TC-BE-02: BE production + code `000000` → verify ล้มเหลว

**Acceptance Criteria:**
- [ ] Dev ข้าม OTP ได้ผ่าน env var โดยไม่ต้องมี UI
- [ ] Production: ไม่มีทางข้าม OTP ผ่าน env var ได้
- [ ] `skipOtp` field + toggle หายออกจาก codebase ทั้งหมด

**Dependencies:** AUTH-LOGIN-02
**Effort estimate:** 3h

---

### [P2] AUTH-LOGIN-04 — Silent Re-auth: 401 interceptor + refresh retry `[3h]` `[FE + BE]`

**Goal:** access token expire ระหว่างทำงาน → silent refresh → user ไม่รู้ตัว ไม่ต้อง OTP ซ้ำ
**Type:** FE + BE — Improvement (Phase 2)
**Spec Ref:** `docs/specs/auth-login-improvements.md` § Phase 2B

**Root Cause:** `AuthProvider.bootstrap()` มี `refresh()` แต่ยังไม่มี 401 interceptor ที่ retry request หลัง silent refresh สำเร็จ → user ถูก redirect login แทน

**BE Changes:**
- ตรวจ Refresh Token TTL → ปรับให้เป็น 7–14 วัน ถ้ายังสั้นอยู่

**FE Changes — `src/lib/api/coreRequest.ts`:**
- เพิ่ม response interceptor: 401 → silent `authApi.refresh()` → retry request เดิม 1 ครั้ง
- ถ้า refresh ล้มเหลว → redirect login (session หมดจริง)
- Guard: ป้องกัน concurrent 401 เรียก refresh ซ้ำหลายครั้ง (queue pattern)

**Test Cases:**
- [ ] P2-TC-04: access token expire → request ถัดไป → silent refresh → retry สำเร็จ → user ไม่เห็น redirect
- [ ] P2-TC-05: access token + refresh token expire → redirect login (ไม่ retry loop)
- [ ] P2-TC-06: หลาย requests fail 401 พร้อมกัน → refresh เรียกแค่ 1 ครั้ง (no race condition)
- [ ] P2-TC-07: network error → ไม่เรียก refresh (network error ≠ 401)

**Acceptance Criteria:**
- [ ] access token expire → user ทำงานต่อได้ ไม่มี interrupt
- [ ] refresh token expire → redirect login ไม่ loop
- [ ] concurrent 401 → refresh เรียกแค่ครั้งเดียว

**Dependencies:** AUTH-LOGIN-03
**Effort estimate:** 3h

---

### [P3] AUTH-LOGIN-05 — Device Trust "Remember this device 30 days" `[13h]` `[FE + BE]`

**Goal:** หลัง OTP verify user เลือก "Remember this device" → login ครั้งต่อไปจาก device เดิมข้าม OTP ได้ (30 วัน)
**Type:** FE + BE — Feature ใหม่ (Phase 3)
**Spec Ref:** `docs/specs/auth-login-improvements.md` § Phase 3

**BE Tasks (~8h):**
- [ ] DB migration: `user_device_tokens` (`id`, `user_id`, `token_hash` SHA-256, `created_at`, `expires_at`, `revoked_at`, `device_hint`)
- [ ] `POST /auth/verify-otp`: รับ `trust_device: bool` → สร้าง token + set httpOnly cookie `dtt` (30 วัน, Secure, SameSite=Strict)
- [ ] `POST /auth/sign-in`: ตรวจ cookie `dtt` → valid → `otpRequired: false` + `preAuthToken` + `memberships` ใน response
- [ ] `POST /auth/devices/revoke`: revoke trusted device
- [ ] `GET /auth/devices`: list trusted devices ของ user

**FE Tasks (~5h):**
- [ ] `lib/auth/deviceFingerprint.ts` (ใหม่): UUID v4 เก็บ localStorage เป็น device identifier
- [ ] `authApi.ts`: ส่ง `X-Device-Fingerprint` header ทุก sign-in
- [ ] `LoginForm.tsx`: ถ้า `otpRequired=false` → ข้าม OTP screen
- [ ] `OtpScreen.tsx`: checkbox "Remember this device for 30 days" + ส่ง `trust_device` flag
- [ ] Profile/Preferences: Trusted Devices list + revoke button

**Test Cases:**
- [ ] P3-TC-01: Login + check "Remember" → cookie `dtt` set (httpOnly, 30 วัน, Secure, SameSite=Strict)
- [ ] P3-TC-02: Login ครั้ง 2 จาก device เดิม → ข้าม OTP screen ทันที
- [ ] P3-TC-03: Login จาก device ใหม่ → OTP ยังต้องกรอกปกติ
- [ ] P3-TC-04: cookie `dtt` expire → OTP required อีกครั้ง
- [ ] P3-TC-05: user revoke device → OTP required ทันที
- [ ] P3-TC-06: ไม่ check "Remember" → ไม่มี cookie → OTP ทุกครั้ง
- [ ] P3-TC-07: `dtt` tampered → BE reject → OTP required (ไม่ 500)

**Acceptance Criteria:**
- [ ] Cookie: httpOnly, Secure, SameSite=Strict
- [ ] Token hash ใน DB ใช้ SHA-256 (ไม่เก็บ raw token)
- [ ] User revoke trusted devices ผ่าน UI ได้
- [ ] Trust ไม่ข้ามระหว่าง users บน device เดียวกัน

**Note:** ต้องมี spec review + security review ก่อน implement
**Dependencies:** AUTH-LOGIN-04
**Effort estimate:** 13h

---

### [P3] AUTH-LOGIN-06 — Shared Device / WH Worker Login `[TBD]` `[FE + BE]`

**Goal:** WH Worker ใช้ shared tablet login/logout ระหว่างกะได้โดยไม่ต้องรับ OTP บนมือถือส่วนตัว
**Type:** FE + BE — Feature ใหม่ (Phase 4)
**Spec Ref:** `docs/specs/auth-login-improvements.md` § Phase 4

**Options (ต้องตัดสินใจก่อน spec):**
- **Option A — Shift PIN**: WH Admin login OTP ครั้งแรก → ตั้ง PIN 6 หลัก → Worker ใช้ PIN เข้ากะ
- **Option B — QR Login**: Worker scan QR จาก mobile app → login tablet ทันที

**Next Step:** เลือก Option → เขียน spec `docs/specs/auth-shared-device.md` → breakdown tasks

**Effort estimate:** TBD (~3–5 วันหลังมี spec)

---

## GROUP — Profile Settings Page `P2`

> **Spec:** `docs/specs/profile-settings-page-spec.md` ✅ Decisions finalised 2026-04-17
> **Effort:** ~22h (6 BE + 9 FE tasks)
> **Goal:** Full-page profile settings — personal info, password, 2FA (TOTP), notifications, theme/locale preferences
> **Decisions:** Email readonly (admin only), TOTP 2FA, notifications wire to BE, beforeunload dirty warning, plan from user object, avatar server-side resize 128px

---

### [~] BE-PRFSETTINGS-04: `PATCH /api/v1/users/me/avatar` — upload + resize `[2h]` `assigned: srattha` `claimed: 2026-04-30`

**Goal:** รับ `multipart/form-data` → resize server-side เป็น 128×128 px → store S3-compatible → return `avatarUrl`
**Type:** BE — Infrastructure + Api
**Spec Ref:** `docs/specs/profile-settings-page-spec.md` § 17 Q1, § 18 API Endpoints

**Files to Create/Modify:**

- `Infrastructure/Storage/AvatarStorageService.cs` — resize (128px) + upload
- `Application/Users/Commands/UploadAvatarCommand.cs` — command + handler
- `Api/Endpoints/Users/UploadAvatarEndpoint.cs` — `PATCH /api/v1/users/me/avatar`
- `Application/Users/Commands/UploadAvatarCommandTests.cs` — unit test

**Acceptance Criteria:**

- [ ] accept `image/jpeg`, `image/png`, `image/webp` เท่านั้น — 400 ถ้า type อื่น
- [ ] max file size 5MB — 400 ถ้าเกิน
- [ ] resize เป็น 128×128 px ก่อน store
- [ ] response: `{ "avatarUrl": "https://..." }`
- [ ] Unit test: valid upload, wrong type, oversize

**Dependencies:** BE-PRFSETTINGS-01
**Effort estimate:** 2h

---

### [~] FE-PRFSETTINGS-01: `ProfileUser` types + Zod schema `[0.5h]`

**Goal:** Type definitions และ Zod validation schemas สำหรับทั้งหน้า
**Type:** FE — types
**Spec Ref:** `docs/specs/profile-settings-page-spec.md` § 2 Data Interface

**Files to Create:**

- `features/users/types/profileSettings.ts` — `ProfileUser`, `NotificationPrefs`, `DisplayPrefs`
- `features/users/schemas/profileSettings.ts` — Zod schemas: `updateProfileSchema`, `changePasswordSchema`, `notificationPrefsSchema`, `displayPrefsSchema`

**Acceptance Criteria:**

- [ ] `ProfileUser` interface ครบ 18 fields ตาม spec
- [ ] `updateProfileSchema` validate bio max 300 chars
- [ ] `changePasswordSchema` validate: min 8, uppercase, number, special char
- [ ] `pnpm typecheck` ผ่าน

**Dependencies:** none
**Effort estimate:** 0.5h

---

### [P2] FE-PRFSETTINGS-03: `LeftCard` component `[1.5h]`

**Goal:** Cover photo, avatar overlap, contact items, session pill, status block
**Type:** FE — View Component
**Spec Ref:** `docs/specs/profile-settings-page-spec.md` § 4

**Files to Create:**

- `features/users/components/ProfileLeftCard/ProfileLeftCard.tsx`
- `features/users/components/ProfileLeftCard/ProfileLeftCard.test.tsx`

**Acceptance Criteria:**

- [ ] Cover gradient + edit button
- [ ] Avatar overlap (`-33px`) + hover camera icon + online dot
- [ ] Contact items: email, phone, location (monospace, ellipsis)
- [ ] Session pill — pulsing dot + lastLogin text
- [ ] Status block: emailVerified (green), twoFAEnabled (amber/green), plan (indigo), memberSince
- [ ] Unit test: renders all sections, correct color classes per status

**Dependencies:** FE-PRFSETTINGS-01
**Effort estimate:** 1.5h

---

### [P2] FE-PRFSETTINGS-06: `Tab2Notifications` — channel pill toggles `[1.5h]`

**Goal:** 5 notification rows × 3 channels (Email/Push/SMS) เป็น toggle pills
**Type:** FE — View Component
**Spec Ref:** `docs/specs/profile-settings-page-spec.md` § 8

**Files to Create:**

- `features/users/components/Tab2Notifications/Tab2Notifications.tsx`
- `features/users/components/Tab2Notifications/Tab2Notifications.test.tsx`

**Acceptance Criteria:**

- [ ] 2 groups: Orders & Fulfillment (3 rows), System & Security (2 rows)
- [ ] แต่ละ row มี 3 channel pills toggle ได้
- [ ] Default state ตาม spec § 8 table
- [ ] toggle → trigger `markDirty`
- [ ] Unit test: renders all rows, toggle pill changes state, markDirty called

**Dependencies:** FE-PRFSETTINGS-02
**Effort estimate:** 1.5h

---

### [P2] FE-PRFSETTINGS-07: `Tab3Settings` — locale/theme/density/danger zone `[1.5h]`

**Goal:** ภาษา/timezone/date format selects, theme + density radio cards, danger zone buttons
**Type:** FE — View Component
**Spec Ref:** `docs/specs/profile-settings-page-spec.md` § 9

**Files to Create:**

- `features/users/components/Tab3Settings/Tab3Settings.tsx`
- `features/users/components/Tab3Settings/Tab3Settings.test.tsx`

**Acceptance Criteria:**

- [ ] 4 locale fields: language, timezone, dateFormat, timeFormat
- [ ] Theme radio cards: Dark/Light/System (`.pi.sel` pattern, `selPref` uses `.closest('.pref-row-opts')`)
- [ ] Density radio cards: Comfortable/Compact
- [ ] Danger zone: Logout ทุกอุปกรณ์ + ปิดการใช้งานบัญชี (`.btn-dng`)
- [ ] เปลี่ยน locale/theme → trigger `markDirty`
- [ ] Unit test: radio card selection, danger buttons render, markDirty on change

**Dependencies:** FE-PRFSETTINGS-02
**Effort estimate:** 1.5h

---

### [ ] DOC-PRFSETTINGS-11: Update `profile-settings-page-spec.md` → v3 (align with mockup v3)

**Goal:** อัพเดท spec ให้ตรงกับ `docs/mockups/profile_settings_v3.html` — เพิ่ม endpoints ที่ขาด, แก้ layout description, ตัดสินใจ open questions
**Type:** Docs — spec update
**Mockup Ref:** `docs/mockups/profile_settings_v3.html`

**Files to Update:**
- `docs/specs/profile-settings-page-spec.md` — version bump → v3

**Changes required:**
- [ ] § 1 Overview: แก้ layout description → single-column + permission card + 4 tabs
- [ ] § 2 Data Interface: เพิ่ม `avatarUrl`, `completionPct`, `passkeyEnabled`, `trustedDeviceCount`
- [ ] § 18 API Endpoints: เพิ่ม Sessions (GET/DELETE), Activity Log (GET paginated), Passkey (POST/DELETE), Permission Request (POST)
- [ ] § 17 Decisions: เพิ่ม Q7 Email change (user-request flow vs admin-only), Q8 Passkey MVP scope, Q9 Activity log pagination
- [ ] Version header: v3 + date

**Acceptance Criteria:**
- [ ] spec อ้าง mockup `profile_settings_v3.html` ถูกต้อง
- [ ] ทุก endpoint ที่ mockup v3 ใช้มีใน § 18
- [ ] ไม่มี open question ที่ยังไม่ตัดสินใจ

**Dependencies:** none
**Effort estimate:** 1h

---

## Profile Settings v3 — FE Implementation (FE-PRFSETTINGS-11 → 24)

> **Master plan:** `tasks/plans/FE-PRFSETTINGS-V3/plan.md`
> **Spec:** `docs/specs/profile-settings-page-spec.md` v3
> **Mockup:** `docs/mockups/profile_settings_v3.html`
> **Total:** 14 tasks · ~24.5h

### [ ] FE-PRFSETTINGS-17: `PasskeySection` + WebAuthn Modal `[3h]`

**Goal:** Passkey section ใน Tab1Security + register flow (WebAuthn API)
**Type:** FE — new sub-components
**Mockup Ref:** § Tab 1 → Passkey / Biometric

**Files to CREATE:**
- `components/Tab1Security/PasskeySection.tsx`
- `components/Tab1Security/PasskeyModal.tsx`
- `components/Tab1Security/PasskeySection.test.tsx`
- `components/Tab1Security/PasskeyModal.test.tsx`

**AC:**
- [ ] Status row: ตั้งค่าแล้ว/ยังไม่ตั้ง
- [ ] "ตั้งค่า" → modal เรียก `navigator.credentials.create()` (mocked ใน test)
- [ ] Success → POST `/passkeys` → updates `passkeyEnabled`
- [ ] "ลบ" → DELETE `/passkeys/:id`
- [ ] Browser ไม่รองรับ → fallback message

**Dependencies:** FE-PRFSETTINGS-11
**Effort:** 3h

---

### [ ] FE-PRFSETTINGS-18: `ActiveSessionsSection` `[1.5h]`

**Goal:** Sessions list wired กับ API + Revoke + Logout All
**Type:** FE — new sub-component
**Mockup Ref:** § Tab 1 → Sessions ที่ Active

**Files to CREATE:**
- `components/Tab1Security/ActiveSessionsSection.tsx`
- `components/Tab1Security/ActiveSessionsSection.test.tsx`

**AC:**
- [ ] List แสดงจาก `GET /sessions`
- [ ] Current session: badge "อุปกรณ์นี้", ไม่มีปุ่ม Revoke
- [ ] Revoke → DELETE `/sessions/:id` + slide out
- [ ] "Logout ทั้งหมด" → confirm modal → DELETE `/sessions`
- [ ] Loading state, error retry

**Dependencies:** FE-PRFSETTINGS-11
**Effort:** 1.5h

---

### [ ] FE-PRFSETTINGS-19: `ActivityLogSection` `[2h]`

**Goal:** Activity log section + filter chips + expand row detail
**Type:** FE — new sub-component
**Mockup Ref:** § Tab 1 → กิจกรรมล่าสุด

**Files to CREATE:**
- `components/Tab1Security/ActivityLogSection.tsx`
- `components/Tab1Security/ActivityLogSection.test.tsx`

**AC:**
- [ ] List จาก `GET /activity-log` (5 รายการล่าสุด)
- [ ] Filter chips: all / login / action / error (client-side filter)
- [ ] Click row → expand detail (IP, browser, OS)
- [ ] Icon color ตาม type (✓green, ✏ind, !red, ↩teal)
- [ ] "ดูทั้งหมด →" → toast (full page = future)
- [ ] Timestamp ใช้ user timezone

**Dependencies:** FE-PRFSETTINGS-11
**Effort:** 2h

---

### [ ] FE-PRFSETTINGS-20: `Tab2Notifications` wire BE `[1h]`

**Goal:** ตรวจ + wire `PATCH /notifications` (อาจ wire แล้วใน v1 — ตรวจก่อน)
**Type:** FE — wire/verify
**Mockup Ref:** § Tab 2

**Files to MODIFY:**
- `components/Tab2Notifications/Tab2Notifications.tsx` — verify save → PATCH
- `components/Tab2Notifications/Tab2Notifications.test.tsx`

**AC:**
- [ ] Master toggle ปิด → opacity 0.45 + pointer-events none
- [ ] Channel pill toggle → state update
- [ ] Save → PATCH body: `{ orderNew: { email, push, sms }, ... }`
- [ ] Reset → refetch จาก API

**Dependencies:** FE-PRFSETTINGS-11
**Effort:** 1h

---

### [ ] FE-PRFSETTINGS-21: `Tab3Settings` wire + locale preview fix `[1h]`

**Goal:** Wire `PATCH /preferences` + แก้ locale preview ที่ใช้ hardcoded date `2025-03-17`
**Type:** FE — wire/fix
**Mockup Ref:** § Tab 3 → ภาษาและภูมิภาค

**Files to MODIFY:**
- `components/Tab3Settings/Tab3Settings.tsx` — `new Date()` แทน hardcoded
- `components/Tab3Settings/Tab3Settings.test.tsx`

**AC:**
- [ ] Locale preview ใช้ current date
- [ ] Format change → preview updates real-time
- [ ] Save → PATCH `/preferences`
- [ ] Reset → refetch

**Dependencies:** FE-PRFSETTINGS-11
**Effort:** 1h

---

### [ ] FE-PRFSETTINGS-26: Theme apply global + Light/Dark verify `[1h]`

**Goal:** ตรวจ theme toggle ใน ProfileHeader apply ทั้ง app (ไม่ใช่แค่ Profile page) + verify v3 components ดูถูกต้องทั้ง dark + light theme
**Type:** FE — verify/wire
**Spec Ref:** § 9.2 ธีม & การแสดงผล + mockup `data-theme="light"` variants

**Files to MODIFY:**
- `components/ProfileHeader/ProfileHeader.tsx` — wire theme toggle ผ่าน global theme context (ไม่ใช่ local state)
- ทุก v3 component (FE-12~22) — verify CSS ใช้ CSS variables (var(--bg), var(--text), ฯลฯ) ไม่มี hardcoded color

**AC:**
- [ ] กด theme toggle ใน ProfileHeader → ทั้ง app เปลี่ยน theme (ไม่ใช่แค่หน้านี้)
- [ ] Light theme: ทุก component (ProfileHeader, PermissionCard, modals, sections) ดูถูกต้อง — ไม่มี contrast issue
- [ ] Dark theme: เหมือนกัน
- [ ] ไม่มี hardcoded color (#fff, rgba(0,0,0,...) ฯลฯ) ใน v3 components — ใช้ CSS vars
- [ ] System theme: respect `prefers-color-scheme` media query

**Dependencies:** FE-PRFSETTINGS-12~22, FE-PRFSETTINGS-23
**Effort:** 1h

---

## GROUP — Integration Settings `P1`

> **Spec:** `docs/specs/integration-settings-spec.md` v1.1 ✅ Ready for implementation
> **Mockup:** `docs/mockups/integration-settings.html` ✅
> **Effort:** BE ~3d · FE ~3d · QA ~1d
> **Goal:** CRUD integration settings per tenant — 9 platforms, 4-tab modal, envelope encryption, alert deduplication, log export CSV
> **Assigned:** srattha (fullstack) `claimed: 2026-04-26`
> **Branch:** `feat/integration-settings`

> ✅ All Done — ดู tasks/archive/2026-05-backlog.md

---

## GROUP — Courier Settings `P1`

> **Spec:** `docs/specs/courier-settings.md` v1.0 ✅ Ready for implementation
> **Mockup:** `docs/mockups/courier-settings.html` ✅
> **Effort:** CRSET-01 ~3-4d + CRSET-02 ~4-5d (BE+FE+DB+E2E)
> **Goal:** CRUD courier providers per tenant — 8 built-in + custom, 3 modes (api/manual/hybrid), 5-tab detail drawer, credential encryption, test connection, pricing tiers, pickup schedule, 30d stats
> **Workflow:** Sequential — CRSET-01 merge ก่อน → CRSET-02 build บน main ที่มี Courier table แล้ว

### [P1] CRSET-01 — Courier Settings: List + CRUD (BE+FE+DB) `[3-4d]` `[x]` `[srattha]` `claimed: 2026-05-13` `merged: 2026-05-14 (PR #178)`

**Type:** Full-stack feature
**Plan:** `tasks/plans/CRSET-01/plan.md`
**Branch:** `feat/courier-settings-list`

**Scope:**
- DB: migration `couriers` table + indexes + partial unique on `is_default`
- BE: `Courier` aggregate, 6 commands/queries (List/Create/Delete/ToggleStatus/SetDefault/Export), 6 API endpoints
- FE: list page + KPIs + filter bar + table + create modal (type picker) + delete modal
- Permissions: `courier.read`, `courier.write`, `courier.delete`
- E2E: `e2e/courier-settings-list/` — happy + edge + permission

**Test Cases (บังคับ):**
- [ ] List renders 6 mock couriers w/ KPIs (Total=6, Active=4, Error=1, Inactive=1)
- [ ] Filter chip 'active' → 4 rows
- [ ] Search 'flash' → 1 row
- [ ] Create SPX → row appears + toast
- [ ] Delete row → confirm → removed + toast
- [ ] Duplicate code on create → DB constraint error → user-friendly toast
- [ ] FF Member: no Add/Delete buttons (PermissionGate)
- [ ] Multi-tenant: tenant A cannot see tenant B couriers
- [ ] Partial unique idx: cannot have 2 defaults per tenant

**Acceptance Criteria:**
- [ ] BE: `dotnet build && dotnet test` green
- [ ] FE: `pnpm lint && pnpm typecheck && pnpm test` green
- [ ] E2E `playwright test e2e/courier-settings-list/` all pass
- [ ] Migration up + down clean
- [ ] Mockup visual fidelity verified (screenshot vs courier-settings.html list section)

**Effort estimate:** 3-4d

---

### [P1] CRSET-02 — Courier Settings: Detail Drawer + Tabs (BE+FE+DB) `[4-5d]` `[x]` `[srattha]` `claimed: 2026-05-13` `merged: 2026-05-15 (PR #185 / BE #75) — w/ deferrals → CRSET-02A..D`

**Type:** Full-stack feature
**Plan:** `tasks/plans/CRSET-02/plan.md`
**Branch:** `feat/courier-settings-detail`
**Depends on:** CRSET-01 merged

**Scope:**
- DB: migration `courier_pricing_rows` + encrypted credential columns (BYTEA)
- BE: `CourierPricingRow` VO, `ICourierAdapter` interface, SPX adapter (real) + stub adapter, factory, Redis stats cache, rate-limiter
- BE: 6 detail handlers (Get/Update/UpdateCredentials/UpdatePricing/TestConnection/Stats)
- BE: 6 API endpoints + `?reveal=true` + rate-limit middleware
- FE: `CourierDetailDrawer` orchestrator + 5 tabs (General/Credentials/Pricing/Pickup/Danger)
- FE: Stats card (30d), unsaved-changes guard, view↔edit toggle, test-connection button
- Permissions: `courier.credentials.write`, `courier.test_connection`
- E2E: `e2e/courier-settings-detail/` — happy + dirty-guard + permission

**Test Cases (บังคับ):**
- [ ] Drawer opens with 5 tabs, General active by default
- [ ] Edit name → close w/o save → unsaved guard dialog
- [ ] Pricing tab: add 2 rows → save → reopen → both present
- [ ] Pricing tab: overlap weight range → zod error
- [ ] Credentials tab: paste API key → Test Connection (SPX) → success toast w/ latency
- [ ] Test Connection rate-limit: 11th call within 1 min → 429
- [ ] Stats card: 30d data shown, 2nd open within 5min → cache hit (no API call)
- [ ] `?reveal=true` for admin → plaintext + audit log row
- [ ] FF Member view drawer: credentials masked, no Edit/Test buttons
- [ ] Visual fidelity: drawer accent bar, header logo+pill, 5 tab indicators

**Acceptance Criteria:**
- [ ] BE: `dotnet build && dotnet test` green
- [ ] FE: `pnpm lint && pnpm typecheck && pnpm test` green
- [ ] E2E `playwright test e2e/courier-settings-detail/` all pass
- [ ] Migration up + down clean
- [ ] Mockup visual fidelity verified (screenshot vs drawer section of courier-settings.html)
- [ ] Redis cache hit verified, rate-limit verified

**Effort estimate:** 4-5d

**Merge note (2026-05-15):** MVP shipped — fully functional with placeholders. 9 scope items deferred to follow-up tickets CRSET-02A..D below:
1. SpxCourierAdapter (real ping) — only `StubCourierAdapter` present
2. CourierAdapterFactory (per-type) — single stub used for all types
3. `InMemoryCourierStatsCache` instead of Redis (no cross-instance, no stampede protection)
4. `InMemoryTestConnectionRateLimiter` instead of Redis
5. `ZeroCourierStatsProvider` — Stats card returns 0 for all 4 metrics
6. FE 5 tab unit tests (`*Tab.test.tsx`) missing — hook + e2e cover behavior
7. E2E 5/7 still `.skip` (E1–E5: happy save/pricing/credentials/dirty-guard/danger)
8. Manual mockup visual-fidelity screenshot review pending
9. Audit-log row on `?reveal=true` — implementation needs verification

---

### [P1] CRSET-02A — Real Courier Stats Provider (Shipment EF query) `[3-4h]` `[x]` `[srattha]` `claimed: 2026-05-15` `merged: 2026-05-15 (commit bd60702)`

**Type:** BE feature (Infrastructure)
**Branch:** `feat/crset-02a-courier-stats-provider`
**Plan:** `tasks/plans/CRSET-02A/plan.md` (Confirmed 2026-05-15)
**Depends on:** CRSET-02 merged
**Why P1:** Stats card ใน Courier Detail Drawer แสดง 0 ทุกค่า — user เห็น UI แต่ข้อมูลหลอก

**Scope (phase นี้):**
- Replace `ZeroCourierStatsProvider` → `ShipmentCourierStatsProvider` (real EF query over `AppDbContext.Shipments` join `Orders`)
- 4 metric: `Orders30d`, `SuccessRate`, `AvgDeliveryHours`, `ReturnRate` ใน 30-day window
- Swap DI registration; keep `ZeroCourierStatsProvider` สำหรับ test fallback
- Filter ด้วย legacy `CourierId` (OAD `CourierCode` ไม่นับใน phase นี้)

**Deferred → follow-up tickets:**
- SPX Adapter (`SpxCourierAdapter`) — real ping `GET {apiEndpoint}/ping` w/ auth + latency + error mapping
- `CourierAdapterFactory` — DI wiring สำหรับ Test Connection
- OAD `CourierCode` → stats mapping

**Test Cases:**
- [ ] Courier ไม่มี shipments → `(0, 0, 0, 0)` ไม่ throw
- [ ] 10 shipments / 8 Delivered + 1 Returned + 1 InTransit → `Orders30d=10, SuccessRate=0.8, ReturnRate=0.1`
- [ ] Delivered shipment `DeliveredAt - CreatedAt = 24h` → `AvgDeliveryHours=24`
- [ ] Shipment `CreatedAt < now-30d` → ไม่ถูกนับ (window filter)
- [ ] Shipment tenant อื่น → ไม่ถูกนับ (multi-tenant isolation)
- [ ] Soft-deleted shipment (`DeletedAt != null`) → ไม่ถูกนับ
- [ ] 10 shipments ไม่มี Delivered → `AvgDeliveryHours=0` ไม่ NaN
- [ ] Integration (TestContainers Postgres) — happy path

**Acceptance Criteria:**
- [ ] BE: `dotnet build && dotnet test` green (unit + integration)
- [ ] Stats card UI แสดง non-zero สำหรับ courier ที่มี shipment จริง

**Effort estimate:** 3-4h (scope ลดลงจาก 2-3d — SPX+Factory deferred)

---

### [P2] CRSET-02B — Redis Stats Cache + Test-Connection Rate Limiter `[1-2d]` `[x]` `[srattha]` `merged: 2026-05-15` · plan: `tasks/plans/CRSET-02B/plan.md`

**Type:** BE infra (caching + rate-limit)
**Depends on:** CRSET-02 merged · CRSET-02A (recommended)
**Why P2:** ปัจจุบัน in-memory ทำงานได้ที่ single-instance — multi-pod deploy จะ broken (cache miss + rate-limit bypass per pod)

**Scope:**
- `RedisCourierStatsCache` (replace `InMemoryCourierStatsCache`) — `SET NX EX 300` lock for stampede protection, fall-through to provider on miss
- `RedisTestConnectionRateLimiter` (replace `InMemoryTestConnectionRateLimiter`) — sliding window key `rl:courier:test:{tenant}:{user}:{courier}` TTL 60s, limit 10/min
- DI swap behind feature flag `Couriers:UseRedis` (default true in prod, false in test)
- Connection multiplexer reuse (existing Redis client if any in solution)

**Test Cases:**
- [ ] Cache 2nd call within TTL → no provider invocation (verify via mock)
- [ ] Cache stampede — 10 concurrent misses → only 1 provider call (lock works)
- [ ] Rate limit 11th call within 60s → `RateLimitExceededException` / 429
- [ ] Rate limit per-(tenant,user,courier) — different user/courier counts independently
- [ ] Redis down → graceful fallback (return uncached + log warn, don't crash)

**Acceptance Criteria:**
- [ ] Integration test w/ Testcontainers Redis — cache + rate-limit pass
- [ ] Verify via `redis-cli MONITOR` in dev env

**Effort estimate:** 1-2d

---

### [P2] CRSET-02C — FE Tab Unit Tests + Un-skip E2E E1–E5 `[1-2d]` `[x]` `[srattha]` `merged: 2026-05-18 PR #191` (E2E un-skip → CRSET-02C-E2E)

**Type:** FE tests
**Depends on:** CRSET-02 merged

**Scope:**
- 5 tab unit tests: `GeneralTab.test.tsx`, `CredentialsTab.test.tsx`, `PricingTab.test.tsx`, `PickupTab.test.tsx`, `DangerTab.test.tsx` — cover 13 cases listed in CRSET-02 plan §Test Plan / Unit (FE)
- Un-skip E2E E1–E5 in `e2e/courier-settings-detail/courier-settings-detail.spec.ts`:
  - E1: happy edit name → save → reopen → persisted
  - E2: pricing add 2 rows → save → reopen → both present
  - E3: credentials paste → Test Connection → success toast
  - E4: dirty edit → close → unsaved dialog
  - E5: danger disable → toast → status=inactive in list
- Fix MSW handler drift causing E1–E5 to be skipped originally

**Test Cases:** (this IS the test work — covered above)

**Acceptance Criteria:**
- [ ] `pnpm test` — 5 new tab test files pass
- [ ] `pnpm playwright test e2e/courier-settings-detail/` — 7/7 pass (no skips)

**Effort estimate:** 1-2d

---

### [P2] CRSET-02C-E2E — Un-skip E1–E5 in courier-settings-detail.spec.ts `[1d]` `[x]` `[srattha]` `Done: 2026-05-18`

**Type:** FE bug + E2E
**Depends on:** CRSET-02C merged (unit tests landed)

**Scope:**
- Diagnose + fix Save-button dirty-state detection (E1 surfaced this — Save remains `disabled` after `fieldName.fill(...)`) in CourierDetailDrawer
- Address remaining skip reasons documented inline in spec (E2 pricing PUT shape, E3 toast wiring, E4 dirty-guard outside-click semantics, E5 disable toast text)
- Un-skip all 5 tests → 7/7 green

**Test Cases:** (the un-skipped E1–E5 themselves)

**Acceptance Criteria:**
- [ ] `pnpm playwright test e2e/courier-settings-detail/` — 7/7 pass, 0 skip
- [ ] Drawer dirty-state tracking covered by added unit test (regression)

**Effort estimate:** ~1d

**Spun out from CRSET-02C 2026-05-15** — un-skip work needs production-code drawer fixes; isolated to its own ticket so CRSET-02C unit-test deliverable can ship.

---

### [P3] CRSET-02D — Manual Visual Fidelity Review + Audit-Log Verify `[0.5d]` `[x]` merged 2026-05-18 PR #83

**Type:** QA / verification
**Depends on:** CRSET-02 merged

**Scope:**
- Run dev server → open detail drawer for each courier type → screenshot side-by-side vs `docs/mockups/courier-settings.html` drawer section
- Verify per `agents/fe-coder.md § Mockup Visual Fidelity` 7-point checklist: accent bar, header icon+pill, tab indicators, section dividers, footer save btn style, toggle iOS-style, audit-trail row rendering
- File missed visual items → fix or open follow-up tickets
- Verify `?reveal=true` flow: enable for admin, hit endpoint, check `AuditLogs` table has row with `action=courier.credentials.reveal`, `subjectId={courierId}`, `actorId={userId}`; if missing → implement

**Test Cases:**
- [ ] Screenshot diff vs mockup: 0 critical visual gaps
- [ ] `GET /api/couriers/{id}?reveal=true` w/ admin → audit log row created
- [ ] `GET /api/couriers/{id}?reveal=true` w/o `courier.credentials.write` → 403 (no audit row)

**Acceptance Criteria:**
- [ ] Screenshots attached to PR/ticket
- [ ] Audit-log row verified via psql query

**Effort estimate:** 0.5d

---
## GROUP — Order List Full Improve (OAD) `P1`

> **Spec:** `docs/specs/order-list-full-improve.md` v1.0
> **Mockup:** `docs/mockups/order-list-full-improve.html`
> **Effort:** BE ~11-13h · FE ~21-25h · E2E ~1.5h ≈ 32-38h total
> **Goal:** เปลี่ยน Action column ใน Order List จาก icon stub → Primary Action Button + Context Menu + Transition Dialogs ที่ทำงานได้จริง ให้ WH Workers / FF Members / WH Admins execute state transitions ได้โดยไม่ต้องเข้า Order Detail

---

### BE Tasks

#### [P1] OAD-BE-02 — POST /api/orders/{id}/transitions endpoint `[L]` `[ ]` `[wat]` `claimed: 2026-04-28`

**Goal:** Endpoint หลักสำหรับ execute state transitions — validate, persist, side effects, audit
**Type:** BE — Application + Api
**Spec Ref:** `docs/specs/order-list-full-improve.md` § WF-01, WF-03a–j, Business Rules, API Contract

**Scope:**
- `TransitionOrderCommand` + Handler — validate BR-01..BR-10
- Side effects: สร้าง `OrderParcel` (450→480), `OrderShipment` + queue `PrintJob` (480→490)
- Publish `OrderStateChangedEvent` + AuditLog entry (`order.transition`)
- Response: `OrderTransitionResultDto`
- Permission guards per transition (`orders:confirm` / `orders:pack` / `orders:admin`)
- TenantId จาก `ICurrentTenant` เสมอ — ห้ามรับจาก body (BR-05)

**Business Rules Enforced:**
- BR-01: weightKg required สำหรับ →480 → 422
- BR-02: matchStatus=unmatched บล็อก 100→400 → 409 `order_match_required`
- BR-03: invalid transition ตาม ACTION_MAP → 409 `order_state_conflict`
- BR-04: cancel terminal state (700/650/800) → 409 `order_not_cancellable`
- BR-06: cancelReason required สำหรับ →800 → 422
- BR-07: courier required สำหรับ →490 → 422
- BR-08: 490→500, 500→600 manual → `orders:admin` เท่านั้น → 403
- BR-09: scheduledPickupAt past → 422
- BR-10: cancelReason=other + ไม่มี cancelDetail → 422

**Test Cases:**
- [ ] happy path 400→450 (wh_worker) → 200, state = 450
- [ ] happy path 450→480 + weightKg=1.25 → 200, OrderParcel created
- [ ] happy path 480→490 + courier + labelSize → 200, OrderShipment + PrintJob created
- [ ] BR-01: →480 ไม่มี weightKg → 422 `validation_failed`
- [ ] BR-02: 100→400 + unmatched → 409 `order_match_required`
- [ ] BR-03: 400→480 (skip 450) → 409 `order_state_conflict`
- [ ] BR-04: 800→800 cancel terminal → 409 `order_not_cancellable`
- [ ] BR-08: ff_member tries 490→500 → 403 `insufficient_permissions`
- [ ] BR-09: scheduledPickupAt yesterday → 422
- [ ] BR-10: cancelReason=other + no detail → 422

**Dependencies:** OAD-BE-01
**Unblocks:** OAD-FE-03, OAD-BE-04

---

#### [P1] OAD-BE-04 — GET /api/orders modified — availableTransitions + operational fields `[S]` `[ ]` `[wat]` `claimed: 2026-04-28`

**Goal:** เพิ่ม fields ใน Order List response เพื่อ drive FE action column
**Type:** BE — Application + Contracts
**Spec Ref:** `docs/specs/order-list-full-improve.md` § GET /api/orders (modified)

**Scope:**
- เพิ่มใน `OrderListItemDto`: `slaDeadlineAt`, `slaMinutesRemaining`, `courier`, `trackingNumber`, `flags[]`, `matchStatus`, `assignedWorker`, `weightKg`
- `availableTransitions[]` — compute per user permission + current state (MVP: compute per request, ไม่มี cache)
- `isPrimary: true` สำหรับ transition แรกที่ user มีสิทธิ์

**Test Cases:**
- [ ] wh_worker: availableTransitions เฉพาะ pack transitions
- [ ] ff_member: availableTransitions เฉพาะ confirm transitions
- [ ] terminal state 700 → availableTransitions = []

**Dependencies:** OAD-BE-02
**Unblocks:** OAD-FE-04

---

### FE Tasks

#### [P1] OAD-FE-02 — MSW mock DB expansion + handlers `[M]` `[ ]` `[wat]` `claimed: 2026-04-28`

**Goal:** ขยาย MSW fake DB + สร้าง 3 handlers ใหม่ (transitions / print / tracking)
**Type:** FE — MSW
**Spec Ref:** `docs/specs/order-list-full-improve.md` § MSW Mock Handlers

**Scope:**
- `db.orders` เพิ่ม fields: `slaDeadlineAt`, `slaMinutesRemaining`, `flags`, `courier`, `trackingNumber`, `matchStatus`, `availableTransitions`, `weightKg`, `assignedWorker`
- `db.printJobs` + `db.shipments` collections ใหม่
- `doTransition(orderId, toState, payload)` helper — validate + apply state + recompute `availableTransitions`
- Handlers: `POST .../transitions` (200/409/422), `POST .../print` (200/409), `GET .../tracking` (200/404)

**Test Cases:**
- [ ] transition 450→480 → order.state = 480, availableTransitions recomputed
- [ ] transition unmatched 100→400 → 409 `order_match_required`
- [ ] print state 700 → 409 `print_not_allowed`
- [ ] tracking no shipment → 404 `shipment_not_found`

**Dependencies:** OAD-FE-01

---

#### [P1] OAD-FE-04 — OrderActionButton component (primary action per row) `[S]` `[ ]` `[wat]` `claimed: 2026-04-28`

**Goal:** Primary action button ใน Action column — render ตาม availableTransitions[isPrimary=true]
**Type:** FE — Component
**Spec Ref:** `docs/specs/order-list-full-improve.md` § New Components (OrderActionButton)
**Mockup Ref:** `docs/mockups/order-list-full-improve.html` — `PRIMARY_ACTION` map + `.pa-btn` styles

**Scope:**
- Props: `order: OrderListItem`, `onSuccess(): void`
- Button variant/color ตาม state (indigo/teal/purple/amber/red/slate) ตาม mockup
- Permission hidden (render `—`) ถ้าไม่มี transition ที่ user มีสิทธิ์
- Click → เปิด `OrderActionDialog` ด้วย transition ที่ตรง
- `data-testid="action-btn-{orderId}"`

**Test Cases:**
- [ ] state 450 → render primary transition button
- [ ] state 700 → render `—` (terminal, no action)
- [ ] click → calls openDialog with correct transitionDef

**Dependencies:** OAD-FE-03
**Unblocks:** OAD-FE-09

---

#### [P1] OAD-FE-05 — OrderContextMenu component (⋯ floating menu) `[M]` `[ ]` `[wat]` `claimed: 2026-04-28`

**Goal:** Context menu 3-section (actions/tools/general) เปิดจากปุ่ม ⋯ ของแต่ละ row
**Type:** FE — Component
**Spec Ref:** `docs/specs/order-list-full-improve.md` § WF-02, New Components (OrderContextMenu)
**Mockup Ref:** `docs/mockups/order-list-full-improve.html` — `.ctx-menu` + 3-section layout

**Scope:**
- 3 sections: Actions (filtered by permission) / เครื่องมือ (print/track/video) / ทั่วไป (detail/export/copy)
- Terminal states (700/650/800): ซ่อน section 1
- Manual override items (490→500, 500→600): เฉพาะ `orders:admin`
- `data-testid="context-menu-{orderId}"`, `data-testid="ctx-transition-{toState}"`
- Click outside → dismiss

**Test Cases:**
- [ ] wh_worker: เห็น pack transitions ใน section 1
- [ ] wh_admin: เห็น 490→500 (manual override item)
- [ ] terminal state 700: section 1 ไม่แสดง, section 3 แสดงปกติ
- [ ] click "Copy Order ID" → clipboard called

**Dependencies:** OAD-FE-03

---

#### [P2] OAD-FE-07 — OrderTrackingDialog (WF-05) `[S]` `[x]` `[wat]` `claimed: 2026-04-28` ✅ **merged 2026-05-14 (PR #177)**

**Goal:** Read-only tracking dialog — courier + tracking number + events timeline
**Type:** FE — Component
**Spec Ref:** `docs/specs/order-list-full-improve.md` § WF-05

**Scope:**
- `useOrderTracking(orderId, { enabled: open })`
- Courier name + tracking number + copy button + external link
- Events timeline (type, description, occurredAt, location)
- Loading skeleton + 404 fallback
- `data-testid="tracking-dialog"`, `data-testid="tracking-number-text"`, `data-testid="tracking-copy-btn"`

**Test Cases:**
- [ ] state 500 + shipment → แสดง tracking + events
- [ ] state 100 → 404 → แสดง "ยังไม่มี Tracking"
- [ ] copy button → clipboard.writeText(trackingNumber)

**Dependencies:** OAD-FE-03

---

#### [P2] OAD-FE-08 — PrintJobDialog (WF-04) `[S]` `[x]` `[wat]` `claimed: 2026-04-28` · **PR #179 merged 2026-05-14**

**Goal:** Print job dialog — format radio group → queue print job (ไม่มี state change)
**Type:** FE — Component
**Spec Ref:** `docs/specs/order-list-full-improve.md` § WF-04
**Mockup Ref:** `docs/mockups/order-list-full-improve.html` — `.print-opts` grid

**Scope:**
- jobType: 'ticket' | 'label' (จาก context menu)
- Format radio grid (a4_2up / a5 / thermal_80mm / pdf)
- `useOrderPrint` mutation
- `data-testid="radio-print-format"`

**Test Cases:**
- [ ] default format = a4_2up
- [ ] เลือก format + submit → useOrderPrint called พร้อม `{ jobType, format }`

**Dependencies:** OAD-FE-03

---

#### [P1] OAD-FE-09 — Order List table wiring — Action + SLA + Flags + Courier columns `[M]` `[ ]` `[wat]` `claimed: 2026-04-28`

**Goal:** Wire components ใหม่เข้า Order List table + ปรับ 3 columns (SLA, Flags, Courier) ตาม mockup
**Type:** FE — Component (table wiring)
**Spec Ref:** `docs/specs/order-list-full-improve.md` § Modified Components
**Mockup Ref:** `docs/mockups/order-list-full-improve.html` — `w-ac: 150px`, cell renderers

**Scope:**
- Action col (150px): แทน `ic-btn ◉` ด้วย `<OrderActionButton>` + wire `⋯` → `<OrderContextMenu>`
- SLA col: `slaMinutesRemaining` + progress bar (green/amber/red) + blink critical + label + "Closed" terminal
- Flags col: COD / EXP (blink) / UNMATCH / PARTIAL / AUTO / RTN / NC / BULK badges
- Courier col: dot color per courier + tracking number clickable → `OrderTrackingDialog`

**Test Cases:**
- [ ] slaMin=45, slaMaxMin=120 → SLA bar amber + "45m"
- [ ] state 700 → SLA "✓ Closed"
- [ ] flags=['exp','cod'] → EXP blink + COD badge
- [ ] tracking clickable → opens OrderTrackingDialog

**Dependencies:** OAD-FE-04, OAD-FE-05, OAD-FE-06, OAD-FE-07
**Unblocks:** OAD-INT-01

---

### E2E Task

#### [P1] OAD-INT-01 — E2E scaffold: order-action-dialog acceptance tests `[S]` `[👀]` `[wat]` `PR #192` `claimed: 2026-04-28` `branch: test/oad-int-01-order-action-dialog-e2e` `plan: tasks/plans/OAD-INT-01/plan.md`

**Goal:** Scaffold E2E acceptance tests (ทุก test.skip) ตาม WF-01~05 + permission scenarios
**Type:** FE — E2E
**Spec Ref:** `docs/specs/order-list-full-improve.md` § data-testid Selectors

**Test Cases to scaffold:**
- [ ] WF-01 happy path: click primary → dialog → fill → confirm → state changes
- [ ] WF-01-A: 422 → field-level error, dialog stays open
- [ ] WF-01-B: 409 → error + Refresh button
- [ ] WF-01-C: dismiss (ESC) → no API call
- [ ] WF-02: ⋯ → context menu 3 sections
- [ ] WF-02 permission: wh_worker ไม่เห็น 490→500 manual override
- [ ] WF-03d: →480 submit no weightKg → form error
- [ ] WF-03j: cancel state 490 → red warning
- [ ] WF-03j: cancelReason=other → cancelDetail required
- [ ] WF-04: print ticket → format select → submit
- [ ] WF-05: track dialog → tracking number + copy
- [ ] Permission: ff_member เห็น confirm actions ไม่เห็น pack actions

**Files:**
- `e2e/order-action-dialog/order-action-dialog.spec.ts` (scaffold — ทุก test.skip)
- `e2e/order-action-dialog/order-action-dialog.po.ts` (page object)

**Dependencies:** OAD-FE-09
**Commit:** `test(e2e): scaffold order-action-dialog acceptance tests`

---

## GROUP — Order Detail Improve (EOD) `P1`

> **Spec:** `docs/specs/spec-order-improve.md` v1.0
> **Mockup:** `docs/mockups/order-detail-improve.html`
> **Effort:** BE ~6-8h · FE ~12-15h · E2E ~1.5h ≈ 20-25h total
> **Goal:** ให้ผู้ใช้ที่มีสิทธิ์แก้ไขข้อมูล Order ที่ยังไม่ปิดผ่าน 3-tab dialog (ผู้รับ / จัดส่ง / หมายเหตุ) — ระบบ enforce lock rules ตาม state อัตโนมัติ + บันทึก Audit Log ทุก edit
> **Naming:** `EOD-BE-*` (BE), `EOD-FE-*` (FE)

---

### BE Tasks

#### [P1] EOD-BE-01 — DB Schema: Orders table — 13 new columns + indexes `[S]` `[ ]`

**Goal:** เพิ่ม columns เข้า `orders` table รองรับ edit fields ทั้งหมด
**Type:** BE — Infrastructure (Migration)
**Spec Ref:** `docs/specs/spec-order-improve.md` § Data Model

**Columns to add (nullable — A-01):**
- `recipient_name` varchar(100), `recipient_phone` varchar(20), `recipient_address` varchar(300), `recipient_province` varchar(50), `recipient_zip` char(5)
- `courier` varchar(50), `service_policy` varchar(20) default `'standard'`, `scheduled_date` date, `is_cod` boolean default false, `cod_amount` numeric(10,2)
- `tag` varchar(20) default `'normal'`, `internal_note` text, `updated_by` uuid (FK → users), `updated_at` timestamptz

**Indexes:**
- `ix_orders_tag` on (tenant_id, tag)
- `ix_orders_service_policy` on (tenant_id, service_policy)

**Files:**
- `Infrastructure/Persistence/Migrations/` (migration ใหม่)
- `Domain/Entities/Order.cs` — เพิ่ม properties ใหม่

**Test Cases:**
- [ ] migration รันสำเร็จ ไม่ break existing data
- [ ] default `is_cod=false`, `service_policy='standard'`, `tag='normal'` ถูกต้อง

**Dependencies:** None
**Unblocks:** EOD-BE-02, EOD-BE-03

---

#### [P1] EOD-BE-02 — PATCH /api/orders/{id}/edit endpoint `[M]` `[ ]`

**Goal:** Endpoint แก้ไข order — validate lock rules, persist, audit log, side effects
**Type:** BE — Application + Api
**Spec Ref:** `docs/specs/spec-order-improve.md` § WF-5, Business Rules, API Contract

**Scope:**
- `EditOrderCommand` — 3 optional sections: `recipient`, `shipping`, `meta`; PATCH semantic (fields ไม่ส่ง = ไม่แก้)
- Permission: `orders:edit` required; `wh-worker` → 403; `sys-admin` bypass lock rules
- Business Rules:
  - BR-01: immutable fields (`platform`, `shop`, `order_external_id`, `items`, `created_at`) ใน payload → 422
  - BR-02: recipient fields + st ≥ 490 + non-sysadmin → 409 `BR-02_ADDRESS_LOCKED`
  - BR-03: shipping fields + st ≥ 490 + non-sysadmin → 409 `BR-03_SHIPPING_LOCKED`
  - BR-04: courier เปลี่ยน + st = 480 → reset `tracking_no = null`, Timeline event; st ≥ 481 sysadmin only
  - BR-05: `is_cod=true` + ไม่มี `cod_amount` → 422
  - BR-06: `service_policy=scheduled` + `scheduled_date` ≤ today → 422
  - BR-07: st ∈ {700,800} + non-sysadmin → 409 `BR-07_ORDER_CLOSED`
  - BR-08: audit_log fail → rollback transaction ทั้งหมด
- Optimistic lock: ตรวจ st ตอน save → 409 `order_status_changed` ถ้าเปลี่ยนไปแล้ว
- Response: updated order + `reprint_required: bool`
- TenantId จาก `ICurrentTenant` เสมอ

**Files:**
- `Application/Orders/Commands/EditOrderCommand.cs` (ใหม่)
- `Application/Orders/Commands/EditOrderCommandHandler.cs` (ใหม่)
- `Contracts/Orders/EditOrderRequest.cs` (ใหม่)
- `Contracts/Orders/EditOrderResponse.cs` (ใหม่)
- `Api/Endpoints/OrderEditEndpoints.cs` (ใหม่)

**Test Cases:**
- [ ] happy path: แก้ recipient (st=400) → 200, DB updated, audit_log created
- [ ] happy path: แก้ meta (st=450) → 200, note/tag/internal_note updated
- [ ] BR-02: recipient field + st=490 → 409 `BR-02_ADDRESS_LOCKED`
- [ ] BR-03: shipping field + st=490 → 409 `BR-03_SHIPPING_LOCKED`
- [ ] BR-04: courier เปลี่ยน + st=480 → tracking_no=null, timeline event
- [ ] BR-05: is_cod=true + no cod_amount → 422
- [ ] BR-06: scheduled_date=yesterday → 422
- [ ] BR-07: st=700 → 409 `BR-07_ORDER_CLOSED`
- [ ] sys-admin: แก้ recipient + st=490 → 200 (bypass BR-02)
- [ ] wh-worker: PATCH → 403
- [ ] Optimistic lock: st เปลี่ยนระหว่าง client อ่านกับ save → 409 `order_status_changed`
- [ ] audit_log สร้างพร้อม diff ใน meta_json ถูกต้อง

**Dependencies:** EOD-BE-01
**Unblocks:** EOD-FE-03

---

#### [P2] EOD-BE-03 — GET /api/orders/{id} — include new edit fields in response `[XS]` `[ ]`

**Goal:** response ของ order detail ต้องมี fields ใหม่ทั้งหมด เพื่อให้ FE pre-fill dialog ได้
**Type:** BE — Application
**Spec Ref:** `docs/specs/spec-order-improve.md` § GET /api/orders/{id}

**Scope:**
- อัพเดท `OrderDetailDto` / `GetOrderDetailQuery` result — include ทุก field ใหม่ (recipient_*, courier, service_policy, scheduled_date, is_cod, cod_amount, tag, internal_note, updated_by, updated_at)

**Test Cases:**
- [ ] GET order ที่มี recipient_name set → response มี recipient object ครบ
- [ ] GET order ที่ไม่มี tag → ได้ `tag: 'normal'` (default)

**Dependencies:** EOD-BE-01

---

### FE Tasks

#### [P1] EOD-FE-02 — MSW mock handler (PATCH .../edit) `[S]` `[ ]`

**Goal:** MSW handler simulate happy path + 3 error scenarios
**Type:** FE — MSW
**Spec Ref:** `docs/specs/spec-order-improve.md` § MSW Mock Handlers

**Scope:**
- `PATCH /gateway/proxy/oms/api/orders/:id/edit`
  - 200: apply changes + return updated order (recompute `reprint_required`)
  - 409 `BR-02_ADDRESS_LOCKED`: เมื่อ order.status=490 + payload มี recipient field
  - 409 `BR-07_ORDER_CLOSED`: เมื่อ order.status=700 หรือ 800
  - 422 `validation_failed`: field validation fail (พร้อม errors array)
- อัพเดท mock order objects ใน fake DB ให้มี field ใหม่ (recipient, courier, service_policy ฯลฯ)

**Test Cases:**
- [ ] happy path st=400 + recipient payload → 200, order updated
- [ ] st=490 + recipient field → 409 `BR-02_ADDRESS_LOCKED`
- [ ] st=700 → 409 `BR-07_ORDER_CLOSED`
- [ ] missing required field → 422 พร้อม errors array

**Dependencies:** EOD-FE-01

---

#### [P1] EOD-FE-03 — useEditOrder mutation hook `[XS]` `[ ]`

**Goal:** React Query mutation hook สำหรับ PATCH edit
**Type:** FE — Hook
**Spec Ref:** `docs/specs/spec-order-improve.md` § WF-5

**Scope:**
- `useEditOrder(orderId)` — `useMutation` PATCH `.../edit`
- On success: invalidate `['order', orderId]` + `['orders']` + call `onSaved(updatedOrder)`
- On error 409 `order_status_changed`: surface message ต้อง refresh
- On error 422: return field-level errors เพื่อ set ใน RHF

**Files:**
- `features/orders/hooks/useEditOrder.ts` (ใหม่)

**Test Cases:**
- [ ] success → invalidates `['order', id]` + `['orders']`
- [ ] 409 order_status_changed → error message set
- [ ] 422 → errors object ส่งกลับมาให้ form

**Dependencies:** EOD-FE-01, EOD-FE-02
**Unblocks:** EOD-FE-04

---

#### [P1] EOD-FE-06 — i18n keys (~30 keys) `[XS]` `[ ]`

**Goal:** เพิ่ม i18n keys ครบทุก key ใน spec ทั้ง en.json + th.json
**Type:** FE — i18n
**Spec Ref:** `docs/specs/spec-order-improve.md` § i18n Keys

**Keys to add (namespace `order.edit.*`):**
- title, tabs (3), recipient fields (5), lock messages (3), shipping fields (7+policy options), meta fields (3+audit), actions (save/cancel/footer), toast success, error messages (2), tag labels (6)

**Test Cases:**
- [ ] ไม่มี raw key แสดงบน UI ใน dialog

**Dependencies:** None
**Unblocks:** EOD-FE-04

---

### Task Summary — Edit Order Dialog

| Task | Role | Effort | Depends On |
|------|------|--------|------------|
| EOD-BE-01 | BE | S · 1-2h | — |
| EOD-BE-02 | BE | M · 4-5h | EOD-BE-01 |
| EOD-BE-03 | BE | XS · 1h | EOD-BE-01 |
| EOD-FE-02 | FE | S · 1.5h | (EOD-FE-01 ✅) |
| EOD-FE-03 | FE | XS · 1h | EOD-FE-02 |
| EOD-FE-06 | FE | XS · 0.5h | — |
| **Total (remaining)** | | **~10-12h** | |

> **Done:** EOD-FE-01 ✅ · EOD-FE-04 ✅ · EOD-FE-05 ✅ · EOD-INT-01 ✅
> **Critical path (FE remaining):** EOD-FE-02 → EOD-FE-03

---

## GROUP — On-demand Sync Orders `P1`

> **Spec BE:** `docs/specs/ondemand_sync_orders_backend.md` v1.2 ✅ Ready for Build
> **Spec FE:** `docs/specs/ondemand_sync_orders_frontend.md` v1.2 ✅ Ready for Build
> **Mockup:** `docs/mockups/ondemand_sync_orders.html`
> **Effort:** BE ~6-8h · FE ~8-10h · E2E ~1.5h ≈ 16-20h total
> **Goal:** เพิ่มปุ่ม Sync Order ใน Order List — UI trigger webhook per marketplace, รับรู้แค่ WEBHOOK_ACCEPTED (Flow 1 only)
> **Naming:** `ODS-BE-*` (BE) · `ODS-FE-*` (FE) · `ODS-E2E-*` (E2E)

---

### BE Tasks

#### [P1] ODS-BE-01 — Domain: OndemandSyncOrderRequest entity + enums `[XS]` `[ ]`

**Goal:** สร้าง Domain layer ครบ — entity + domain methods + enums
**Type:** BE — Domain
**Spec Ref:** `docs/specs/ondemand_sync_orders_backend.md` § State Machine, § Data Model

**Files to create:**
- `Domain/Orders/OndemandSync/OndemandSyncOrderRequest.cs` — Aggregate Root: `Create(...)`, `MarkCallingWebhook()`, `MarkWebhookAccepted(requestId, statusCode, message)`, `MarkWebhookFailed(errorCode, message)` — invalid transition → `InvalidOperationException`
- `Domain/Orders/OndemandSync/OndemandSyncOrderStatus.cs` — enum: `REQUESTED`, `CALLING_WEBHOOK`, `WEBHOOK_ACCEPTED`, `WEBHOOK_FAILED`
- `Domain/Orders/OndemandSync/MarketplacePlatform.cs` — enum: `LAZADA`, `SHOPEE`, `TIKTOK`
- `Domain/Orders/OndemandSync/SyncMode.cs` — enum: `ON_DEMAND`

**Test Cases:**
- [ ] `Create(...)` → status = REQUESTED, id ≠ null, created_at set
- [ ] `MarkCallingWebhook()` เมื่อ REQUESTED → CALLING_WEBHOOK
- [ ] `MarkCallingWebhook()` เมื่อ CALLING_WEBHOOK → throw InvalidOperationException
- [ ] `MarkWebhookAccepted(...)` → WEBHOOK_ACCEPTED, completed_at set
- [ ] `MarkWebhookFailed(...)` → WEBHOOK_FAILED, error_code set, completed_at set
- [ ] `MarkWebhookAccepted()` จาก WEBHOOK_FAILED → throw (terminal state)

**Commit:** `feat(domain): add OndemandSyncOrderRequest aggregate`

---

#### [P1] ODS-BE-02 — Application: Command + Handler + Validator + Interfaces `[M]` `[ ]`

**Goal:** Application layer ครบ — command/handler/validator/interfaces
**Type:** BE — Application
**Spec Ref:** `docs/specs/ondemand_sync_orders_backend.md` § API Contract, § Business Rules
**Depends on:** ODS-BE-01

**Files to create:**
- `Application/Orders/OndemandSync/Commands/OndemandSyncOrdersCommand.cs` — record `{ Request, IdempotencyKey, CorrelationId, TenantId, RequestedBy }`
- `Application/Orders/OndemandSync/Commands/OndemandSyncOrdersHandler.cs` — `IRequestHandler<Command, ApplicationResult<OndemandSyncOrdersResponse>>`
  - ตรวจ idempotency key → return existing ถ้าซ้ำ (`isDuplicate: true`)
  - resolve webhook client ผ่าน `IMarketplaceSyncWebhookClientFactory`
  - state transitions: REQUESTED → CALLING_WEBHOOK → WEBHOOK_ACCEPTED / WEBHOOK_FAILED
  - structured log ทุก event (ดู Logging Spec ใน spec)
- `Application/Orders/OndemandSync/Dtos/OndemandSyncOrdersRequest.cs`
- `Application/Orders/OndemandSync/Dtos/OndemandSyncOrdersResponse.cs` — รวม `isDuplicate: bool`
- `Application/Orders/OndemandSync/Dtos/WebhookSyncOrdersRequest.cs`
- `Application/Orders/OndemandSync/Dtos/WebhookSyncOrdersResponse.cs`
- `Application/Orders/OndemandSync/Validators/OndemandSyncOrdersRequestValidator.cs` — FluentValidation ครบ BR-04 ถึง BR-07
- `Application/Orders/OndemandSync/Services/IOndemandSyncOrderRequestRepository.cs` — `FindByIdempotencyKeyAsync`, `InsertAsync`, `UpdateAsync` (ทุก method มี `CancellationToken ct`)
- `Application/Orders/OndemandSync/Services/IMarketplaceSyncWebhookClient.cs` — `string Endpoint`, `SyncOrdersAsync(..., CancellationToken ct)`
- `Application/Orders/OndemandSync/Services/IMarketplaceSyncWebhookClientFactory.cs` — `Create(MarketplacePlatform)`

**Test Cases:**
- [ ] Validator: `platform` = "TOKOPEDIA" → 422 ORD-SYNC-4220
- [ ] Validator: `toDate` < `fromDate` → 422 ORD-SYNC-4221
- [ ] Validator: `pageSize` = 501 → 422 ORD-SYNC-4222
- [ ] Validator: `syncMode` = "SCHEDULED" → 422 ORD-SYNC-4223
- [ ] Validator: all valid (shopId omitted) → passes
- [ ] Handler: idempotency key ซ้ำ → return existing, `SyncOrdersAsync` ไม่ถูกเรียก
- [ ] Handler: idempotency key ใหม่ → `SyncOrdersAsync` ถูกเรียก 1 ครั้ง

**Commit:** `feat(application): add OndemandSyncOrders command handler and validator`

---

#### [P1] ODS-BE-03 — Infrastructure: Webhook clients + Repository + EF Core config + DI `[M]` `[ ]`

**Goal:** Infrastructure layer ครบ — 3 webhook clients, repository, EF config, DI registration
**Type:** BE — Infrastructure
**Spec Ref:** `docs/specs/ondemand_sync_orders_backend.md` § Platform Routing, § Data Model § EF Core Configuration, § File Plan
**Depends on:** ODS-BE-02

**Files to create:**
- `Infrastructure/WebhookClients/LazadaSyncOrdersWebhookClient.cs` — Named HttpClient `"WebhookClient"` via `IHttpClientFactory`, endpoint `/api/v1/lazada/sync/orders`, set `X-Correlation-Id` header
- `Infrastructure/WebhookClients/ShopeeSyncOrdersWebhookClient.cs` — endpoint `/api/v1/shopee/sync/orders`
- `Infrastructure/WebhookClients/TiktokSyncOrdersWebhookClient.cs` — endpoint `/api/v1/tiktok/sync/orders`
- `Infrastructure/WebhookClients/MarketplaceSyncWebhookClientFactory.cs` — inject 3 clients, switch on platform
- `Infrastructure/Persistence/Repositories/OndemandSyncOrderRequestRepository.cs` — inject `AppDbContext`
- Migration: `dotnet ef migrations add AddOndemandSyncOrderRequests -p src/Infrastructure -s src/Api`

**Files to modify:**
- `Infrastructure/DependencyInjection.cs` — เพิ่มใน `OmsFulfillmentInfrastructure()`:
  ```csharp
  services.AddHttpClient("WebhookClient")
      .ConfigureHttpClient(c => c.Timeout = TimeSpan.FromSeconds(
          configuration.GetValue("Webhook:TimeoutSeconds", 30)));
  services.AddScoped<IOndemandSyncOrderRequestRepository, OndemandSyncOrderRequestRepository>();
  services.AddScoped<IMarketplaceSyncWebhookClientFactory, MarketplaceSyncWebhookClientFactory>();
  services.AddScoped<LazadaSyncOrdersWebhookClient>();
  services.AddScoped<ShopeeSyncOrdersWebhookClient>();
  services.AddScoped<TiktokSyncOrdersWebhookClient>();
  ```
- `Infrastructure/Persistence/AppDbContext.cs` — เพิ่ม `DbSet<OndemandSyncOrderRequest>` + inline config ใน `OnModelCreating` (ดู spec § EF Core Configuration)

**Test Cases (WebhookClientFactory unit):**
- [ ] `Create(LAZADA)` → typeof LazadaSyncOrdersWebhookClient
- [ ] `Create(SHOPEE)` → typeof ShopeeSyncOrdersWebhookClient
- [ ] `Create(TIKTOK)` → typeof TiktokSyncOrdersWebhookClient
- [ ] `Create(unknown)` → throw ArgumentOutOfRangeException

**Commit:** `feat(infra): add webhook clients, repository, EF config for ondemand sync`

---

#### [P1] ODS-BE-04 — API Endpoint + appsettings `[S]` `[ ]`

**Goal:** Minimal API endpoint + config keys
**Type:** BE — Api
**Spec Ref:** `docs/specs/ondemand_sync_orders_backend.md` § API Contract
**Depends on:** ODS-BE-03

**Files to create:**
- `Api/Endpoints/OndemandSyncOrdersEndpoints.cs`:
  - `app.MapPost("/api/oms/orders/ondemand-sync", ...)`
  - extract `Idempotency-Key` (→ 400 `ORD-SYNC-4000` ถ้าหาย) + `X-Correlation-Id` จาก headers
  - inject `ICurrentTenant` (`B1dx.OmsFulfillment.Application.Contracts`) → `TenantId`
  - inject `ICurrentUser` → `UserId` เป็น `RequestedBy`
  - send `OndemandSyncOrdersCommand` ผ่าน Mediator
  - map `ApplicationResult<T>` → `TypedResults.Ok(result.Value)` หรือ `TypedResults.Problem(...)`
  - permission guard: `orders:sync`

**Files to modify:**
- `appsettings.json` — เพิ่ม:
  ```json
  "Webhook": {
    "BaseUrl": "https://webhook.benzapps.com",
    "TimeoutSeconds": 30
  }
  ```

**Test Cases (Integration):**
- [ ] Happy path, webhook mock 200 → HTTP 200, `status: WEBHOOK_ACCEPTED`, record ใน DB
- [ ] Webhook mock 500 → HTTP 502 `ORD-SYNC-5002`, `status: WEBHOOK_FAILED` ใน DB
- [ ] Idempotency key ซ้ำ → HTTP 200, DB record count ไม่เพิ่ม
- [ ] ไม่มี `Idempotency-Key` header → HTTP 400 `ORD-SYNC-4000`
- [ ] ขาด permission `orders:sync` → HTTP 403 `ORD-SYNC-4030`
- [ ] `platform` ผิด → HTTP 422 `ORD-SYNC-4220`
- [ ] `toDate` < `fromDate` → HTTP 422 `ORD-SYNC-4221`

**Commit:** `feat(api): add POST /api/oms/orders/ondemand-sync endpoint`

---

### FE Tasks

#### [P1] ODS-FE-00 — Gateway upstream `oms` + apiRoutes helper `[XS]` `[ ]`

**Goal:** เพิ่ม upstream `oms` ใน gateway proxy และ `oms()` helper ใน apiRoutes
**Type:** FE — Infrastructure
**Spec Ref:** `docs/specs/ondemand_sync_orders_frontend.md` § Gateway Upstream — Required Setup

**Files to modify:**
- `src/app/gateway/proxy/[upstreamKey]/[...path]/route.ts` — เพิ่ม `oms` upstream:
  ```typescript
  const OMS_API_BASE_URL = process.env.OMS_API_BASE_URL ?? "";
  // ใน upstreams:
  oms: {
    baseUrl: OMS_API_BASE_URL,
    allowedPathPrefixes: ["/api/oms"],
    forwardCookies: true,
  },
  ```
- `src/lib/api/apiRoutes.ts` — เพิ่ม:
  ```typescript
  export const oms = (path: string) => `${GATEWAY_PROXY_BASE}/oms${path}`;
  ```
- `.env.local` / `.env.example` — เพิ่ม `OMS_API_BASE_URL=http://localhost:5000`

**Test Cases:**
- [ ] `oms('/api/oms/orders/ondemand-sync')` returns `/gateway/proxy/oms/api/oms/orders/ondemand-sync`

**Commit:** `feat(gateway): add oms upstream and apiRoutes helper`

---

#### [P1] ODS-FE-01 — Types + Zod schema + date-range util `[XS]` `[ ]`

**Goal:** TypeScript types, Zod validation schema, date range calculator
**Type:** FE — Foundation
**Spec Ref:** `docs/specs/ondemand_sync_orders_frontend.md` § TypeScript Interfaces, § Range → Date Mapping
**Depends on:** ODS-FE-00

**Files to create:**
- `src/features/orders/types/ondemandSync.ts` — ทุก interface/type จาก spec (MarketplacePlatform, SyncRangeKey, WebhookHealthStatus, SyncJobStatus, SyncJob, OndemandSyncResponse, ฯลฯ)
- `src/features/orders/utils/syncDateRange.ts` — `computeDateRange(range: SyncRangeKey): { fromDate: string; toDate: string }` ใช้ `date-fns` + `formatISO()`

**Test Cases:**
- [ ] `computeDateRange('1h')` → fromDate = now-1h, toDate = now (ISO 8601 with offset)
- [ ] `computeDateRange('today')` → fromDate = startOfDay, toDate = now
- [ ] `computeDateRange('yesterday')` → fromDate = startOfDay(yesterday), toDate = endOfDay(yesterday)
- [ ] output format มี timezone offset (+07:00) ไม่ใช่ Z

**Commit:** `feat(orders): add ondemand sync types, zod schema, date range util`

---

#### [P1] ODS-FE-02 — MSW mock handlers `[XS]` `[ ]`

**Goal:** Mock handlers ครบทุก scenario ก่อน build components
**Type:** FE — Mock
**Spec Ref:** `docs/specs/ondemand_sync_orders_frontend.md` § MSW Mock Handlers
**Depends on:** ODS-FE-01

**Files to create:**
- `src/mocks/handlers/ondemandSync.ts`:
  - `POST /gateway/proxy/oms/api/oms/orders/ondemand-sync` → 200 WEBHOOK_ACCEPTED (default)
  - TIKTOK → 502 WEBHOOK_FAILED
  - ไม่มี `Idempotency-Key` → 400 `ORD-SYNC-4000`
  - mock flag no-permission → 403
- `GET https://webhook.benzapps.com/health` → `{ status: 'ok' }` (default) + slow/error variants

**Commit:** `test(mocks): add ondemand sync MSW handlers`

---

#### [P1] ODS-FE-03 — E2E test scaffold `[XS]` `[ ]`

**Goal:** scaffold Playwright acceptance tests (test.skip) ก่อน implement
**Type:** FE — E2E Test
**Spec Ref:** `docs/specs/ondemand_sync_orders_frontend.md` § E2E Tests
**Depends on:** ODS-FE-02

**Files to create:**
- `e2e/ondemand-sync/ondemand-sync.spec.ts` — test.skip ทุกตัว:
  - Happy path: Shopee+Lazada 1 ชม. → Sync → modal ปิด + banner accepted
  - ไม่เลือก marketplace → ปุ่ม disabled ตลอด
  - TikTok 502 → banner partial (Shopee=accepted, TikTok=failed)
  - health mock degraded → badge SLOW amber
  - health mock error → badge ERROR red
  - dismiss banner → banner ซ่อน
  - `user.tenantId` null → toast error, ไม่ submit

**Commit:** `test(e2e): scaffold ondemand-sync acceptance tests`

---

#### [P1] ODS-FE-04 — Hook: `useOndemandSync` `[M]` `[ ]`

**Goal:** Hook จัดการ state ทั้งหมด — modal, health check, API call, job tracking
**Type:** FE — Hook
**Spec Ref:** `docs/specs/ondemand_sync_orders_frontend.md` § Hook: useOndemandSync
**Depends on:** ODS-FE-01, ODS-FE-02

**Files to create:**
- `src/features/orders/hooks/useOndemandSync.ts`:
  - `openModal()` → `isModalOpen: true` + fire health check (`AbortSignal.timeout(5000)`)
  - `submitSync()` → validate `user.tenantId` → `Promise.all` API calls → map results → `overallStatus`
  - error handling: `ApiRequestError` + `TypeError` (network)
  - validate response ด้วย `ondemandSyncResponseSchema.safeParse()`

**Test Cases (hook unit):**
- [ ] `openModal()` → health check fetch fired
- [ ] health `{ status: 'ok' }` → `webhookHealth: 'online'`
- [ ] health timeout > 5s → `webhookHealth: 'error'`
- [ ] `submitSync()` เลือก Shopee+Lazada → API เรียก 2 ครั้ง parallel, Idempotency-Key ต่างกัน
- [ ] ทุก platform accepted → `overallStatus: 'accepted'`
- [ ] ทุก platform failed → `overallStatus: 'failed'`
- [ ] Shopee accepted + Lazada failed → `overallStatus: 'partial'`
- [ ] Lazada throws TypeError → `errorMessage: 'Network error...'`
- [ ] `user.tenantId` = null → ไม่ call API

**Commit:** `feat(orders): add useOndemandSync hook`

---

#### [P1] ODS-FE-05 — Components: SyncOrderButton + WebhookHealthBadge `[S]` `[ ]`

**Goal:** 2 atomic components ที่ไม่ depend on hook
**Type:** FE — Component
**Spec Ref:** `docs/specs/ondemand_sync_orders_frontend.md` § Component Visual Spec
**Depends on:** ODS-FE-01

**Files to create:**
- `src/features/orders/components/SyncOrderButton.tsx` — teal outline, spinning icon, hidden ถ้าขาด `orders:sync`
- `src/features/orders/components/WebhookHealthBadge.tsx` — loading/online/slow/error badge + `lastSyncLabel`

**Test Cases:**
- [ ] SyncOrderButton: render ถ้ามี permission
- [ ] SyncOrderButton: ไม่ render ถ้าขาด permission
- [ ] SyncOrderButton: spinning เมื่อ `isSpinning=true`
- [ ] WebhookHealthBadge: green pill เมื่อ online
- [ ] WebhookHealthBadge: amber เมื่อ slow
- [ ] WebhookHealthBadge: red เมื่อ error
- [ ] WebhookHealthBadge: spinner เมื่อ loading

**Commit:** `feat(orders): add SyncOrderButton and WebhookHealthBadge components`

---

#### [P1] ODS-FE-06 — Component: SyncOrderModal `[M]` `[ ]`

**Goal:** Dialog component ครบ — marketplace checkboxes, range chips, info/warning box
**Type:** FE — Component
**Spec Ref:** `docs/specs/ondemand_sync_orders_frontend.md` § SyncOrderModal
**Depends on:** ODS-FE-04, ODS-FE-05

**Files to create:**
- `src/features/orders/components/SyncOrderModal.tsx`:
  - Header: gradient bg, title, subtitle badges, ✕ button
  - Section 1: 3 marketplace rows (checkbox + icon + WebhookHealthBadge + lastSyncLabel)
  - Section 2: 5 range chips (default "1h" active)
  - Info box (indigo) / Warning box (amber) — reactive
  - Footer: ghost Cancel + teal Confirm (disabled ถ้า selectedPlatforms.length === 0)

**Test Cases:**
- [ ] render เมื่อ `isOpen: true` → header, 3 checkboxes, 5 chips, info box, footer
- [ ] ไม่เลือก marketplace → ปุ่ม disabled + warning box
- [ ] เลือก Lazada เท่านั้น → info box มี "Lazada"
- [ ] เปลี่ยน range "7d" → info box มี "7 วัน"
- [ ] `webhookHealth: 'error'` → badge red ERROR
- [ ] กด ✕ / ยกเลิก → `onClose()` called
- [ ] กด เริ่ม Sync → `onConfirm(platforms, range)` called

**Commit:** `feat(orders): add SyncOrderModal component`

---

#### [P1] ODS-FE-07 — Component: SyncJobBanner `[S]` `[ ]`

**Goal:** Banner แสดงสถานะ job — indeterminate progress, per-platform results, dismiss
**Type:** FE — Component
**Spec Ref:** `docs/specs/ondemand_sync_orders_frontend.md` § SyncJobBanner
**Depends on:** ODS-FE-04

**Files to create:**
- `src/features/orders/components/SyncJobBanner.tsx`:
  - Hidden เมื่อ `job === null`
  - Platform icons + job ID (mono) + indeterminate progress bar (processing) / 100% (terminal)
  - Status badge: processing=teal / accepted=green / partial=amber / failed=red
  - Per-platform result rows เมื่อ partial/failed
  - Dismiss button เมื่อ overallStatus ≠ 'processing'

**Test Cases:**
- [ ] processing → progress indeterminate, ไม่มี dismiss
- [ ] accepted → green badge, dismiss แสดง
- [ ] partial → amber badge, per-platform rows
- [ ] failed → red badge, dismiss แสดง
- [ ] กด dismiss → `onDismiss()` called

**Commit:** `feat(orders): add SyncJobBanner component`

---

#### [P1] ODS-FE-08 — Integration: เพิ่ม SyncOrderButton + SyncJobBanner ใน OrderAll `[S]` `[ ]`

**Goal:** wire ทุก component เข้า Order List page
**Type:** FE — Integration
**Spec Ref:** `docs/specs/ondemand_sync_orders_frontend.md` § File Locations
**Depends on:** ODS-FE-05, ODS-FE-06, ODS-FE-07

**Files to modify:**
- `src/app/(app)/orders/page.tsx` หรือ `OrderAll` component:
  - เพิ่ม `useOndemandSync()` hook
  - เพิ่ม `<SyncOrderButton>` ใน topbar (ก่อนปุ่ม "+ สร้าง Order")
  - เพิ่ม `<SyncOrderModal>` (controlled by `isModalOpen`)
  - เพิ่ม `<SyncJobBanner>` ใต้ State Machine bar เหนือ Filter bar (inline, conditional render)

**Test Cases:**
- [ ] SyncOrderButton แสดงใน topbar ถ้ามี `orders:sync`
- [ ] กดปุ่ม → modal เปิด
- [ ] confirm → modal ปิด, banner แสดง

**Commit:** `feat(orders): wire sync order components into OrderAll page`

---

#### [P1] ODS-E2E-01 — E2E: un-skip + run acceptance tests `[S]` `[ ]`

**Goal:** un-skip ทุก test ใน scaffold + ผ่านทั้งหมด
**Type:** FE — E2E
**Depends on:** ODS-FE-08, ODS-FE-03

**Files to modify:**
- `e2e/ondemand-sync/ondemand-sync.spec.ts` — remove all `test.skip`

**Test Cases:**
- [ ] Happy path ผ่าน
- [ ] Partial failure ผ่าน
- [ ] Health badge states ผ่าน
- [ ] Dismiss banner ผ่าน
- [ ] Permission guard ผ่าน

**Commit:** `test(e2e): enable ondemand-sync acceptance tests`

---

### Task Summary — On-demand Sync Orders

| Task | Role | Effort | Depends On |
|------|------|--------|------------|
| ODS-BE-01 | BE | XS · 1h | — |
| ODS-BE-02 | BE | M · 3-4h | ODS-BE-01 |
| ODS-BE-03 | BE | M · 3-4h | ODS-BE-02 |
| ODS-BE-04 | BE | S · 1-2h | ODS-BE-03 |
| ODS-FE-00 | FE | XS · 0.5h | — |
| ODS-FE-01 | FE | XS · 0.5h | ODS-FE-00 |
| ODS-FE-02 | FE | XS · 0.5h | ODS-FE-01 |
| ODS-FE-03 | FE | XS · 0.5h | ODS-FE-02 |
| ODS-FE-04 | FE | M · 2-3h | ODS-FE-01, ODS-FE-02 |
| ODS-FE-05 | FE | S · 1-1.5h | ODS-FE-01 |
| ODS-FE-06 | FE | M · 2-3h | ODS-FE-04, ODS-FE-05 |
| ODS-FE-07 | FE | S · 1-1.5h | ODS-FE-04 |
| ODS-FE-08 | FE | S · 1h | ODS-FE-05,06,07 |
| ODS-E2E-01 | FE | S · 1h | ODS-FE-08, ODS-FE-03 |
| **Total** | | **~17-21h** | |

**Critical path BE:** ODS-BE-01 → ODS-BE-02 → ODS-BE-03 → ODS-BE-04
**Critical path FE:** ODS-FE-00 → ODS-FE-01 → ODS-FE-04 → ODS-FE-06 → ODS-FE-08 → ODS-E2E-01

---

## GROUP — PIM (Product Master Data) `P2`

**Mockup:** [`docs/mockups/pim/beone_pim_crud_mock.html`](../docs/mockups/pim/beone_pim_crud_mock.html) (v7 Variant Pro, 4759 lines)
**Spec entry:** [`docs/specs/pim.md`](../docs/specs/pim.md) — index → sub-specs [`product-master-overview`](../docs/specs/product-master-overview.md), [`-domain`](../docs/specs/product-master-domain.md), [`-api`](../docs/specs/product-master-api.md), [`-readiness`](../docs/specs/product-master-readiness.md), [`-pricing`](../docs/specs/product-master-pricing.md), [`-channel-sync`](../docs/specs/product-master-channel-sync.md)
**Foundation:** PIM-FE-00 scaffold ✅ merged 2026-05-12; PIM-FE-01 spec consolidation ✅ merged 2026-05-12
**Theme:** OMS indigo `--ind` 100% — ห้ามใช้ purple PIM ตาม mockup ต้นทาง
**Permission keys:** `product_master.{read,manage,publish,sync}` — ห้ามใช้ `pim.*`
**Status (2026-05-17):** 14/15 PIM tickets done ✅ — remaining: **PIM-BE-01c** (BE Pricing/Channels/Audit + event dispatch) + 1 P2 bug (`TEST-PIM-DRIFT-AUDIT-PROP` — audit tab not wired with productId). PIM-E2E-01 completed today (34/37 pass, 3 audit fixmes block on the prop bug).

> All FE tabs done → PIM-E2E-01 ✅ done. BE-01a+01b done → BE-01c unblocked.

---

### [x] PIM-FE-00 — Module Scaffold (Product List + Detail + Overview tab) ✓ pushed 2026-05-12

**Type:** `[FE]` | **Effort:** M ~4h | **Owner:** Nuchit
**Branch:** `feat/product_master_data` — commits FE `e9067ec`, Workspace `7ce5ca5`
**Plan:** [`tasks/plans/PIM-FE-00/plan.md`](plans/PIM-FE-00/plan.md)
**Scope (done):**
- features/pim/ 4-layer (types/mocks/ViewModels/containers/views)
- Pages: /pim/{products,products/[id],categories,brands,attributes}
- AppShell PIM nav group + breadcrumb + i18n EN/TH (5 keys)
- ProductListView (stats bar + table) + ProductDetailView (7 tabs + readiness panel)
- OverviewTab complete + 6 placeholder tabs
- 26/26 vitest pass + next lint clean + typecheck PIM-only clean

---

### [x] PIM-FE-01 — Spec consolidation + gap-fill ✓ merged 2026-05-12

**Type:** `[Docs]` | **Effort:** S ~1h | **Owner:** Nuchit
**PR:** [#5](https://github.com/B1DXDev/b1dx-fulfillment-workspace/pull/5) — squash `2323b45`
**Plan:** [`tasks/plans/PIM-FE-01/plan.md`](plans/PIM-FE-01/plan.md) (Status: Done)

**Scope:**

- Patch [`product-master-domain.md`](../docs/specs/product-master-domain.md) — + **Attribute** entity, per-category attribute bindings, variant attribute values, supporting tables/indexes/events/tests (v1.1 → v1.2)
- Patch [`product-master-api.md`](../docs/specs/product-master-api.md) — + **Categories / Brands / Attributes** CRUD endpoints + DTOs + query params + error codes (v1.2 → v1.3)
- Patch [`product-master-overview.md`](../docs/specs/product-master-overview.md) — + § 11 PIM Module Mapping (PIM-FE-XX ↔ spec section) + rollout sequence ticket IDs (v1.1 → v1.2)
- Create [`docs/specs/pim.md`](../docs/specs/pim.md) — thin index page (≤80 lines) pointing to 7 sub-specs + permission table + ticket→spec mapping
- Align permission keys → `product_master.{read,manage,publish,sync}` (drop `pim.*`)
- Update `tasks/backlog.md` + `tasks/current-sprint.md` spec links + permission keys

**Test Cases:** n/a (docs task — verify checklist 7 ข้อใน plan)

---

### [P1] PIM-SPEC-02 — Spec 100% coverage of PIM mockups 🔄 **claimed: 2026-05-13 (nuchit)**

**Type:** `[Docs]` | **Effort:** M ~3-5h | **Depends on:** PIM-FE-01 ✅
**Branch:** `docs/pim-spec-02-coverage-100` (workspace repo)
**Plan:** [`tasks/plans/PIM-SPEC-02/plan.md`](plans/PIM-SPEC-02/plan.md) (Status: Confirmed)
**Mockup ref:** beone_pim_crud_mock.html (4759 lines) + beone_product_master_complete.html (4592 lines)

**Scope (close 41 gaps from 2026-05-13 coverage audit):**

- 5 top gaps: bulk formula editor, view modes (table/matrix/gallery), drift detection + resync, product flags, CSV/Excel import wizard
- 23 partial coverage items (multi-level barcode, SKU pattern algorithm, excluded combos rules, per-variant marketplace mapping, price history, etc.)
- 18 missing items (variant expand drawer, field lock toggle, inline cell editing, Cmd+K palette, sync banner, provenance chips, etc.)

**Files modified:**

- `docs/specs/product-master-{overview,domain,api,readiness,pricing,channel-sync}.md` (6 files)
- `docs/specs/pim.md` — index update

**Out of scope:** FE/BE code (เป็น future tickets), `product-master-marketplace-download.md` (out-of-MVP), detailed CSS/component UX specs

**Test Cases:** n/a (docs task) — verify gates:

- [ ] Coverage re-audit by Explore agent ≥98%
- [ ] Markdown lint clean (`markdownlint docs/specs/*.md`)
- [ ] All internal links resolve
- [ ] Mockup line citations correct (sample 5)
- [ ] Existing PIM tickets' test cases unchanged unless spec changed
- [ ] BE-01 entry updated with split note (BE-01a/01b/01c)

---

### [P2] PIM-BE-01 — Product aggregate + EF migration + endpoints 🔄 **claimed: 2026-05-13 (nuchit) — Wave 2 (waiting SPEC-02 merge)**

**Type:** `[BE]` | **Effort:** L ~5-8h (split → BE-01a/01b/01c after SPEC-02 merge) | **Depends on:** PIM-FE-01 ✅, PIM-SPEC-02
**Branch:** `feat/pim-be-01-product-aggregate`
**Plan:** [`tasks/plans/PIM-BE-01/plan.md`](plans/PIM-BE-01/plan.md) (Status: Confirmed — waiting SPEC-02 merge)
**Scope:**
- Domain: `Product`, `Variant`, `ProductStatus` value object + domain events
- Application: `CreateProduct`, `UpdateProduct`, `ListProducts`, `GetProductDetail` (Mediator)
- Infrastructure: EF Core entity config + migration **snake_case** (per `feedback_db_naming_convention.md`)
- Api: endpoints + permission guards + integration tests

**Test Cases:**
- [ ] Unit: status transition rules (draft→pending allowed, archived→active blocked, ...)
- [ ] Unit: tenant isolation (ICurrentTenant)
- [ ] Integration: list + filter + pagination + tenant scope
- [ ] Integration: create + update + permission denied path

---

### [P2] PIM-FE-02 — Variants tab (axes builder + matrix + table view) ✅ **merged 2026-05-13 (nuchit)**

**Type:** `[FE]` | **Effort:** L ~6-8h | **Depends on:** PIM-FE-00 ✅, PIM-FE-01 ✅
**Branch:** `feat/pim-fe-02-variants-tab` (FE repo, deleted post-merge)
**PR:** [#167](https://github.com/B1DXDev/b1dx-oms-fulfillment-web/pull/167) — squash `0e975cc`
**Plan:** [`tasks/plans/PIM-FE-02/plan.md`](plans/PIM-FE-02/plan.md) (Status: Done)
**Mockup ref:** beone_pim_crud_mock.html lines 1709-1900+ (axes-builder, sku-pattern, var-toolbar, var-bulk-bar, var-table)
**Scope:**
- axes builder + value chips + SKU pattern editor + matrix preview
- 3 view modes (table / matrix / gallery) + view toggle
- bulk-edit toolbar (fill-down, formula `+10%`/`-50`)

**Out of scope:** CSV/Excel TSV paste parser (defer FE-02b), MSW handlers (defer until BE-01), drag-reorder axes

**Test Cases:**
- [ ] Generate axes → SKU matrix render ครบทุก combo
- [ ] Exclude combo → SKU ที่ exclude ไม่ render
- [ ] Fill-down → ทุก row selected รับค่าจาก row แรก
- [ ] Formula `+10%` → คำนวณถูกต้อง
- [ ] View toggle table ↔ matrix ↔ gallery — UI สลับ
- [ ] SKU pattern editor → preview updates real-time

---

### [P2] PIM-FE-03 — Pricing tab ✅ **merged 2026-05-13 (direct push)**

**Type:** `[FE]` | **Effort:** M ~3-4h | **Depends on:** PIM-FE-00 ✅
**Branch:** `feat/pim-fe-03-pricing-tab-impl`
**Plan:** [`tasks/plans/PIM-FE-03/plan.md`](plans/PIM-FE-03/plan.md) (Status: Done)
**Scope:** tier pricing + currency overrides per channel + promo rules display + price history + compare-at/markup
**Follow-up:** `FIX-PIM-FE-03-FOOTER-BUTTONS` (P3) — Save/Price history buttons missing

**Test Cases:**
- [x] เพิ่ม tier → render row ใหม่
- [x] channel override → ค่า override priority สูงกว่า base price

---

### [P2] PIM-FE-04 — Media tab ✅ **merged 2026-05-15 (i18n retrofit PR #187)**

**Type:** `[FE]` | **Effort:** M ~3-4h | **Depends on:** PIM-FE-00 ✅
**Branch:** `feat/pim-fe-04-media-tab` + `refactor/pim-fe-04-i18n`
**Plan:** [`tasks/plans/PIM-FE-04/plan.md`](plans/PIM-FE-04/plan.md) (Status: Done)
**Scope:** image gallery + alt text + per-variant inherit/override toggle + drag-to-reorder

**Test Cases:**
- [x] Upload image → preview thumbnail
- [x] Drag reorder → order array update
- [x] Per-variant override toggle → inherit/local switch

---

### [P3] PIM-FE-05 — Logistics tab ✅ **merged 2026-05-16 (PR #193)**

**Type:** `[FE]` | **Effort:** S ~1-2h | **Depends on:** PIM-FE-00 ✅
**Branch:** `feat/pim-fe-05-logistics-tab`
**Plan:** [`tasks/plans/PIM-FE-05/plan.md`](plans/PIM-FE-05/plan.md) (Status: Done)
**Scope:** weight + dimensions + HS code + barcode (EAN/UPC) + package type

**Test Cases:**
- [x] Input weight → validate positive number
- [x] Barcode field accept EAN-13 / UPC-A patterns

---

### [P2] PIM-FE-06 — Channels tab ✅ **merged 2026-05-13 (direct push, retrospective PR #172)**

**Type:** `[FE]` | **Effort:** M ~3-4h | **Depends on:** PIM-FE-00 ✅
**Branch:** `feat/pim-fe-06-channels-tab`
**Plan:** [`tasks/plans/PIM-FE-06/plan.md`](plans/PIM-FE-06/plan.md) (Status: Done)
**Scope:** per-marketplace mapping (Lazada/Shopee/TikTok/Web) + publish state + sync log + drift chips + provenance + sync banner

**Test Cases:**
- [x] Connect channel → publish state แสดง "Published · {time}"
- [x] Drift detected → warning chip แสดง driftField

---

### [P3] PIM-FE-07 — Audit tab ✅ **merged 2026-05-16 (PR #194)**

**Type:** `[FE]` | **Effort:** S ~1-2h | **Depends on:** PIM-FE-00 ✅
**Branch:** `feat/pim-fe-07-audit-tab`
**Plan:** [`tasks/plans/PIM-FE-07/plan.md`](plans/PIM-FE-07/plan.md) (Status: Done)
**Scope:** change history list + before/after diff + revert button

**Test Cases:**
- [x] Render history rows
- [x] Diff view เปิด → แสดง before/after ของ field ที่เปลี่ยน
- [x] Revert (confirm dialog) → call BE endpoint

---

### [P2] PIM-FE-08 — Categories CRUD ✅ **merged 2026-05-15 (PR #184)**

**Type:** `[FE]` | **Effort:** M ~3-4h | **Depends on:** PIM-FE-01 ✅
**Branch:** `feat/pim-fe-08-categories`
**Plan:** [`tasks/plans/PIM-FE-08/plan.md`](plans/PIM-FE-08/plan.md) (Status: Done)
**Scope:** hierarchical tree view + add/edit/delete + category-level attributes + channel mapping

**Test Cases:**
- [x] Tree expand/collapse
- [x] Add child → parent has new child node
- [x] Delete category with children → block + warning

---

### [P3] PIM-FE-09 — Brands CRUD ✅ **merged 2026-05-15 (PR #190)**

**Type:** `[FE]` | **Effort:** S ~1-2h | **Depends on:** PIM-FE-01 ✅
**Branch:** `feat/pim-fe-09-brands`
**Plan:** [`tasks/plans/PIM-FE-09/plan.md`](plans/PIM-FE-09/plan.md) (Status: Done)
**Scope:** list + add/edit/delete + logo upload + default warranty/contact info

**Test Cases:**
- [x] CRUD operations
- [x] Logo upload validation

---

### [P3] PIM-FE-10 — Attributes library ✅ **merged 2026-05-15 (PR #188)**

**Type:** `[FE]` | **Effort:** M ~3-4h | **Depends on:** PIM-FE-01 ✅, PIM-FE-08 (category bindings)
**Branch:** `feat/pim-fe-10-attributes`
**Plan:** [`tasks/plans/PIM-FE-10/plan.md`](plans/PIM-FE-10/plan.md) (Status: Done)
**Scope:** reusable attribute library (color/size/material) + validation rules + per-category bindings

**Test Cases:**
- [x] Add attribute + validation rule
- [x] Bind attribute to category → category form แสดง field

---

### [P1] PIM-E2E-01 — E2E: implement + un-skip acceptance tests ✅ **completed 2026-05-17 (commit `ea913d8`)**

**Type:** `[FE]` — E2E | **Effort:** ~~S 1-2h~~ **L 6-10h** (scope expanded — placeholder stubs)
**Depends on:** PIM-FE-02..07 ทั้งหมด done ✅
**Plan:** [`tasks/plans/PIM-E2E-01/plan.md`](plans/PIM-E2E-01/plan.md) (Status: Done)
**Outcome:** 34 pass / 3 fixme (audit tab — see `TEST-PIM-DRIFT-AUDIT-PROP`)

**Scope:**
- `e2e/pim/pim.spec.ts` — un-skip ทุก test
- Test cases: navigate, tabs switch, search, status filter, EN/TH toggle, create wizard

**Test Cases:**
- [ ] navigate `/pim/products` จาก sidebar
- [ ] คลิก row → ไป detail + tabs render
- [ ] สลับ tabs (overview → variants → pricing) state เปลี่ยน
- [ ] search/filter ทำงาน
- [ ] EN ↔ TH label เปลี่ยน

**Commit:** `test(e2e): enable pim acceptance tests`

---

### Task Summary — PIM (Product Master Data)

| Task | Role | Effort | Depends On | Status |
|------|------|--------|------------|--------|
| PIM-FE-00 | FE | M · 4h | — | ✅ Done (2026-05-12) |
| PIM-FE-01 | Docs | S · 1-2h | PIM-FE-00 | ✅ Done (2026-05-12) |
| PIM-SPEC-02 | Docs | M · 3-5h | PIM-FE-01 | ✅ Done (2026-05-13, PR #7) |
| PIM-BE-01a | BE | M · 3h | SPEC-02 | ✅ Done (2026-05-15, PR #78) |
| PIM-BE-01b | BE | M · 3h | BE-01a | ✅ Done (2026-05-15, PR #79) |
| PIM-BE-01c | BE | M · 2-3h | BE-01b | ⏳ Todo (unblocked) |
| PIM-FE-02 | FE | L · 5-8h | PIM-FE-01 | ✅ Done (2026-05-13, PR #167) |
| PIM-FE-03 | FE | M · 3-4h | PIM-FE-00 ✅ | ✅ Done (2026-05-13, direct push) |
| PIM-FE-04 | FE | M · 3-4h | PIM-FE-00 ✅ | ✅ Done (2026-05-15, PR #187) |
| PIM-FE-05 | FE | S · 1-2h | PIM-FE-00 ✅ | ✅ Done (2026-05-16, PR #193) |
| PIM-FE-06 | FE | M · 3-4h | PIM-FE-00 ✅ | ✅ Done (2026-05-13, retro PR #172) |
| PIM-FE-07 | FE | S · 1-2h | PIM-FE-00 ✅ | ✅ Done (2026-05-16, PR #194) |
| PIM-FE-08 | FE | M · 3-4h | PIM-FE-01 ✅ | ✅ Done (2026-05-15, PR #184) |
| PIM-FE-09 | FE | S · 1-2h | PIM-FE-01 ✅ | ✅ Done (2026-05-15, PR #190) |
| PIM-FE-10 | FE | M · 3-4h | PIM-FE-01 ✅, FE-08 | ✅ Done (2026-05-15, PR #188) |
| PIM-E2E-01 | FE | L · 6-10h (revised) | PIM-FE-02..07 ✅ | ✅ Done (2026-05-17, commit `ea913d8` — 34/37, 3 fixme) |
| **Total** | | **~35-50h** | | **14/15 done — only BE-01c remains + 1 P2 bug (`TEST-PIM-DRIFT-AUDIT-PROP`)** |

**Critical path FE:** PIM-FE-00 ✅ → PIM-FE-01 ✅ → PIM-FE-02 ✅ → Wave 1 (FE-03 ✅ + FE-06 ✅) → Wave 2 (FE-08 ✅ + FE-04 ✅ + FE-10 ✅) → Wave 3 (FE-05 ✅ + FE-07 ✅ + FE-09 ✅) → PIM-E2E-01 ✅
**Critical path BE:** PIM-FE-01 ✅ → PIM-SPEC-02 ✅ → PIM-BE-01a ✅ → PIM-BE-01b ✅ → **PIM-BE-01c** (next)
**Remaining (as of 2026-05-17 evening):**
- **PIM-BE-01c-2** (Pricing + variant attr values + their audit handlers, ~2-3h) ⏳ Next
- **PIM-BE-01c-3** (Channels + sync log + their audit handlers, ~2h) ⏳ After 01c-2

**Merged (2026-05-17 evening):** ✅
- BE PR #80 — `BUG-PIM-BE-01B-MERGE-LOSS` restored (squash `a4cdca8` on main)
- FE PR #197 — `PIM-E2E-01` 37 tests (squash `ba30a69` on main)
- FE PR #198 — `TEST-PIM-DRIFT-AUDIT-PROP` audit fix (merged onto main)
- FE PR #199 — `TEST-PIM-FE-08-MISSING` 7 component tests (squash `f5d904c` on main)
- FE PR #196 — wat `BUG-FE-RETURN-600` returnReason validation (squash `7f2d30d` on main)
- FE PR #192 — wat `OAD-INT-01` order-action-dialog scaffold (squash `030ef31` on main)

**Corrective merges resolved:**
- BE PR #81 — merged into wrong base (fix branch) at `7fcf311`
- BE PR [#82](https://github.com/B1DXDev/b1dx-oms-fulfillment-api/pull/82) — corrective re-merge ✅ **MERGED to main** via squash `b4e474c`. BE-01c-1 (Domain Event Dispatch + Audit) now on main. 542/542 unit tests pass.
- FE PR #198 — merged into wrong base (`test/pim-e2e-01` stacked branch) at `a19f9a8`
- FE PR [#200](https://github.com/B1DXDev/b1dx-oms-fulfillment-web/pull/200) — corrective re-merge ✅ **MERGED to main** via squash `66cf121`. `ProductDetailView.tsx` now passes `productId` to `<AuditTab>`; 3 audit E2E un-fixmed.

**Lesson learned (stacked PR merge trap):**
Squash-merging the BASE of a stacked PR (e.g., merge PR #80 first) does NOT update the stacked PR's base. When you then merge the stacked PR (e.g., #81 or #198), GitHub merges it into the original base branch (which still exists as a remote ref), NOT into main. The stacked PR shows MERGED state but the code is stranded on the base branch in the remote. **Mitigation:** before merging a stacked PR, change its base to main first via `gh pr edit <num> --base main`; OR cherry-pick the squash commit onto main after the fact.

**Branch cleanup (2026-05-17 evening):**
24 local + 17 remote stale branches deleted across 3 repos. Final state: BE main only; FE main + 3 remote-only unknowns; Workspace main + 2 active.

**Resolved (no PR — not actually a bug):**
- `FIX-PIM-FE-03-FOOTER-BUTTONS` — buttons existed since FE-03 merge

---

## Issues Backlog — Profile Settings Security Tab

### [P2] FIX-PRFSETTINGS-SECURITY-V3 — Security tab ไม่ตรง mockup v3 `[srattha]` `[FE]`

**Found:** 2026-05-21 | **Type:** UI fidelity | **Repo:** `b1dx-oms-fulfillment-web`
**Mockup Ref:** `docs/mockups/profile_settings_v3.html` § Tab 1 Security

**Symptom:** Tab Security ใน `/profile/settings` ไม่ตรง mockup v3 — 3 ส่วนหลักขาดหาย/ไม่ครบ แม้ tasks FE-PRFSETTINGS-16/17/18/19 ถูก mark done ใน sprint

**Root Cause:** Tasks ถูก mark done ใน `current-sprint.md` แต่ component files ที่ plan ระบุไม่ได้ถูกสร้างจริง — implementation ยังเป็น simplified version จาก FE-PRFSETTINGS-05 (v1) ไม่ใช่ v3

**Gap (current vs mockup v3):**

| Element | Mockup v3 | Current | สถานะ |
|---|---|---|---|
| 2FA Nudge Banner | Amber dismissible banner (🔐 + ข้อความ + ✕ ปิด) | ❌ ไม่มี | MISSING |
| 2FA row | status tag (ปิดอยู่/เปิดอยู่) + 🔔 nudge btn + enable/disable | badge + btn เท่านั้น | INCOMPLETE |
| Trusted Devices | "จัดการ ▾" toggle → expandable device list + delete per item | single summary row เท่านั้น | INCOMPLETE |
| Active Sessions section | card แยก: session list + "อุปกรณ์นี้" tag + IP/browser meta + Revoke per session + "Logout ทั้งหมด" | ❌ ไม่มีเลย | MISSING |
| Activity Log | filter chips (ทั้งหมด/Login/Actions/Errors) + expandable row detail | simple list ไม่มี filter/expand | INCOMPLETE |

**Files to CREATE:**
- `features/users/components/Tab1Security/ActiveSessionsSection.tsx` + `.test.tsx`
- `features/users/components/Tab1Security/ActivityLogSection.tsx` + `.test.tsx`
- `features/users/components/Tab1Security/TrustedDevicesExpand.tsx` + `.test.tsx`

**Files to MODIFY:**
- `features/users/components/Tab1Security/Tab1Security.tsx` — 2FA row + Nudge banner + wire sub-components

**Test Cases:**
- [ ] regression: 2FA nudge banner แสดงเมื่อ 2FA ปิด + ซ่อนได้ด้วยปุ่ม ✕
- [ ] regression: Trusted Devices row กด "จัดการ ▾" → expand/collapse device list
- [ ] regression: Active Sessions section แสดง session list + current device tag + Revoke button
- [ ] regression: Activity Log filter chips กรอง type (login/action/error) ได้
- [ ] regression: Activity Log row click → expand detail (IP, browser, OS)
- [ ] visual: screenshot vs mockup v3 § Tab 1 ผ่านทุก element

**Dependencies:** FE-PRFSETTINGS-11 (types) ✅ merged

---

### 🎯 CI-GATE Initiative — PR-gated test enforcement (4 repos) `[wat]`

**Found:** 2026-05-23 | **Type:** CI/Infra | **Repos:** FE, BE, Sync Worker, Webhook
**Source:** Roadmap doc `docs/guides/9.ci-roadmap.html` (2026-05-23)
**Trigger:** Test ครบทุก repo แต่ไม่มี GH Action บังคับ → bug หลุดเข้า main ได้
**Status:** Tickets opened, no work started

#### TL;DR — ไม่มีระบบบังคับ test ก่อน merge

- 4 repos มี test เขียนแล้ว แต่ไม่มี `.github/workflows/ci.yml`
- Dev ต้องจำเอง (CLAUDE.md rule) → discipline-dependent → ลืมได้
- Incident reference: EOD-FE-04 (2026-04-27) merged โดย visual fidelity ไม่ครบ

#### Solution — 2 phases (Phase 3 deferred)

**Phase 1 (Test Gate, ~5d):** บังคับ test run ทุก PR + branch protection
**Phase 2 (Quality Gates, ~7d):** @critical smoke + husky pre-push + visual regression + coverage

#### Tickets summary

| ID | P | Phase | Repo | Effort | ผลที่ได้ |
|---|---|---|---|---|---|
| CI-WEBHOOK-GATE-01 | P1 | 1 | webhook | ~3h | go test gate ทุก PR |
| CI-BE-GATE-01 | P1 | 1 | BE | ~4h | dotnet test gate ทุก PR |
| CI-SYNC-GATE-01 | P1 | 1 | sync-worker | ~4h | dotnet test gate (TestContainers) |
| CI-FE-GATE-01 | P1 | 1 | FE | ~6h | lint + typecheck + vitest + Playwright |
| CI-FE-CRITICAL-01 | P2 | 2 | FE | ~4h | @critical smoke set (~30s pre-push) |
| CI-HUSKY-01 | P2 | 2 | all 4 | ~3h | pre-push hook ลด CI fail loop |
| CI-VISUAL-01 | P2 | 2 | FE | ~6h | toHaveScreenshot baseline (mockup fidelity auto) |
| CI-COVERAGE-01 | P2 | 2 | all 4 | ~4h | Codecov badge + PR comment delta |

**Recommended execution order (Phase 1 first):**
CI-WEBHOOK-GATE-01 (ง่ายสุด pilot) → CI-BE-GATE-01 + CI-SYNC-GATE-01 (parallel) → CI-FE-GATE-01 → user ตั้ง branch protection → Phase 2 tickets

> **Note:** Branch protection setup ทำหลัง workflow merge ลง main สำเร็จ + run pass 1 รอบ. Agent print `gh` cmd ให้ user รัน (Tier 1 stop). AI auto-review (Phase 3) deferred — ต้องตั้ง ANTHROPIC_API_KEY + spend cap.

---

### 🆕 [P1] CI-WEBHOOK-GATE-01 — Webhook CI workflow (go test gate) `[wat]`

**Found:** 2026-05-23 | **Type:** CI/Infra | **Repo:** `webhook`
**Source:** CI-GATE Initiative (Phase 1)
**Plan:** `tasks/plans/CI-WEBHOOK-GATE-01/plan.md` (TBD)
**Effort:** ~3h
**Depends on:** — (pilot ticket — ง่ายสุด ใช้ตรวจ pattern)

#### Goal
สร้าง `.github/workflows/ci.yml` ใน webhook repo. ทุก PR auto-run `go vet` + `go build` + `go test`. Fail → block merge.

#### Scope
- NEW `.github/workflows/ci.yml` — trigger `pull_request` + `push: main`
- Setup Go 1.25 + actions/cache@v4
- Steps: `go vet ./...` + `go build ./...` + `go test ./... -race -count=1`
- Upload test log artifact on failure
- Concurrency: cancel-in-progress

#### Done Gate
- [ ] PR สร้าง workflow merged ลง main
- [ ] CI run pass บน main 1 รอบ
- [ ] Branch protection ตั้งแล้ว (check name = `ci`)
- [ ] Test PR ใหม่ — fail = merge button disabled

---

### 🆕 [P1] CI-BE-GATE-01 — BE CI workflow (dotnet test gate) `[wat]`

**Found:** 2026-05-23 | **Type:** CI/Infra | **Repo:** `b1dx-oms-fulfillment-api`
**Source:** CI-GATE Initiative (Phase 1)
**Plan:** `tasks/plans/CI-BE-GATE-01/plan.md` (TBD)
**Effort:** ~4h
**Depends on:** —

#### Goal
สร้าง `.github/workflows/ci.yml` ใน BE repo. ทุก PR auto-run `dotnet build` + `dotnet test`.

#### Scope
- NEW `.github/workflows/ci.yml` — trigger `pull_request` + `push: main`
- Setup .NET 10 + nuget cache
- Steps: `dotnet restore` + `dotnet build --configuration Release` + `dotnet test --no-build`
- Upload trx artifact on failure
- ตรวจ env vars / secrets ที่ unit + integration test ต้องใช้ (connection string, etc.) — ถ้าขาด → ระบุใน plan

#### Done Gate
- [ ] PR merged + CI run pass บน main 1 รอบ
- [ ] Branch protection ตั้งแล้ว
- [ ] Test PR ใหม่ — fail = block merge

---

### 🆕 [P1] CI-SYNC-GATE-01 — Sync Worker CI workflow (dotnet + TestContainers) `[wat]`

**Found:** 2026-05-23 | **Type:** CI/Infra | **Repo:** `b1dx.omsorders.sync.worker`
**Source:** CI-GATE Initiative (Phase 1)
**Plan:** `tasks/plans/CI-SYNC-GATE-01/plan.md` (TBD)
**Effort:** ~4h
**Depends on:** —

#### Goal
สร้าง `.github/workflows/ci.yml` ใน Sync Worker repo. รัน `dotnet test` รวม TestContainers integration test.

#### Scope
- NEW `.github/workflows/ci.yml` — `ubuntu-latest` (รองรับ Docker for TestContainers)
- Setup .NET 10 + nuget cache + Docker (built-in on ubuntu-latest)
- Steps: `dotnet build` + `dotnet test --logger trx`
- ถ้า TestContainers fail → ลอง `services:` block (Postgres, RabbitMQ)
- Upload trx + container log on failure

#### Done Gate
- [ ] PR merged + CI run pass บน main 1 รอบ
- [ ] TestContainers ทำงานจริงใน CI (Docker available)
- [ ] Branch protection ตั้งแล้ว
- [ ] Test PR — fail = block merge

---

### 🆕 [P1] CI-FE-GATE-01 — FE CI workflow (vitest + Playwright) `[wat]`

**Found:** 2026-05-23 | **Type:** CI/Infra | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** CI-GATE Initiative (Phase 1)
**Plan:** `tasks/plans/CI-FE-GATE-01/plan.md` (TBD)
**Effort:** ~6h (longest — Playwright setup)
**Depends on:** — (but Webhook/BE/Sync ลำดับก่อน เพื่อ test pattern)

#### Goal
สร้าง `.github/workflows/ci.yml` ใน FE repo. รัน lint + typecheck + vitest + Playwright (chromium).

#### Scope
- NEW `.github/workflows/ci.yml` — `pull_request` + `push: main`
- Setup pnpm 9 + Node 22 + cache pnpm-lock + cache Playwright browsers
- Steps:
  1. `pnpm install --frozen-lockfile`
  2. `pnpm lint && pnpm typecheck`
  3. `pnpm test` (Vitest unit)
  4. `pnpm exec playwright install chromium --with-deps`
  5. `pnpm exec playwright test --project=chromium --reporter=html,list` (env `NEXT_PUBLIC_MSW=1`)
- Upload `playwright-report/` artifact on failure (retention 14 days)
- Concurrency: cancel-in-progress
- Timeout: 30min

#### Done Gate
- [ ] PR merged + CI run pass บน main 1 รอบ (Playwright + Vitest ทั้งคู่)
- [ ] Branch protection ตั้งแล้ว
- [ ] Test PR ใหม่ — fail = block merge
- [ ] Verify cache hit rate ดี (boot time ครั้งที่ 2 < 1min)

---

### 🆕 [P2] CI-FE-CRITICAL-01 — @critical smoke set + CI fast lane (FE) `[wat]`

**Found:** 2026-05-23 | **Type:** Test infra | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** CI-GATE Initiative (Phase 2)
**Plan:** `tasks/plans/CI-FE-CRITICAL-01/plan.md` (TBD)
**Effort:** ~4h
**Depends on:** CI-FE-GATE-01

#### Goal
Tag test 5-10 ตัวครอบ flow หลัก ด้วย `@critical`. รันก่อน full suite (~30s) → fail เร็ว ประหยัด CI minutes + dev pre-push ได้.

#### Scope
- เลือก critical flows: login + perm gate + bulk action + transition 450/800 + integrations sync
- เพิ่ม `@critical` tag ใน test title (~10 tests)
- Update `ci.yml` — 2-step job: critical smoke ก่อน (`-g @critical`) แล้วค่อย full suite
- Document tag convention ใน `e2e/README.md` (ถ้ามี) หรือ inline comment

#### Done Gate
- [ ] 10 tests tagged `@critical` covering core flows
- [ ] CI runs smoke ก่อน full suite (`-g @critical`)
- [ ] Smoke time < 45s บน CI
- [ ] PR comment / annotation บอกว่า smoke pass ก่อน full suite

---

### 🆕 [P2] CI-HUSKY-01 — Husky pre-push hook (4 repos) `[wat]`

**Found:** 2026-05-23 | **Type:** Dev tooling | **Repos:** FE, BE, Sync, Webhook
**Source:** CI-GATE Initiative (Phase 2)
**Plan:** `tasks/plans/CI-HUSKY-01/plan.md` (TBD)
**Effort:** ~3h (4 repos × ~45min)
**Depends on:** CI-FE-CRITICAL-01 (FE smoke set ต้องพร้อมก่อน)

#### Goal
ติดตั้ง husky pre-push hook ใน 4 repos. ก่อน push → run smoke (~30s) → fail = ห้าม push → ลด CI fail loop.

#### Scope
- FE: husky + lint-staged → `pnpm exec playwright test -g @critical`
- BE: pre-push git hook (bash script) → `dotnet test --filter Category=Critical` (ต้อง tag test ก่อน)
- Sync: same as BE
- Webhook: `go test ./... -run Critical -short`

#### Done Gate
- [ ] Husky/git hook ติดตั้งใน 4 repos (auto-install on `pnpm install` / `npm install`)
- [ ] Pre-push fail = exit non-zero → push aborted
- [ ] Document `--no-verify` escape hatch + กฎห้ามใช้ (CLAUDE.md)

---

### 🆕 [P2] CI-VISUAL-01 — Visual regression baselines (FE) `[wat]`

**Found:** 2026-05-23 | **Type:** Test infra | **Repo:** `b1dx-oms-fulfillment-web`
**Source:** CI-GATE Initiative (Phase 2)
**Plan:** `tasks/plans/CI-VISUAL-01/plan.md` (TBD)
**Effort:** ~6h
**Depends on:** CI-FE-GATE-01

#### Goal
ทำ visual regression สำหรับ `/orders` 6 roles × order detail × bulk ops. Pixel-diff auto detect mockup drift. แทน manual screenshot review (CLAUDE.md "Mockup Visual Fidelity BLOCKING" rule).

#### Scope
- ใช้ `await expect(page).toHaveScreenshot('orders-{role}.png')`
- Stabilize timestamps + animations: `animations: 'disabled'` + freeze date
- Baseline ครั้งแรก: `--update-snapshots` (manual approve PR)
- `maxDiffPixels: 100` กัน font sub-pixel
- ใส่ใน existing `permissions.spec.ts` หรือสร้าง `visual.spec.ts` ใหม่
- CI: ใช้ docker container เดียวกับ baseline (font/render consistent)

#### Done Gate
- [ ] 6 roles × order list baseline บันทึก
- [ ] Order detail baseline (3 statuses: 400, 490, 700)
- [ ] PR diff visible เมื่อ UI change
- [ ] False positive rate < 5% (animation, timestamp, async data)

---

### 🆕 [P2] CI-COVERAGE-01 — Coverage reporter (4 repos) `[wat]`

**Found:** 2026-05-23 | **Type:** Test infra | **Repos:** FE, BE, Sync, Webhook
**Source:** CI-GATE Initiative (Phase 2)
**Plan:** `tasks/plans/CI-COVERAGE-01/plan.md` (TBD)
**Effort:** ~4h
**Depends on:** CI-{FE,BE,SYNC,WEBHOOK}-GATE-01

#### Goal
ใส่ coverage reporter + PR comment delta. ดูแลไม่ให้ coverage ลด.

#### Scope
- FE: `pnpm test --coverage` → upload to Codecov
- BE/Sync: `dotnet test --collect:"XPlat Code Coverage"` → coverlet → Codecov
- Webhook: `go test -coverprofile=coverage.out` → Codecov
- PR comment via `codecov/codecov-action@v4`
- README badge 4 repos

#### Done Gate
- [ ] Codecov account + token ตั้งเสร็จ (org-level secret)
- [ ] 4 workflows upload coverage
- [ ] PR comment แสดง delta (+/-%)
- [ ] Badge ใน README ทุก repo
