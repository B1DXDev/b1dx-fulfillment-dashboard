---
# Executive Summary — Source of Truth
# Edit this file, then run: node scripts/gen-executive-summary.mjs
# Output: tasks/executive-summary.html

# ─── Hero ────────────────────────────────────────────────────────────────
reporting_period: "Week 21–22 · 19–27 May 2026"
report_date: "27 May 2026"
sprint_day: 12
sprint_days_total: 14
sprint_status: "ON TRACK"
sprint_period: "16 → 31 May 2026 · Q2 Sprint · Day 12/14"
hero_subtitle: "Day 12 ของรอบ — <b>PIM BE Wave 2 ปิดสมบูรณ์</b> (01c-3 + Media + product-level pricing/channels) · <b>PIM FE wire-up blitz</b> 21 May (10 PRs/วัน) · <b>Order Detail rebuild</b> (Fidelity 01a–f + Improve 98–106 + 27 May bug-bash 10 PRs) · <b>Webhook Capacity epic complete</b> (WH-CAP-01..09 + WH-RECON-01/02 ใน 7 วัน) · <b>+sompon-benz</b> เป็น engineer คนที่ 4 · <b>GH-Sync workflow</b> (4 slices) · ~114 PRs / ~159 commits / 4 repos / 4 devs / zero critical bugs"

metrics:
  tasks_done:    { val: "114", unit: "+", trend: "▲ 9 days · 4 repos" }
  commits:       { val: "159", unit: "+", trend: "▲ 5.4×/day" }
  prs_merged:    { val: "114", unit: "+", trend: "▲ 50 FE · 15 BE · 13 WH · 36 WS" }
  quality:       { val: "100", unit: "%", trend: "▲ tests passing" }
  risk_level:    { val: "LOW", unit: "",  trend: "0 critical · 22 in-flight" }

# ─── Health Strip ────────────────────────────────────────────────────────
health:
  - { ring: "green", icon: "A+", label: "Project Health", value: "Excellent",       sub: "5 of 5 indicators green" }
  - { ring: "green", icon: "▲",  label: "Schedule",       value: "Ahead of plan",   sub: "PIM Wave 2 ปิด · Webhook epic ปิด · Day 12/14" }
  - { ring: "blue",  icon: "$",  label: "Team Velocity",  value: "4 devs · 4 repos", sub: "+sompon-benz ผูก WH-CAP/RECON · 21 PRs ใน 7 วัน" }

# ─── Feature At-a-Glance ─────────────────────────────────────────────────
features:
  - { id: "01", name: "Order Mgmt<br>(Detail + OAD)",        pct: 90, state: "active", status: "Detail rebuild ✓ · OAD-INT-01 review" }
  - { id: "02", name: "Sync<br>Order",                       pct: 80, state: "active", status: "Live · stable" }
  - { id: "03", name: "PIM<br>(Product Master)",             pct: 85, state: "active", status: "BE Wave 2 ปิด ✓ · FE wire-up ✓" }
  - { id: "04", name: "RBAC<br>(Permissions)",               pct: 92, state: "active", status: "Order perm matrix ✓" }
  - { id: "05", name: "Courier Settings<br>(CRSET-02..04)",  pct: 95, state: "active", status: "02H + Detail fidelity ✓" }
  - { id: "06", name: "Webhook (Go)<br>Capacity + Recon",    pct: 70, state: "active", status: "WH-CAP-01..09 ✓ · WH-RECON-01/02 ✓" }
  - { id: "07", name: "Reconciliation<br>OMS UI",            pct: 10, state: "active", status: "RECON-UI-01..05 Draft (sompon)" }
  - { id: "08", name: "Workflow Infra<br>(QA-TC + GH-Sync)", pct: 85, state: "active", status: "12-col TC ✓ · Slices A–D ✓" }

