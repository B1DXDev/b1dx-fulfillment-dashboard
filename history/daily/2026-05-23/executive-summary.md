---
# Executive Summary — Source of Truth
# Edit this file, then run: node scripts/gen-executive-summary.mjs
# Output: tasks/executive-summary.html

# ─── Hero ────────────────────────────────────────────────────────────────
reporting_period: "Week 20–21 · 16–31 May 2026"
report_date: "22 May 2026"
sprint_day: 7
sprint_days_total: 14
sprint_status: "ON TRACK"
sprint_period: "16 → 31 May 2026 · Q2 Sprint"
hero_subtitle: "Day 3 ของรอบ — เปิดรอบด้วย <b>PIM Wave 2 (BE)</b> 3 PRs ปิดรวด (01a/01b/01c-1) · <b>Courier Settings hardening</b> (CRSET-02A/02B/03/04 + E2E un-skip) · <b>Workspace ขยายเป็น 4-repo</b> (เพิ่ม Webhook Go) · 14 FE PRs + 7 BE merges + 50+ commits ใน 3 วัน · zero critical bugs"

metrics:
  tasks_done:    { val: "21", unit: "+", trend: "▲ 3 working days" }
  commits:       { val: "50", unit: "+", trend: "▲ 4 repos" }
  prs_merged:    { val: "21", unit: "+", trend: "▲ 14 FE · 7 BE" }
  quality:       { val: "100", unit: "%", trend: "▲ tests passing" }
  risk_level:    { val: "LOW", unit: "",  trend: "0 critical" }

# ─── Health Strip ────────────────────────────────────────────────────────
health:
  - { ring: "green", icon: "A+", label: "Project Health", value: "Excellent",       sub: "5 of 5 indicators green" }
  - { ring: "green", icon: "▲",  label: "Schedule",       value: "Ahead of plan",   sub: "PIM BE Wave 2 ครึ่งทาง · Day 3/14" }
  - { ring: "blue",  icon: "$",  label: "Team Velocity",  value: "3 devs · 4 repos", sub: "+Webhook (Go) ผูก 2026-05-18" }

# ─── Feature At-a-Glance ─────────────────────────────────────────────────
features:
  - { id: "01", name: "Order Mgmt<br>(OAD + Bulk)",         pct: 85, state: "active", status: "OAD-INT-01 PR #192 review" }
  - { id: "02", name: "Sync<br>Order",                       pct: 80, state: "active", status: "Live · stable" }
  - { id: "03", name: "PIM<br>(Product Master)",             pct: 85, state: "active", status: "BE 01a/01b/01c-1 ✓" }
  - { id: "04", name: "RBAC<br>(Permissions)",               pct: 90, state: "active", status: "PERM-FE-ORDERS-01 ✓" }
  - { id: "05", name: "Courier Settings<br>(CRSET-02/03/04)", pct: 85, state: "active", status: "02A/B/C/03/04 ✓ · 02D fix" }
  - { id: "06", name: "Webhook (Go)<br>4-repo onboarding",   pct: 15, state: "active", status: "Submodule + agent ✓" }

# ─── Timeline ────────────────────────────────────────────────────────────
timeline_title: "Delivery Timeline · 2026"
timeline_subtitle: "เส้นเวลาส่งมอบรายฟีเจอร์ · ตำแหน่งวันนี้ = กลางไตรมาส 2"
timeline:
  - { state: "done",    left: 8,  quarter: "Q1 ’26",           label: "Authen + RBAC",                          detail: "JWT · OTP · Permission" }
  - { state: "done",    left: 26, quarter: "Q2 ’26 · Apr",     label: "Sync Order",                              detail: "Marketplace + ERP · Live" }
  - { state: "done",    left: 48, quarter: "Q2 ’26 · Mid-May", label: "PIM FE + OAD + Profile",                  detail: "4 epics ปิด · 35+ PRs · 03–15 May" }
  - { state: "active",  left: 68, quarter: "Q2 ’26 · Now",     label: "PIM BE Wave 2 + Courier Hardening",       detail: "01a/01b/01c-1 ✓ · 01c WIP · CRSET-03/04 ✓ · +Webhook repo" }
  - { state: "planned", left: 88, quarter: "Q3–Q4 ’26",        label: "WMS + Finance + MP Webhooks",             detail: "Pick · Pack · Invoice · Shopee/Lazada/TikTok" }

