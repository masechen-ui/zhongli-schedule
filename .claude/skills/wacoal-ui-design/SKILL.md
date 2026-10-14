---
name: wacoal-ui-design
description: "華歌爾專案的 UI/UX 設計與前端優化流程。Automatically use when asked to design, redesign, beautify, review, implement or fix UI, dashboards, reports, printable layouts, mobile/responsive screens, tables, charts, forms, KPI cards, typography, color, navigation, or interaction. Do not trigger for backend, authentication, data sync, POS, calculation-only requests."
---
# Wacoal UI Design Skill V1 — Cross-Project

## Auto-activation
Use for 美化、改版、UI、UX、排版、圖表、手機版、響應式、設計稿、介面、卡片、表單、表格、報告版型、前端使用體驗. Trigger based on the task's meaning, not a literal phrase. Claude Code can explicitly use `/wacoal-ui-design`; Codex can select `wacoal-ui-design`.

## Before changes
1. Read `README.md`, applicable project master / agreed specs, `docs/WACOAL_UI_DESIGN_SYSTEM.md`, existing markup, CSS, JS/TS and package/config files. Follow approved existing functions, branding and UX decisions.
2. Determine user job and critical information: current performance, target, YoY, rate, gap, timestamp, exceptions or schedule state as appropriate. Preserve correct units and Taiwan/Traditional Chinese display.
3. Respect existing project stack: HTML/CSS/JS single-page projects should NOT be force-migrated to React or install new packages; React projects reuse existing components/design tokens.

## Design route
- For a broad new UI or redesign, create 2–3 sufficiently different layout approaches and preview them if tools allow. For a specific small fix, take the best focused approach without needless variants.
- Default to calm enterprise UI, strong hierarchy, readable numbers and touch-friendly mobile layout; follow shared tokens and product-specific exceptions.
- Superdesign, Better Design, Design Taste, UI Layouts and Motion Primitives are **optional reference/tools**, not installed by this skill. Call an available tool only when explicitly installed/authorized. Otherwise do UI design using project-native code. Never send private business data to design services by default.
- Avoid Canvas/WebGL, shader backgrounds, cursor animation, heavy animation and new dependencies for operating/reporting screens.

## Safety
- UI-only request may change front-end visual code, styles, responsive layout and accessible interactions ONLY. Do NOT change metrics, sales logic, target/forecast, comparison-day mapping, branch/store ownership, source precedence, Firestore schema/security, Google Sheets, POS/POSEX, API, cloud sync, import/export data contracts or permissions.
- Missing/unreported/stale is not zero; keep source cutoff and loading/error labels.
- Use a branch or PR for coding changes; do not merge or publish merely because a UI request was made. Respect user authorizations. Never check in credentials, IDs, raw operational data or secrets.

## QA checklist
- Verify **390px mobile**, **768px tablet**, **1440px desktop**, including 320px overflow edge case if relevant.
- 44px touch targets; legible Traditional Chinese; tabular numbers; data/table scroll indicated; logical filter layout and clear selected states.
- Keyboard focus, headings, screen-reader labels, WCAG AA contrast targets, `prefers-reduced-motion`; color never the only status signal.
- Run existing lint, tests and build if available. Compare data output to baseline to confirm no metric changes. Clearly report what was actually checked; never claim screenshot validation or production deployment without evidence.

## Output
State chosen direction, changed files, mobile/desktop considerations, checks, any unavailable design tools, branch/PR, and accurate deployment status.