# ─── Timeline ────────────────────────────────────────────────────────────
timeline_title: "Delivery Timeline · 2026"
timeline_subtitle: "เส้นเวลาส่งมอบรายฟีเจอร์ · ตำแหน่งวันนี้ = ปลาย Q2"
timeline:
  - { state: "done",    left: 6,  quarter: "Q1 ’26",           label: "Authen + RBAC",                          detail: "JWT · OTP · Permission" }
  - { state: "done",    left: 22, quarter: "Q2 ’26 · Apr",     label: "Sync Order",                              detail: "Marketplace + ERP · Live" }
  - { state: "done",    left: 42, quarter: "Q2 ’26 · Mid-May", label: "PIM FE + OAD + Profile",                  detail: "4 epics ปิด · 35+ PRs · 03–15 May" }
  - { state: "done",    left: 62, quarter: "Q2 ’26 · Late-May", label: "PIM Wave 2 + Order Detail + Webhook Cap", detail: "PIM 100% · OD rebuild · WH-CAP epic · 114 PRs · 19–27 May" }
  - { state: "active",  left: 80, quarter: "Q2 ’26 · Now",     label: "Recon UI + Drain in-flight",              detail: "RECON-UI-01..05 (sompon) · 22 in-flight tasks · sprint end 31 May" }
  - { state: "planned", left: 95, quarter: "Q3–Q4 ’26",        label: "WMS + Finance + MP Webhooks",             detail: "Pick · Pack · Invoice · Shopee/Lazada/TikTok" }

# ─── KPI Cards ───────────────────────────────────────────────────────────
kpi:
  be_merges:     { label: "BE Merges (9 days)", val: "15",  unit: "",  delta: "▲ PIM Wave 2 close",     foot: "PIM-BE-01c-3 + Media + pricing/channels + seeders" }
  fe_prs:        { label: "FE PRs Merged",      val: "50",  unit: "+", delta: "▲ Order Detail rebuild", foot: "Fidelity 01a–f + Improve 98–106 + bug-bash" }
  in_flight:     { label: "In-Flight",          val: "22",  unit: "",  delta: "◐ Day 12 / 14",          foot: "Drain risk · sprint end 31 May" }
  total_commits: { label: "Total Commits",      val: "159", unit: "+", delta: "★ 4 repos · 9 days",     foot: "WS 50 · FE 50+ · BE 20 · WH 39" }

# ─── Donut ───────────────────────────────────────────────────────────────
donut:
  pct: 81
  heading: "Period to-date"
  subtitle: "141 tasks · 19–27 May (Day 12/14)"
  done_dash: "234 55"
  active_dash: "31 258"
  active_offset: -234
  review_dash: "14 275"
  review_offset: -265
donut_legend:
  - { cls: "done",    label: "Done · Merged",       value: 114 }
  - { cls: "review",  label: "In Review",            value: 7 }
  - { cls: "active",  label: "In Progress",          value: 15 }
  - { cls: "pending", label: "Queued (RECON-UI)",    value: 5 }