# ─── KPI Cards ───────────────────────────────────────────────────────────
kpi:
  be_merges:     { label: "BE Merges (3 days)", val: "7",  unit: "/7", delta: "▲ PIM + Courier",        foot: "BE-01a/01b/01c-1 · CRSET-02A/B/03/04" }
  fe_prs:        { label: "FE PRs Merged",      val: "14", unit: "",   delta: "▲ PRs #193–206",         foot: "16–18 May (3 days)" }
  in_flight:     { label: "In-Flight",          val: "4",  unit: "",   delta: "◐ Day 3 / 14",           foot: "PIM-BE-01c · OAD-INT-01 · CRSET-02D · Webhook" }
  total_commits: { label: "Total Commits",      val: "50", unit: "+",  delta: "★ FE + BE + Worker + Webhook", foot: "50+ commits · 4 repos · 3 days" }

# ─── Donut ───────────────────────────────────────────────────────────────
donut:
  pct: 81
  heading: "Period to-date"
  subtitle: "26 tasks · 16–18 May (Day 3/14)"
  done_dash: "234 55"
  active_dash: "29 260"
  active_offset: -236
  review_dash: "14 275"
  review_offset: -267
donut_legend:
  - { cls: "done",    label: "Done · Merged",       value: 21 }
  - { cls: "review",  label: "In Review",            value: 1 }
  - { cls: "active",  label: "In Progress",          value: 3 }
  - { cls: "pending", label: "Queued (Wave 3 FE)",   value: 1 }

# ─── Team Achievements ───────────────────────────────────────────────────
achievements_title: "Team Achievements · 16–18 May"
achievements_subtitle: "contribution highlights per engineer (period to-date)"
achievements_meta: "3 ENGINEERS · 21+ TASKS · 3 DAYS"
achievements:
  - name: "Nuchit"
    role: "PIM BE Owner · Sync Worker · DevOps"
    avatar: "n"
    initial: "N"
    bg: "#f5f3ff"
    count: 9
    count_color: "var(--violet)"
    count_bg: "#ede9fe"
    items:
      - "✓ <b>PIM-BE-01a</b> — Product/Variant aggregate + 6 endpoints + 22 tests merged"
      - "✓ <b>PIM-BE-01b</b> — Categories + Brands + Attributes + permission seed merged"
      - "✓ <b>PIM-BE-01c-1</b> — Domain Event Dispatch + Audit + 8 handlers merged"
      - "✓ <b>PIM-FE-05 + 07 + 08</b> — Logistics + Audit tab + missing tests (#193/194/199)"
      - "✓ <b>PIM-E2E-01</b> — 37 PIM acceptance tests implemented (#197)"
      - "✓ <b>FE-OD-KPI + FE-DASH-STATCARD</b> — chart-bg StatCard rollout 5 dashboards (#202)"
      - "◐ <b>PIM-BE-01c</b> — Pricing/Channels/Audit + 49 tests (In Progress)"
  - name: "Srattha"
    role: "Courier Settings Owner"
    avatar: "s"
    initial: "S"
    bg: "#ecfdf5"
    count: 8
    count_color: "var(--emerald)"
    count_bg: "#d1fae5"
    items:
      - "✓ <b>CRSET-02A</b> — Real Stats Provider (Shipment EF) merged"
      - "✓ <b>CRSET-02B</b> — Redis cache + sliding-window rate limiter merged"
      - "✓ <b>CRSET-02C + 02C-E2E</b> — 26 unit + E1–E5 un-skip merged (#191/#203)"
      - "✓ <b>CRSET-03</b> — Real HTTP test connection + SSRF hardening merged"
      - "✓ <b>CRSET-04</b> — Credentials at create-time + audit event merged"
      - "✓ <b>FE courier creds mockup</b> — drawer all tabs + i18n (#204/205)"
      - "◐ <b>CRSET-02D-review-fix</b> — Confirmed plan"
  - name: "Wat"
    role: "RBAC · Specs · Workspace infra"
    avatar: "w"
    initial: "W"
    bg: "#fff7ed"
    count: 7
    count_color: "var(--amber)"
    count_bg: "#fef3c7"
    items:
      - "✓ <b>PERM-FE-ORDERS-01</b> — gate order actions by permissions (#206)"
      - "✓ <b>FONT-SIZE-STD-PREF-01</b> — design-system font tokens + scalable pref (#201)"
      - "✓ <b>BUG-FE-RETURN-600 + BUG-FE-EXPORT-01</b> — return validation + export fix"
      - "✓ <b>WS-WEBHOOK-SUBMODULE-01</b> — 4-repo expansion + webhook-coder agent"
      - "✓ <b>Auto-Proceed mode</b> — Pre-Build Plan Gate workflow update"
      - "✓ <b>Agent docs review</b> — 15/17 PR #24 findings resolved"
      - "◐ <b>OAD-INT-01</b> — E2E scaffold (PR #192 in review)"

