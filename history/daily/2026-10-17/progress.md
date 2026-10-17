# BeOne OMS — Progress Tracker
> อัพเดทล่าสุด: 2026-04-10
> Legend: ✅ Done | 🔵 In Progress | 🟡 Blocked | ⬜ Todo | ❌ Cancelled
>
> **หมายเหตุ:** ไฟล์นี้เป็น high-level sprint overview
> งาน task detail ดูที่ `tasks/backlog.md` | sprint ปัจจุบัน `tasks/current-sprint.md`

---

## Phase 1 — Foundation

### Sprint 1 — Auth + RBAC
> Auth flow, Security, Forgot/Reset, RBAC (Users + Permission Matrix)

#### Auth (Sprint 1 + 1.1)
| ID | งาน | สถานะ | หมายเหตุ |
|---|---|---|---|
| S1-FE-1 | FE Foundation (X-Tenant-Id, 401 Interceptor, App Shell, Error Pages) | ✅ | — |
| S1-FE-2 | FE Auth UX (OTP Paste, Skip Role Screen, defaultMembershipId) | ✅ | — |
| S1-FE-3 | FE Forgot/Reset Flow (ForgotPasswordForm + ResetPasswordForm) | ✅ | — |
| S1-FE-4 | FE OTP Resend (Cooldown 60s + no flicker) | ✅ | — |
| S1-BE-1 | BE Auth Flow (JWT, OtpService, verify-otp, token, refresh, logout, me) | ✅ | — |
| S1-BE-2 | BE Security (OTP Rate Limiting, ActivityLog writers, Invited→Active) | ✅ | — |
| S1-BE-3 | BE Forgot/Reset (DB Migration + 3 endpoints) | ✅ | — |

#### RBAC (Sprint 1.2)
| ID | งาน | สถานะ | หมายเหตุ |
|---|---|---|---|
| S12-BE-1 | BE Users CRUD (list, detail, update, invite, assign role) | ✅ | — |
| S12-BE-2 | BE Roles + Permission Editor (list roles, PUT permissions) | ✅ | — |
| S12-FE-1 | FE Users (Types, MSW, hooks, KPI cards, table, Add/Detail drawers) | ✅ | — |
| S12-FE-2 | FE RBAC (Role Overview cards, Permission Editor matrix) | ✅ | — |
| S12-FE-3 | FE RBAC Gaps (UserStatus fix, usePermission hook, User type v2 fields) | ⬜ | S2-RBAC-01/02/03 |
| S12-FE-4 | FE Bulk select bar + E2E test | ⬜ | — |

---

## Phase 2 — Order Management + Marketplace Sync

### Sprint 2 — Order BE + FE + Sync Worker
> BE Order Domain/API เสร็จ, FE กำลังทำ, Sync Worker Step 1.5

#### BE Order
| ID | งาน | สถานะ | หมายเหตุ |
|---|---|---|---|
| S2-BE-1 | Shared Foundation (8 entities, 9 enums, TransitionService, EF Migration) | ✅ | Srattha |
| S2-BE-2 | Order List (GetOrdersQuery + GET /orders + /orders/options) | ✅ | Wat+Srattha |
| S2-BE-3 | Order Detail (GetDetailQuery, ChangeStatus, Notes commands + endpoints) | ✅ | Srattha |

#### FE Order
| ID | งาน | สถานะ | หมายเหตุ |
|---|---|---|---|
| S2-FE-1 | Types+Enums update + MSW handlers (Order List) | ⬜ | S3-OL-01/02 |
| S2-FE-2 | useOrders hook + Status/Payment/Method badges | ⬜ | S3-OL-03/04/05/06 |
| S2-FE-3 | Order List page wire (filters, tabs, URL params) | ⬜ | S3-OL-07 |
| S2-FE-4 | MSW Order Detail + useOrderDetail + mutation hooks | ⬜ | S3-OD-01/02 |
| S2-FE-5 | Order Detail page wire (Hero, Items, Notes, Timeline, Actions) | ⬜ | S3-OD-03 |

#### Sync Worker
| ID | งาน | สถานะ | หมายเหตุ |
|---|---|---|---|
| S2-SY-1 | DB Migrations — Foundation, Warehouse, Checkpoint PK | ✅ | Nuchit |
| S2-SY-2 | Domain Models + Parsers (Shopee/Lazada/TikTok scaffold) + StatusRegressionGuard | ✅ | Nuchit |
| S2-SY-3 | Step 1.5 — Seed Data + Unit Tests (Shopee/Lazada parsers + Guard) | ✅ | Nuchit, 2026-04-10 |
| S2-SY-4 | Marketplace Sync (Phase 0-4: Schema, Domain, Services, Tests, Observability) | ✅ | Nuchit, 2026-04-10 |
| S2-SY-5 | ERP Order Sync (Phase 1-4: Domain, Reader, Repo, Service, Tests, Observability) | ✅ | Nuchit, 2026-04-10 |

---

## Phase 3 — Master Data & WMS

### Sprint 3+ — Master Data, WMS Operations
> รอ Phase 2 Order FE เสร็จก่อน