# ─── Team Achievements ───────────────────────────────────────────────────
achievements_title: "Team Achievements · 19–27 May"
achievements_subtitle: "contribution highlights per engineer (period to-date)"
achievements_meta: "4 ENGINEERS · 114+ MERGES · 9 DAYS"
achievements:
  - name: "Nuchit"
    role: "PIM Owner · Workspace · DevOps"
    avatar: "n"
    initial: "N"
    bg: "#f5f3ff"
    count: 30
    count_color: "var(--violet)"
    count_bg: "#ede9fe"
    items:
      - "✓ <b>PIM-BE-01c-3</b> — Channel Mapping + Sync Log + audit handlers (BE #90) ปิด Wave 2"
      - "✓ <b>PIM-BE-MEDIA-01 + pricing/channels aggregate</b> — BE #99/100 product-level endpoints"
      - "✓ <b>PIM FE Wire-up blitz</b> 21 May — 10 PRs/วัน (Products, Pricing, Media, Variant, Brands, Audit, i18n)"
      - "✓ <b>PIM-SEED-PRODUCTS + Attribute + Nav</b> — BE #101–103 seeders + nav menu"
      - "✓ <b>Workspace exec presentation + dashboard regen</b> — WS #83/85"
      - "✓ <b>RECON-UI-01..05 epic plans + GH issues</b> — WS #159 (assigned to sompon-benz)"
      - "◐ <b>FE-BRAND-01</b> Gold logo refresh (Confirmed) · <b>FIX-COURIER-DETAIL-PRICING</b> (blocked on CRSET-02 main)"
  - name: "Srattha"
    role: "Order Detail · Courier · OMS Bug Fixes"
    avatar: "s"
    initial: "S"
    bg: "#ecfdf5"
    count: 38
    count_color: "var(--emerald)"
    count_bg: "#d1fae5"
    items:
      - "✓ <b>FE-OD-FIDELITY-01a..f</b> — 6 PRs (22 May) Order Detail visual fidelity sweep"
      - "✓ <b>FE-OD-IMPROVE-98..106</b> — 9 PRs (25–26 May) hero/state-machine/Info/History/Recipient/Payment/Notes"
      - "✓ <b>OMS bug-bash 27 May</b> — 10 PRs/วัน (i18n formatter SSOT, status chip, PLATFORM_COLOR, BUG-137/138/139/140/141)"
      - "✓ <b>CRSET-02H + 02I/J + courier nav</b> — BE #89/91/92/93/98 pickup + SLA + pricing contract"
      - "✓ <b>Courier follow-up FE</b> — 12 PRs (19–21 May) drawer + skeletons + i18n"
      - "◐ <b>IMPROVE-FE-FILTER-01</b> /orders filter bar (In Review FE #207)"
      - "⚠ <b>4 stale plans</b> — BULK-FE-06/07, BE-PRFSETTINGS-11, MATCH-WORKER-01 (≥16 วัน In Progress)"
  - name: "Wat"
    role: "Workflow · RBAC · CI Gates"
    avatar: "w"
    initial: "W"
    bg: "#fff7ed"
    count: 14
    count_color: "var(--amber)"
    count_bg: "#fef3c7"
    items:
      - "✓ <b>WORKFLOW-GH-SYNC Slices A/B/C/D</b> — auto-sync plan.md ↔ GH Issue ↔ Project (retry/backfill/drift)"
      - "✓ <b>QA-TC-01..06</b> — 12-col TC template + workflow guide + agent prompts + FE loader (#250)"
      - "✓ <b>Replaced 3-file task tracking with GH Issues SoT</b> — effective 2026-05-26"
      - "✓ <b>WS-WEBHOOK-SUBMODULE-01</b> — 4-repo expansion + webhook-coder agent (In Review #88)"
      - "✓ <b>RBAC: perm matrix for order list + MSW e2e session hook</b> — FE #244"
      - "✓ <b>QA-ORDER-SM-01 Block 0</b> — test artifacts + Done tracking (WS #91/93)"
      - "◐ <b>CI-BE-GATE-01 + CI-WEBHOOK-GATE-01</b> In Progress · <b>OAD quadruplet</b> (Dialog/Content/Button/Dropdown) In Progress"
  - name: "Sompon-Benz"
    role: "Webhook Owner · Reconciliation BE"
    avatar: "b"
    initial: "SB"
    bg: "#eff6ff"
    count: 21
    count_color: "var(--blue)"
    count_bg: "#dbeafe"
    items:
      - "✓ <b>WH-CAP-01..09 (full 9-slice epic ใน 7 วัน)</b> — capacity hardening complete"
      - "✓ <b>WH-CAP-01/03</b> — worker concurrency + HTTP timeouts + 1MB body cap (webhook #19/20)"
      - "✓ <b>WH-CAP-02/04</b> — publisher channel pool + Confirm mode + queue safety (DLX/DLQ TTL)"
      - "✓ <b>WH-CAP-05/06/07/08/09</b> — idempotency + bounded fanout + rate limiter + horizontal scale + async admin"
      - "✓ <b>WH-RECON-01/02</b> — reconciliation tenant isolation + error safety (webhook #28/29)"
      - "✓ <b>Go module rename</b> → b1dx-marketplace-webhook (webhook #23)"
      - "◐ <b>RECON-UI-01..05</b> — Reconciliation OMS UI epic (5 slices Draft · claimed 2026-05-27)"

