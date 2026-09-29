---
# Backlog Features — Source of Truth
# Edit this file, then run: node scripts/gen-backlog.mjs
# Output: tasks/backlog.html
# Each group renders ~3-5 feature cards. Preserve order — UI relies on it.

groups:
  - id: g1
    title: "GROUP 1 — Auth Foundation"
    priority: P1
    features:
      - id: auth-flow
        title: "Auth Flow — Login, OTP, Select Role"
        priority: P1
        goal: "ผู้ใช้ login ด้วย email/password → verify OTP → เลือก Role → เข้าสู่ระบบ"
        screens: [login, otp, role]
        spec: "docs/specs/auth-flow.md"
        notes: "Token strategy — access token in-memory, refresh token httpOnly cookie"
        defaultStatus: sprint
      - id: forgot-password
        title: "Forgot / Reset Password"
        priority: P1
        goal: "ผู้ใช้ขอรีเซ็ตรหัสผ่านผ่าน email"
        screens: [forgot, reset, success]
        notes: "reset token single-use, expire ภายใน 15 นาที"
        defaultStatus: backlog
  - id: g2
    title: "GROUP 2 — Order Management"
    priority: P1
    features:
      - id: order-list
        title: "Order List + KPI Dashboard"
        priority: P1
        goal: "แสดงรายการ Order พร้อม KPI cards (ยอดรวม, pending, shipped ฯลฯ)"
        screens: [dashboard, orders]
        spec: "docs/specs/order-management.md"
        notes: "pagination server-side, filter by status/brand/warehouse, search by order no."
        defaultStatus: backlog
      - id: order-detail
        title: "Order Detail"
        priority: P1
        goal: "ดูรายละเอียด Order ครบถ้วน — items, timeline, status, actions"
        screens: [detail]
        notes: "แสดง audit trail ของ order นั้นๆ"
        defaultStatus: backlog
      - id: status-flow
        title: "Status Flow (Workflow Engine)"
        priority: P1
        goal: "กำหนด workflow การเปลี่ยน status ของ Order แต่ละ type"
        screens: [flow]
        notes: "visual state machine, กำหนด allowed transitions"
        defaultStatus: backlog
      - id: bulk-ops
        title: "Bulk Operations"
        priority: P2
        goal: "จัดการ Order หลายรายการพร้อมกัน (approve, cancel, export)"
        screens: [bulkops]
        notes: "max 100 orders ต่อ batch"
        defaultStatus: backlog
  - id: g3
    title: "GROUP 3 — Access Control"
    priority: P1
    features:
      - id: user-management
        title: "User Management (Add / Edit / Detail)"
        priority: P1
        goal: "Admin จัดการ User ในระบบ — เพิ่ม, แก้ไข, ดูรายละเอียด, deactivate"
        screens: [ac-users]
        spec: "docs/specs/user-management.md"
        specReady: true
        defaultStatus: backlog
      - id: rbac
        title: "Roles & RBAC + Permission Matrix"
        priority: P1
        goal: "กำหนด Role และ Permission ต่อ Resource/Action"
        screens: [ac-rbac, ac-perm]
        spec: "docs/specs/rbac.md"
        specReady: true
        defaultStatus: backlog
      - id: status-perm
        title: "Status Permissions"
        priority: P2
        goal: "กำหนดว่า Role ไหนทำ action อะไรกับ Order status ได้บ้าง"
        screens: [ac-stperm]
        notes: "permission matrix ระดับ status transition"
        defaultStatus: backlog
  - id: g4
    title: "GROUP 4 — Warehouse WMS"
    priority: P2
    features:
      - id: inbound
        title: "Inbound Management"
        priority: P2
        goal: "รับสินค้าเข้า warehouse — scan, verify, putaway"
        screens: [inbound]
        defaultStatus: backlog
      - id: pickpack
        title: "Pick / Pack"
        priority: P2
        goal: "หยิบและแพ็คสินค้าตาม Order — print label, confirm"
        screens: [pickpack]
        defaultStatus: backlog
      - id: returns
        title: "Returns & Non-Conforming"
        priority: P2
        goal: "จัดการสินค้าคืน และสินค้าที่ไม่ผ่าน QC"
        screens: [returns]
        defaultStatus: backlog
      - id: cyclecount
        title: "Cycle Count (Stock Verification)"
        priority: P2
        goal: "นับสต็อกจริง เทียบกับ system และ reconcile"
        screens: [cyclecount]
        defaultStatus: backlog
  - id: g5
    title: "GROUP 5 — Admin & Reporting"
    priority: P2
    features:
      - id: reports
        title: "Reports & Analytics"
        priority: P2
        goal: "รายงานสรุป order, inventory, performance ตาม period/brand/warehouse"
        screens: [reports]
        notes: "chart + export CSV/Excel, filter date range"
        defaultStatus: backlog
      - id: audit-log
        title: "Audit Log"
        priority: P2
        goal: "บันทึกทุก action ที่เกิดในระบบ — who did what, when"
        screens: [audit]
        notes: "immutable, searchable, retention 90 วัน"
        defaultStatus: backlog
      - id: lifecycle
        title: "Order Lifecycle Timeline"
        priority: P2
        goal: "แสดง timeline ของ Order ตั้งแต่สร้างจนปิด"
        screens: [lifecycle]
        defaultStatus: backlog
  - id: g6
    title: "GROUP 6 — Billing"
    priority: P3
    features:
      - id: billing
        title: "Billing & Invoices"
        priority: P3
        goal: "สรุปค่าใช้จ่าย, ออก invoice, tracking การชำระ"
        screens: [billing]
        notes: "รอ payment gateway integration ก่อน"
        defaultStatus: backlog
  - id: g7
    title: "GROUP 7 — Features นอก Mockup"
    priority: P2
    features:
      - id: email-notify
        title: "Email / Notification Service"
        priority: P2
        goal: "ส่ง email / push notification เมื่อ event สำคัญเกิดขึ้น"
        beOnly: true
        notes: "background queue, retry, template-based"
        defaultStatus: backlog
      - id: webhook
        title: "Webhook Outbound"
        priority: P2
        goal: "ส่ง event ออกไปยัง external system เมื่อ order status เปลี่ยน"
        beOnly: true
        notes: "signature verification, retry with backoff, delivery log"
        defaultStatus: backlog
      - id: tenant-settings
        title: "Tenant Settings"
        priority: P2
        goal: "Admin ตั้งค่าระดับ tenant — timezone, currency, branding, feature flags"
        notes: "BE + FE (หน้า Settings)"
        defaultStatus: backlog
      - id: api-keys
        title: "API Keys / Integrations"
        priority: P3
        goal: "สร้างและจัดการ API Key สำหรับ external integration"
        notes: "BE + FE"
        defaultStatus: backlog
      - id: i18n
        title: "Multi-language / i18n"
        priority: P3
        goal: "รองรับ TH/EN ครบทุก screen"
        notes: "เริ่มทำควบคู่ทุก feature — FE (ส่วนใหญ่) + BE (error messages)"
        defaultStatus: backlog
---

# Backlog Features

This file is the source of truth for `tasks/backlog.html`. The HTML is regenerated
from the YAML frontmatter above via `node scripts/gen-backlog.mjs` and committed
alongside this file so the dashboard renders without a build step.

To add/edit/remove features:

1. Edit the YAML frontmatter above (preserve indentation — 2 spaces, no tabs).
2. Run `node scripts/gen-backlog.mjs`.
3. Verify the output by opening `tasks/backlog.html` in a browser.
4. Commit both files together.

Field reference per feature:

- `id` (string, required) — stable identifier; used as localStorage key.
- `title` (string, required) — card heading.
- `priority` (P1 | P2 | P3, required) — colour-coded badge.
- `goal` (string, required) — short description rendered as the card body.
- `screens` (string[], optional) — tag list rendered below the goal.
- `spec` (string, optional) — relative path to spec; displayed as a tag.
- `specReady` (bool, optional) — if true, spec tag uses the ready (✅) variant.
- `beOnly` (bool, optional) — adds the "BE only" tag.
- `notes` (string, optional) — italic note rendered below tags.
- `defaultStatus` (backlog | sprint | done) — initial status when no
  localStorage entry exists for this feature.

Groups themselves have `id`, `title`, `priority`, and a `features` array.