# ─── Decisions Needed ────────────────────────────────────────────────────
decisions_title: "Decisions Needed from Leadership"
decisions_subtitle: "รายการที่ต้องการการตัดสินใจระดับบริหาร · เพื่อไม่ให้บล็อกความคืบหน้า"
decisions_meta: "2 ITEMS · DUE THIS BI-WEEKLY"
decisions:
  - num: 1
    q: "PIM-BE-01c — Pricing/Channels/Audit merge gate"
    ctx: "7 aggregates + Domain event dispatch infra + 15 audit handlers + 9 endpoints + 49 tests · effort revised M→L (6-8h) · in progress on <code>feat/pim-be-01c-pricing-channels-audit</code> · ปิดได้จะปิด PIM BE Wave 2 ทั้งหมด · unblock FE Wave 3 (Logistics/Audit tabs ใช้ data จริง)"
    due: "22 May 2026"
  - num: 2
    q: "Webhook (Go) — feature ตัวแรกที่จะ ship?"
    ctx: "Submodule + agent infra ผูกแล้ว 2026-05-18 (Shopee/Lazada/TikTok/Facebook/LINE async ingestion + RabbitMQ DLX-DLQ) · ต้องเลือก: OAuth shop bind ตัวแรก (Lazada มี code อยู่แล้ว) vs Order webhook end-to-end vs Marketplace health check rollout — เพื่อกำหนด scope sprint หน้า"
    due: "24 May 2026"

# ─── Active Work + Team Strip ────────────────────────────────────────────
active_work_title: "Active Work &amp; Team"
active_work_subtitle: "in-flight · 3 engineers"
active_work_meta: "4 TASKS"
active_work:
  - { id: "PIM-BE-01c",    name: "Pricing · Channels · Audit · Domain Events (49 tests)", owner: "Nuchit",  avatar: "n", initial: "N", badge_cls: "active", badge_label: "In Progress" }
  - { id: "OAD-INT-01",    name: "Order Action Dialog · E2E scaffold (PR #192)",          owner: "Wat",     avatar: "w", initial: "W", badge_cls: "review", badge_label: "In Review" }
  - { id: "CRSET-02D",     name: "Courier review-fix follow-up (Confirmed)",              owner: "Srattha", avatar: "s", initial: "S", badge_cls: "active", badge_label: "In Progress" }
  - { id: "WS-WEBHOOK-01", name: "Webhook submodule + agent + 4-repo docs",                owner: "Wat",     avatar: "w", initial: "W", badge_cls: "active", badge_label: "Active" }