# ─── Decisions Needed ────────────────────────────────────────────────────
decisions_title: "Decisions Needed from Leadership"
decisions_subtitle: "รายการที่ต้องการการตัดสินใจระดับบริหาร · เพื่อไม่ให้บล็อกความคืบหน้า"
decisions_meta: "2 ITEMS · DUE BEFORE SPRINT END"
decisions:
  - num: 1
    q: "Drain in-flight queue ก่อน sprint end (31 May) — freeze new starts?"
    ctx: "22 tasks In Progress/Review · Day 12/14 · เสี่ยง carry-over เข้า sprint หน้า · กลุ่มเสี่ยงสุด: 4 stale plans srattha (BULK-FE-06/07, BE-PRFSETTINGS-11, MATCH-WORKER-01 ≥16 วัน) + OAD-INT-01 (PR #192 stale 29 วัน in review) + wat OAD quadruplet (4 PRs WIP พร้อมกัน) · ทางเลือก: (a) freeze new starts → drain queue (b) close-out stale → carry-over เฉพาะ in-progress (c) ปล่อย — ยอม carry-over"
    due: "29 May 2026"
  - num: 2
    q: "RECON-UI-01 foundation merge gate — กำหนด priority ของ Recon UI epic?"
    ctx: "sompon-benz เพิ่ง claim 5 slices วันนี้ (RECON-UI-01..05) · 01 เป็น foundation (~3-4h) ที่ gate slice 02/03/04 · spec docs/specs/reconciliation.md ต้อง author ระหว่าง 01 · ถ้าเริ่มทันทีจะกินเวลา sprint หน้า · ทางเลือก: (a) ลุย sprint หน้า ทั้ง epic (~3-4 สัปดาห์) (b) ทำเฉพาะ 01+02 ก่อน เก็บ dashboard ไว้ Q3 (c) parallel กับ Webhook Phase 2 feature pick (OAuth bind/order webhook end-to-end)"
    due: "30 May 2026"

# ─── Active Work + Team Strip ────────────────────────────────────────────
active_work_title: "Active Work &amp; Team"
active_work_subtitle: "in-flight · 4 engineers · 22 tasks"
active_work_meta: "TOP 6 OF 22"
active_work:
  - { id: "OAD-INT-01",     name: "OAD E2E scaffold (stale PR #192, 29 วัน)",                owner: "Wat",         avatar: "w", initial: "W",  badge_cls: "review", badge_label: "In Review" }
  - { id: "WS-WEBHOOK-01",  name: "Webhook submodule + agent (PR #88)",                       owner: "Wat",         avatar: "w", initial: "W",  badge_cls: "review", badge_label: "In Review" }
  - { id: "IMPROVE-FE-FILTER-01", name: "/orders filter bar improvements (FE #207)",          owner: "Srattha",     avatar: "s", initial: "S",  badge_cls: "review", badge_label: "In Review" }
  - { id: "FE-BRAND-01",    name: "Gold logo refresh (Confirmed, kicked off วันนี้)",          owner: "Nuchit",      avatar: "n", initial: "N",  badge_cls: "active", badge_label: "In Progress" }
  - { id: "CI-BE-GATE-01",  name: "BE dotnet test CI gate",                                    owner: "Wat",         avatar: "w", initial: "W",  badge_cls: "active", badge_label: "In Progress" }
  - { id: "RECON-UI-01..05", name: "Reconciliation OMS UI epic (5 slices Draft)",              owner: "Sompon-Benz", avatar: "b", initial: "SB", badge_cls: "pending", badge_label: "Draft" }

team:
  - { name: "Nuchit",      role: "PIM · Workspace · DevOps",          avatar: "n", initial: "N",  count: 30 }
  - { name: "Srattha",     role: "Order Detail · Courier · Bugs",     avatar: "s", initial: "S",  count: 38 }
  - { name: "Wat",         role: "Workflow · RBAC · CI Gates",        avatar: "w", initial: "W",  count: 14 }
  - { name: "Sompon-Benz", role: "Webhook · Reconciliation BE",       avatar: "b", initial: "SB", count: 21 }