#### Master Data: SKU
| ID | งาน | สถานะ | หมายเหตุ |
|---|---|---|---|
| S3-MD-1 | BE SKU (entity, CRUD commands/queries, EF migration, endpoints) | ⬜ | — |
| S3-MD-2 | FE SKU (types, MSW, hooks, list page + form) | ⬜ | — |

#### PIM (Product Master Data) — `feat/product_master_data`
| ID | งาน | สถานะ | หมายเหตุ |
|---|---|---|---|
| PIM-FE-00 | Scaffold (List + Detail + Overview tab + nav + i18n) | ✅ | nuchit · merged 2026-05-12 (PR #166 `afe1bd6` + WS PR #3 `05d9419`) |
| PIM-FE-01 | Spec authoring (data model + API contract + permission keys) | ⬜ | depends PIM-FE-00 |
| PIM-BE-01 | Product aggregate + EF migration + endpoints | ⬜ | depends PIM-FE-01 |
| PIM-FE-02 | Variants tab (axes + matrix + 3 view modes + bulk-edit) | ⬜ | depends PIM-FE-01 |
| PIM-FE-03 | Pricing tab | ⬜ | depends PIM-FE-00 |
| PIM-FE-04 | Media tab | ⬜ | depends PIM-FE-00 |
| PIM-FE-05 | Logistics tab | ⬜ | depends PIM-FE-00 |
| PIM-FE-06 | Channels tab (LZD/SHO/TIK/Web mapping) | ⬜ | depends PIM-FE-00 |
| PIM-FE-07 | Audit tab | ⬜ | depends PIM-FE-00 |
| PIM-FE-08 | Categories CRUD | ⬜ | depends PIM-FE-01 |
| PIM-FE-09 | Brands CRUD | ⬜ | depends PIM-FE-01 |
| PIM-FE-10 | Attributes library | ⬜ | depends PIM-FE-01 |
| PIM-E2E-01 | E2E acceptance tests (un-skip) | ⬜ | depends PIM-FE-02..07 |

#### WMS Operations
| ID | งาน | สถานะ | หมายเหตุ |
|---|---|---|---|
| S3-WMS-1 | BE WMS (Inbound, PickList, Pack, Ship, Return + Stock) | ⬜ | — |
| S3-WMS-2 | FE WMS pages (Inbound, Pick, Pack, Ship, Return) | ⬜ | — |

---

## Phase 4 — Finance, Reports & Integrations

### Sprint 4+ — Finance + Reports + Integrations
> รอ Phase 3 เสร็จก่อน

#### Finance & Reports
| ID | งาน | สถานะ | หมายเหตุ |
|---|---|---|---|
| S4-FIN-1 | Finance & Documents (Invoice, COD, Billing) | ⬜ | — |
| S4-RPT-1 | Reports & Monitor (Dashboard, Export Excel) | ⬜ | — |
| S4-ACT-1 | Activity Traceability (Log, Video, Audit) | ⬜ | — |

#### Integrations & Promotions
| ID | งาน | สถานะ | หมายเหตุ |
|---|---|---|---|
| S4-INT-1 | Platform Integrations full (Shopee/Lazada/TikTok/LINE) | ⬜ | — |
| S4-PRO-1 | Promotion Engine (buy A get X, date range rules) | ⬜ | — |

---

## Phase 5 — Webhook Peak Load Initiative

### Sprint WH-CAP — 11/11 Readiness
> Owner: Sompon (Benz) · 8 tickets, ~5-7 dev-days · onboarded 2026-05-19 · deadline 11/11 peak

#### Webhook Capacity
| ID | งาน | สถานะ | หมายเหตุ |
|---|---|---|---|
| WH-CAP-04 | Queue safety: x-max-length + DLX + message TTL | ✅ | Benz, 2026-05-21 (PR #18) |
| WH-CAP-01 | Worker concurrency + Shopee per-webhook resync rewrite | ⬜ | Benz · P1 (blocks 07,08) |
| WH-CAP-02 | Publisher channel pool + Confirm mode + reconnect | ⬜ | Benz · P2 |
| WH-CAP-03 | HTTP server timeouts + body size cap | ⬜ | Benz · P2 |
| WH-CAP-05 | WebhookEvent idempotency (unique index + Upsert) | ⬜ | Benz · P2 |
| WH-CAP-06 | Bounded fan-out fallback (errgroup) | ⬜ | Benz · P2 |
| WH-CAP-07 | Per-platform/per-shop rate limiter (token bucket) | ⬜ | Benz · blocked by WH-CAP-01 |
| WH-CAP-08 | Horizontal scale worker pods + queue contention test | ⬜ | Benz · blocked by WH-CAP-01 |

---

## Ad-hoc — งานนอกแผน
> งานที่เพิ่มระหว่างทาง ไม่ได้อยู่ใน phase เดิม

| ID | งาน | Layer | สถานะ | หมายเหตุ |
|---|---|---|---|---|
| S2-CLEAN-01 | ลบ apps/saas-starter ออกจาก FE monorepo | FE | ✅ | Wat, 2026-03-29 |
| S2-RBAC-01 | UserStatus inactive→disabled fix | FE | ✅ | Srattha, 2026-03-31 |
| S2-RBAC-02 | usePermission hook | FE | ✅ | Wat, 2026-04-02 |
| S2-RBAC-03 | User type: skipOtp + defaultMembershipId | FE | ✅ | Wat, 2026-04-02 |

---

## Developer Workload — Sprint 2 (2026-04-10)

### Nuchit — Sync Worker Lead
> **Tasks:** S2-MSY-02a (Marketplace Sync), S2-ERP-01 to 09 (ERP Order Sync) + S2-SEC-01 (JWT RS256)
> **Status:** 🎯 **18 tasks completed** | **Lines of Code:** ~8,500 | **Tests Written:** 65+ unit + integration

#### Completed (2026-04-10)
- ✅ **S2-MSY-08 to S2-MSY-02a-11** — Marketplace Sync (Phase 0-4)
  - Schema migration public → oms
  - Connection factories + domain models + parsers (Shopee, Lazada)
  - Sync service + background service + health checks
  - Integration tests (10 scenarios, TestContainers PostgreSQL)
  - Documentation + observability + performance tuning

- ✅ **S2-ERP-01 to S2-ERP-09** — ERP Order Sync
  - Domain models (ErpOrder, ErpOrderItem) + UTC conversion
  - Flag-based reader (xfer/xferstep states)
  - OMS repository with optimistic locking
  - Sync service + background service + DI registration
  - Integration tests (happy path, idempotency, concurrency)
  - Health checks + OTel metrics

#### Today's Commits
- `19798cd` oms_dev → oms_db (env var rename)
- `8b2d314` RenameColumnsToSnakeCase migration (safe, idempotent)
- `9621f15` Mark S2-ERP-01 to 07 done
- `651c7f6` Update standup: S2-ERP completed
- `798ca7b` Remove sprint summary file

---

### Srattha — BE Lead + FE
> **Tasks:** S2-RBAC order/auth implementation, email infrastructure, FE Order pages
> **Status:** 🎯 **25+ tasks completed** | **Merged PRs:** 8

#### Recent (last 2 days)
- ✅ S2-AUTH-01/02 — ForgotPasswordForm + ResetPasswordForm (RHF + Zod)
- ✅ S2-EMAIL-INFRA-01 to 06 — Email infrastructure (SMTP, Fluid templates, background service)
- ✅ S2-EMAIL-02/03/04 — Email templates (password reset, welcome invite, OTP verification)
- ✅ S1-06 to 09 — Forgot/Reset password (BE) with 3 endpoints
- ✅ S2-RBAC-14/15/16 — Permission editor wire + seed data alignment
- ✅ S2-ABAC-01/02 — ABAC conditions CRUD + FE wire
- ✅ S2-CR-01 — Custom roles CRUD API
- 🔄 S2-CR-02/S2-RH-02 — In progress (custom role modal + hierarchy wire)

#### FE Order (Completed)
- ✅ S3-OL/OD complete (8 FE tasks + 1 BE search expansion)
  - Types, enums, MSW, hooks, badges, pages, search bar

---

### Wat — BE + FE RBAC Specialist
> **Tasks:** S2-RBAC permission/role gates, FE auth enhancements
> **Status:** 🎯 **12+ tasks completed** | **Merged PRs:** 5

#### Completed
- ✅ S2-RBAC-02/03 — usePermission hook + user type v2 (permissions, skipOtp)
- ✅ S2-RBAC-02b — GET /me + permissions[] in response
- ✅ S2-RBAC-04 — PermissionGate component + 4-layer UsersPage
- ✅ S2-RBAC-05 — Permission editor dirty state + save UX
- ✅ S2-RBAC-12 — Change role dialog in UserDetailDrawer
- ✅ S2-RBAC-17 — Permission keys v2 alignment
- ✅ BUG-RBAC-PERM-ALIGN — BE seed + FE 9-file alignment
- ✅ S2-RBAC-06/11 — User count badge + reset password button
- ✅ BUG-RBAC-07/09 — UserList sort/resize + search/filter
- ✅ S2-CLEAN-01 — Remove saas-starter from monorepo
- ✅ S2-OL-01/02 — BE Order list queries + endpoints (helper to Srattha)

---

## Sprint 2 Progress Summary
- **Total Tasks Open:** 85
- **✅ Done:** 76 (89%) — Order BE/FE, RBAC core, Auth, Email, Marketplace Sync, ERP Sync
- **🔄 In Progress:** 2 (2%) — S2-CR-02, S2-RH-02
- **⏳ Pending:** 7 (8%) — Force Password Change flow (6 tasks) + User role quick filter

### Critical Path Unblocked for Sprint 3
- ✅ Order API + FE pages ready
- ✅ User/RBAC permission gates ready
- ✅ Sync worker (Marketplace + ERP) ready for production
- 🔄 Awaiting: FPC flow completion, custom role modal finalization