team:
  - { name: "Nuchit",  role: "PIM BE · Sync Worker · DevOps",   avatar: "n", initial: "N", count: 9 }
  - { name: "Srattha", role: "Courier (CRSET-02/03/04)",         avatar: "s", initial: "S", count: 8 }
  - { name: "Wat",     role: "RBAC · Specs · Workspace infra",   avatar: "w", initial: "W", count: 7 }

# ─── Shipped This Period ─────────────────────────────────────────────────
shipped_title: "Shipped This Period (16–18 May)"
shipped_subtitle: "grouped by epic · Day 3 of 14"
shipped_meta: "21+ MERGES · 3 DAYS"
shipped:
  - { icon: "✓", title: "PIM-BE-01a · Product Aggregate",                         desc: "Domain + Application + Infra + 6 endpoints + 22 tests",                       owner: "NUCHIT" }
  - { icon: "✓", title: "PIM-BE-01b · Categories/Brands/Attributes",              desc: "3 aggregates + permission seed + DI restore",                                 owner: "NUCHIT" }
  - { icon: "✓", title: "PIM-BE-01c-1 · Domain Event Dispatch + Audit",           desc: "infra + 8 handlers + EventDispatchInterceptor",                               owner: "NUCHIT" }
  - { icon: "✓", title: "PIM-FE-05/07/08 · Logistics + Audit tabs",               desc: "PRs #193/194/199/200 · weight/dimensions + diff drawer + revert",             owner: "NUCHIT" }
  - { icon: "✓", title: "PIM-E2E-01 · 37 acceptance tests",                       desc: "PR #197 · full PIM coverage",                                                  owner: "NUCHIT" }
  - { icon: "✓", title: "CRSET-02A/02B · Real Stats + Redis cache",               desc: "Shipment EF query + sliding-window rate limiter + Testcontainers",            owner: "SRATTHA" }
  - { icon: "✓", title: "CRSET-02C + 02C-E2E · 26 unit + E1–E5 un-skip",          desc: "PRs #191/203 · drawer dirty-state regression hardening",                      owner: "SRATTHA" }
  - { icon: "✓", title: "CRSET-03 · Real HTTP test connection + SSRF",            desc: "HttpCourierAdapter + private/loopback guard",                                  owner: "SRATTHA" }
  - { icon: "✓", title: "CRSET-04 · Credentials at create-time + audit",          desc: "+ FE Credentials tab UI (PRs #204/205)",                                       owner: "SRATTHA" }
  - { icon: "✓", title: "PERM-FE-ORDERS-01 · Gate order actions",                 desc: "PR #206 · permission-driven UI",                                               owner: "WAT" }
  - { icon: "✓", title: "FONT-SIZE-STD-PREF-01 · Design-system tokens",           desc: "PR #201 · scalable user pref + percentToTextScale",                            owner: "WAT" }
  - { icon: "✓", title: "BUG-FE-RETURN-600 + BUG-FE-EXPORT-01",                   desc: "PRs #195/196 · return validation + xlsx/PDF export fix",                       owner: "WAT" }
  - { icon: "✓", title: "FE-OD-KPI-CARDS-01 + FE-DASH-STATCARD-02",               desc: "PR #202 · chart-bg StatCard rollout 5 dashboards",                             owner: "NUCHIT" }
  - { icon: "✓", title: "WS-WEBHOOK-SUBMODULE-01 · 4-repo workspace",             desc: "+Webhook (Go) submodule + webhook-coder agent + trigger flow doc",            owner: "WAT" }
  - { icon: "◐", title: "PIM-BE-01c · Pricing/Channels/Audit",                    desc: "7 aggregates + 49 tests · In Progress",                                         owner: "NUCHIT" }