# ─── Shipped This Period ─────────────────────────────────────────────────
shipped_title: "Shipped This Period (19–27 May)"
shipped_subtitle: "grouped by epic · Day 12 of 14"
shipped_meta: "114+ MERGES · 9 DAYS"
shipped:
  - { icon: "✓", title: "PIM-BE-01c-3 · Wave 2 close",                            desc: "Channel Mapping + Sync Log + audit handlers (BE #90)",                          owner: "NUCHIT" }
  - { icon: "✓", title: "PIM-BE-MEDIA + product pricing/channels aggregate",     desc: "BE #99/100 — product-level endpoints",                                          owner: "NUCHIT" }
  - { icon: "✓", title: "PIM FE Wire-up blitz · 10 PRs/วัน (21 May)",              desc: "Products + Pricing + Media + Variant + Brands + Audit + 137 i18n keys",         owner: "NUCHIT" }
  - { icon: "✓", title: "PIM seeders + nav (Product/Attribute/Nav)",              desc: "BE #101/102/103 + Category Remark",                                              owner: "NUCHIT" }
  - { icon: "✓", title: "FE-OD-FIDELITY-01a..f · Order Detail visual fidelity",   desc: "6 PRs (22 May) — full mockup-fidelity sweep",                                    owner: "SRATTHA" }
  - { icon: "✓", title: "FE-OD-IMPROVE-98..106 · Order Detail rebuild",           desc: "9 PRs (25–26 May) hero/state-machine/Info/Status/Recipient/Payment/Notes",      owner: "SRATTHA" }
  - { icon: "✓", title: "OMS bug-bash 27 May",                                    desc: "10 PRs/วัน — i18n formatter SSOT · PLATFORM_COLOR · BUG-137/138/139/140/141",   owner: "SRATTHA" }
  - { icon: "✓", title: "CRSET-02H + 02I/J + courier nav",                        desc: "BE #89/91/92/93/98 — pickup + SLA + pricing contract + courier-code-exists",   owner: "SRATTHA" }
  - { icon: "✓", title: "WH-CAP-01..09 · Webhook Capacity epic (full 9 slices)",  desc: "concurrency + timeouts + DLX + idempotency + rate limiter + horizontal scale",  owner: "SOMPON-BENZ" }
  - { icon: "✓", title: "WH-RECON-01/02 · Reconciliation tenant isolation",       desc: "webhook #28/29 — tenant safety + error handling",                                owner: "SOMPON-BENZ" }
  - { icon: "✓", title: "Go module rename → b1dx-marketplace-webhook",            desc: "webhook #23 — workspace identity",                                                owner: "SOMPON-BENZ" }
  - { icon: "✓", title: "WORKFLOW-GH-SYNC Slices A/B/C/D",                        desc: "plan.md ↔ GH Issue ↔ Project auto-sync · retry · backfill · sub-issue linkage", owner: "WAT" }
  - { icon: "✓", title: "QA-TC-01..06 · 12-col TC standard",                      desc: "template + workflow guide + agent prompts + FE loader (#250)",                   owner: "WAT" }
  - { icon: "✓", title: "Replaced 3-file task tracking with GH Issues",           desc: "effective 2026-05-26 — single source of truth",                                  owner: "WAT" }
  - { icon: "✓", title: "WS-WEBHOOK-SUBMODULE-01 · 4-repo workspace",             desc: "+webhook-coder agent + trigger flow doc (In Review #88)",                        owner: "WAT" }
  - { icon: "✓", title: "RBAC: perm matrix for order list + MSW session hook",    desc: "FE #244",                                                                         owner: "WAT" }
  - { icon: "◐", title: "RECON-UI-01..05 · Reconciliation OMS UI epic",           desc: "5 slices Draft · claimed 2026-05-27 · foundation gates 02/03/04",                owner: "SOMPON-BENZ" }
  - { icon: "◐", title: "OAD-INT-01 · Order Action Dialog E2E",                   desc: "stale PR #192 · 29 วัน in review · needs merge decision",                        owner: "WAT" }

# ─── Milestones ──────────────────────────────────────────────────────────
milestones_title: "Upcoming Milestones"
milestones_subtitle: "2026 roadmap"
milestones_meta: "H2 2026"
milestones:
  - { when: "29 May ’26",   desc: "<b>Drain decision</b> — freeze new starts vs close-out stale · sprint ends 31 May" }
  - { when: "30 May ’26",   desc: "<b>RECON-UI-01 foundation</b> merge gate — กำหนด priority Reconciliation OMS UI epic" }
  - { when: "Jun ’26",      desc: "<b>Webhook Phase 2</b> — OAuth shop bind / order webhook end-to-end · <b>Recon UI 02–04</b>" }
  - { when: "Q3 ’26",       desc: "<b>WMS Operations</b> · Inbound · Pick · Pack · Ship · Return · <b>MP integration</b> Shopee/Lazada/TikTok" }
  - { when: "Q4 ’26",       desc: "<b>Finance &amp; Reports</b> · Promotion Engine · Compliance/Invoice" }

# ─── Risks ───────────────────────────────────────────────────────────────
risks_title: "Risks &amp; Mitigation"
risks_subtitle: "tracked &amp; owned"
risks_meta: "4 ITEMS"
risks:
  - level_cls: "med"
    level_label: "MED"
    desc: "<b>In-flight queue 22 tasks ที่ Day 12/14</b> — เสี่ยง carry-over · sprint ends 31 May"
    own: "Owner: Orchestrator · Mitigation: decision gate 29 May (freeze vs drain) · stale plans audit"
  - level_cls: "med"
    level_label: "MED"
    desc: "<b>4 stale srattha plans</b> — BULK-FE-06/07, BE-PRFSETTINGS-11, MATCH-WORKER-01 In Progress ≥16 วัน"
    own: "Owner: Srattha · Mitigation: close-out หรือ status update ภายใน 29 May · เริ่ม drain ทันที"
  - level_cls: "med"
    level_label: "MED"
    desc: "<b>OAD-INT-01 PR #192 stale</b> — claimed 2026-04-28 (29 วัน in review) + wat OAD quadruplet WIP พร้อมกัน"
    own: "Owner: Wat · Mitigation: merge decision PR #192 ก่อน · consolidate 4 OAD WIPs"
  - level_cls: "low"
    level_label: "LOW"
    desc: "<b>RECON-UI epic scope ยังไม่ confirm</b> — 5 slices Draft วันนี้ · spec ต้อง author ระหว่าง slice 01"
    own: "Owner: Sompon-Benz + Nuchit · Mitigation: priority decision 30 May · 01 foundation ~3-4h ก่อน gate 02/03/04"

# ─── Takeaway ────────────────────────────────────────────────────────────
takeaway_label: "Executive Summary &amp; Asks"
takeaway_sub: "Status · Direction · Support Needed"
takeaway_body: |
  ปิดท้ายรอบ Bi-weekly Day 12/14 — velocity ~5× จากช่วงต้นรอบ: <span class="hi">PIM BE Wave 2 ปิดสมบูรณ์</span> + <span class="hi">PIM FE wire-up blitz</span> 10 PRs/วัน · <span class="hi">Order Detail rebuild</span> 25+ PRs (Fidelity + Improve + bug-bash) ·
          <span class="hi">Webhook Capacity epic</span> 9 slices ปิดใน 7 วัน + Reconciliation tenant safety · <span class="hi">+sompon-benz</span> เป็น engineer คนที่ 4 (21 PRs/9 วัน) · <span class="hi">GH-Sync workflow</span> 4 slices ปิด
          — รวม 114+ merges · 159+ commits · 4 repos · 4 devs · zero critical bugs
takeaway_asks:
  - { lbl: "★ Wins",        txt: "<b>PIM 100% + Webhook Capacity epic ปิด</b> · Order Detail rebuild 25+ PRs · GH-Sync auto-flow + 12-col TC standard locked · onboard sompon-benz (21 PRs/9 วัน เริ่มเดือนนี้)" }
  - { lbl: "▲ Focus Next",  txt: "<b>Drain in-flight 22 tasks</b> (29 May) → <b>RECON-UI-01 foundation</b> (30 May) → Webhook Phase 2 feature pick → Q3 WMS" }
  - { lbl: "! Asks",        txt: "<b>2 decisions</b> — drain vs freeze (29 May) + RECON-UI epic priority (30 May)" }

# ─── Footer ──────────────────────────────────────────────────────────────
sources:
  - "tasks/plans/*/plan.md (22 In Progress/Review + 5 Draft)"
  - "git log 19–27 May · 4 repos · 159+ commits"
  - "gh pr list (FE 50+ · BE 15 · WH 13 · WS 36)"
  - "docs/standup/{nuchit,srattha,wat}/2026-05-*.md"
footer_conf: "BeOne Digital · Internal &amp; Confidential · v2.3 · 27 May 2026 · Day 12/14"
---

<!--
Body sections (markdown) are reserved for future narrative content.
All current data is captured in the YAML frontmatter above.
-->