# ─── Milestones ──────────────────────────────────────────────────────────
milestones_title: "Upcoming Milestones"
milestones_subtitle: "2026 roadmap"
milestones_meta: "H2 2026"
milestones:
  - { when: "22 May ’26",   desc: "<b>PIM-BE-01c</b> merge — Pricing/Channels/Audit closes PIM BE Wave 2 ทั้งหมด" }
  - { when: "Late May ’26", desc: "<b>Webhook (Go) Phase 1</b> — เลือก feature แรกที่จะ ship (OAuth bind vs order webhook vs health check)" }
  - { when: "Jun ’26",      desc: "<b>Order SKU Match Wave 2</b> · <b>Webhook MP integration</b> Shopee/Lazada/TikTok order events" }
  - { when: "Q3 ’26",       desc: "<b>WMS Operations</b> · Inbound · Pick · Pack · Ship · Return" }
  - { when: "Q4 ’26",       desc: "<b>Finance &amp; Reports</b> · Promotion Engine · Compliance/Invoice" }

# ─── Risks ───────────────────────────────────────────────────────────────
risks_title: "Risks &amp; Mitigation"
risks_subtitle: "tracked &amp; owned"
risks_meta: "3 ITEMS"
risks:
  - level_cls: "med"
    level_label: "MED"
    desc: "<b>PIM-BE-01c effort blow-up</b> — revised M→L (6-8h) · 7 aggregates + dispatch infra + 49 tests"
    own: "Owner: Nuchit · Mitigation: 01c-1 (event dispatch) แยก merge แล้ว · ใช้ pattern เดียวกับ 01a/01b · 100% test green"
  - level_cls: "low"
    level_label: "LOW"
    desc: "<b>Webhook (Go) scope undefined</b> — submodule ผูกแล้วแต่ feature แรกยังไม่เลือก"
    own: "Owner: Wat · Decision Due 24 May · trigger-flow doc พร้อม · มี code Lazada/OAuth บางส่วนใน repo แล้ว"
  - level_cls: "low"
    level_label: "LOW"
    desc: "<b>Process drift</b> — direct-push hardening (5 วันที่ผ่านมาสะอาด)"
    own: 'Owner: Nuchit · Mitigation: CLAUDE.md hard-stop rule · auto-proceed mode + explicit "build" trigger ลด drift'

# ─── Takeaway ────────────────────────────────────────────────────────────
takeaway_label: "Executive Summary &amp; Asks"
takeaway_sub: "Status · Direction · Support Needed"
takeaway_body: |
  เปิดรอบ Bi-weekly ใหม่ Day 3/14 — ทีม 3 คนส่งมอบเร็วผิดปกติ: <span class="hi">PIM BE Wave 2</span> ปิด 3 PRs ใน 3 วัน (01a/01b/01c-1) ·
          <span class="hi">Courier Settings hardening</span> 5 รายการ (02A/02B/02C/03/04) · <span class="hi">RBAC + design-system</span> ปิด · workspace ขยายเป็น
          <span class="hi">4-repo</span> (เพิ่ม Webhook Go) — รวม 21+ merges · 50+ commits · zero critical bugs · 100% test green
takeaway_asks:
  - { lbl: "★ Wins",        txt: "<b>PIM BE Wave 2 ครึ่งทางใน 3 วัน</b> — Product + Categories/Brands/Attributes + Domain Events ปิด · CRSET-03 SSRF + CRSET-04 credentials hardening ครบ" }
  - { lbl: "▲ Focus Next",  txt: "<b>PIM-BE-01c merge</b> (22 May) → Webhook Phase 1 feature pick → MATCH Wave 2 → Q3 WMS" }
  - { lbl: "! Asks",        txt: "<b>2 decisions</b> — PIM-BE-01c merge gate (22 May) + Webhook feature แรกที่จะ ship (24 May)" }

# ─── Footer ──────────────────────────────────────────────────────────────
sources:
  - "tasks/assignments/2026-05-{nuchit,wat,srattha}.md"
  - "tasks/current-sprint.md"
  - "git log 16–18 May · 4 repos"
  - "gh pr list (FE #192–206)"
footer_conf: "BeOne Digital · Internal &amp; Confidential · v2.2 · 22 May 2026 · Day 7/14"
---

<!--
Body sections (markdown) are reserved for future narrative content.
All current data is captured in the YAML frontmatter above.
-->
