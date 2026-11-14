---
avatar_id: "utilities-cx-architect"
avatar_name: "Felix — The Energy & Utilities CX Architect"
industry: "Utilities (Electric, Gas, Water & Municipal Services)"
initiative_name: "Utilities Workspace — Outage Triage, Meter Billing & Service Turn-On/Off"
jira_keys: ["UTIL-WORKSPACE-01", "UTIL-OUTAGE-02", "UTIL-BILLING-03"]
status: "Ready for Dev / Deployed"
stakeholders:
  design_lead: "Selva"
  product_manager: "Product & Utilities Vertical Lead"
  engineering_lead: "CIS & OMS Integration Team"
  target_customers: ["Investor-Owned Utilities (IOUs)", "Municipal Water & Power Providers", "Multi-Meter Commercial Accounts"]
abstract_patterns:
  - "#context-aware-dynamic-banners"
  - "#explicit-zero-balance-fallbacks"
  - "#multi-premise-progressive-disclosure"
  - "#safe-dismissal-hierarchy"
sibling_initiatives_to_refer:
  - "CRS (Field Service Management): CRS Workspace (Reason: Shares side-panel drawer patterns, 3-property accordion collapsing, and explicit zero-balance billing fallbacks)"
---

# 🏷️ Utilities — Utilities Workspace: Outage Triage, Meter Billing & Service Turn-On/Off

## 📖 Project Context (The Story)

### 1. The Problem We Needed to Solve (Context)
- **The Opportunity:** Customer Service Representatives (CSRs) handling high-volume utility interactions (power outages, gas leaks, high-bill inquiries, and move-in/move-out service requests) needed a streamlined workspace panel connected to Customer Information Systems (CIS) and Outage Management Systems (OMS).
- **User & Business Pain Points:**
  * **High Call Volume Spikes During Outages:** During weather events or grid outages, agents wasted critical seconds reading dense OMS logs or creating duplicate outage tickets for already-known grid events.
  * **Billing & Zero-Balance Ambiguity:** Customers calling to verify autopay or final move-out balances saw blank or `$0.00` fields without context, forcing agents to dig into ledger tabs to confirm whether an invoice was paid or failed to sync.
  * **Multi-Premise & Meter Clutter:** Landlords, property managers, and commercial accounts with 5–20+ service meters overwhelmed the side panel, pushing billing and service order actions below the fold.
- **The Impact:** Elevated Average Handle Time (AHT) during peak storm events, duplicate field crew dispatches, and preventable repeat calls regarding payment and service restoration status.

### 2. How We Solved It (Design Perspective & End-to-End Flows)
- **The Core UX Engine:** **Real-Time Status Surfacing & High-Contrast Triage.** Surface active grid/outage alerts immediately at the top of the workspace, enforce strict visual hierarchy on safety-critical actions, and apply progressive disclosure across multi-meter premises and billing ledgers.
- **Detailed User Flows:**
  * **Flow 1 — Active Outage Triage & Restoration Banner:**
    * When a customer's premise falls within a known OMS outage polygon, an alert banner pins automatically above the Premise Card displaying outage cause, affected crew status, and Estimated Time of Restoration (ETR), shifting the primary action from *"Report New Outage"* to *"Subscribe to SMS Restoration Updates"*.
  * **Flow 2 — Service Turn-On / Turn-Off (Start/Stop Service) Drawer:**
    * Agents initiate Start/Stop Service orders inside a dedicated side-panel drawer featuring premise selection, requested effective date (with same-day emergency validation), and automated meter-read requirements.
    * Dismissing the drawer uses **`Close`** to prevent confusion with canceling an active utility disconnection/reconnection order.
  * **Flow 3 — Billing, Autopay & Multi-Premise Management:**
    * Premises exceeding 3 active meters/addresses collapse behind a `Show more (X)` accordion.
    * The Billing Card explicitly displays **`No payment due`** for `$0` balances and **`-`** for `Next Bill` on closed/inactive accounts, while filtering ledger views to actionable open balances and recent payment statuses.

---

## 🧩 Pattern #[01]: Context-Aware Outage & Grid Status Banners

- **1. The Core Problem:**
  * Agents frequently log duplicate outage tickets during active grid events because OMS outage data is buried in secondary tabs rather than surfaced at the premise level.
- **2. The Winning UX Pattern:**
  * **Rule A (Visuals — Pinned Severity Callout):** Render a high-contrast status banner directly above the Premise Summary Card whenever an active OMS incident or planned maintenance window matches the customer's service point ID.
  * **Rule B (Interactions — Primary CTA Pivoting):** Dynamically pivot the primary workflow button from `Report Outage` to `Notify Customer on Restoration` (SMS/Voice opt-in) when a confirmed outage already exists, preventing duplicate ticket creation while still giving the agent an actionable resolution step.
  * **Rule C (UX Copy — Plain-English Restoration Script):** Display ETR (Estimated Time of Restoration) and crew status (`Crew Dispatched`, `Assessing Damage`) in plain, customer-ready language rather than raw OMS status codes.
- **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **Modal Pop-Up Warnings on Account Load:** Blocking pop-ups interrupted agent greeting scripts and required an extra dismissal click on every call.
  * ❌ **Raw OMS Telemetry Codes:** Showing internal feeder/substation codes caused confusion and forced agents to translate technical grid jargon on live calls.
- **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** Handles delayed ETR updates gracefully by displaying *"ETR Pending Crew Assessment"* rather than blank timestamps or `1970/01/01` fallback dates.
  * **Accessibility (WCAG 2.2):** Outage severity banners pair semantic icons and bold status labels (`Active Outage`, `Planned Maintenance`) with WCAG AA+ color contrast.
  * **Telemetry & Reporting:** Tracks duplicate outage deflection rate and SMS restoration opt-in conversions per call.
- **5. Cross-Industry Reusability Tags:**
  * `#dynamic-status-banner` `#duplicate-ticket-deflection` `#cta-pivoting` `#live-triage`

---

## 🧩 Pattern #[02]: Multi-Premise Progressive Disclosure & Explicit Billing Fallbacks

- **1. The Core Problem:**
  * Multi-premise utility accounts and ambiguous `$0` / blank billing states clutter the side panel and force agents to verify basic account standing across multiple CIS screens.
- **2. The Winning UX Pattern:**
  * **Rule A (Visuals — 3-Item Premise Cap & Explicit Zero States):** Display up to 3 active premises/meters by default and collapse the rest behind a `Show more (X)` accordion. In the Billing Card, replace ambiguous `$0.00` or blank fields with explicit status copy: **`No payment due`** (muted gray) when current balance is zero, and **`-`** for `Next Bill` when service is inactive/finalized.
  * **Rule B (Interactions — Actionable Balance Filtering):** Display positive overdue/current balances clearly with standardized status tags (`Paid`, `Pending`, `Overdue`, `Failed`), stripping unnecessary execution timestamps from invoice rows.
  * **Rule C (UX Copy — Premise Classification):** Pair each premise with clear service type indicators (`Residential`, `Commercial`, `Electric`, `Gas`, `Water`) while stripping redundant `Active` badges from pre-filtered active service points.
- **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **Unbounded Premise Lists:** Rendering all 15+ meters for commercial property managers pushed billing and service order controls completely off-screen.
  * ❌ **Blank Fields on Zero-Balance Accounts:** Leaving `Next Bill` or `Amount Due` blank caused agents to suspect CIS API timeouts rather than confirming a paid-in-full account.
- **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** API queries pre-filter inactive/closed service agreements by default, with an optional toggle only when agents investigate final bills.
  * **Accessibility (WCAG 2.2):** Accordion controls use `aria-expanded` attributes and maintain logical keyboard tab order across premises and billing actions.
  * **Telemetry & Reporting:** Logs `Show more` expansion frequency and billing ledger drill-down clicks.
- **5. Cross-Industry Reusability Tags:**
  * `#progressive-disclosure` `#explicit-zero-state` `#multi-premise` `#billing-clarity`

---

## 🧩 Pattern #[03]: Safe Drawer Dismissal & De-Escalated Service Disconnect Controls

- **1. The Core Problem:**
  * Service Turn-Off / Disconnect workflows carry high operational risk, and labeling drawer exit buttons `"Cancel"` causes agents to fear accidentally canceling live field orders.
- **2. The Winning UX Pattern:**
  * **Rule A (Visuals — Neutral Secondary Controls):** Style destructive or sensitive account actions (`Stop Service`, `Hold Payment Arrangement`, `Cancel Order`) as neutral gray secondary buttons so they never visually overpower primary service restoration or payment CTAs.
  * **Rule B (Interactions — Context-Aware Action Gating):** Disable or hide incompatible order actions (e.g., hiding `Reschedule Meter Read` once a technician is already `Onsite` or `Completed`) and collapse empty `Field Notes` / `Access Instructions` containers when null.
  * **Rule C (UX Copy — Unambiguous Exit Labels):** Strictly label all side-drawer dismissal buttons **`Close`** (never `"Cancel"`), reserving `"Cancel"` exclusively for explicit order-cancellation confirmation dialogs.
- **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **Using "Cancel" to Dismiss Side Drawers:** Agents hesitated to close Start/Stop service drawers out of fear they would void an active customer service request.
  * ❌ **Empty Field Instruction Boxes:** Displaying empty `Meter Access Notes` containers wasted vertical drawer space and misled agents into thinking notes failed to load.
- **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** Enforces cut-off time validation on same-day Turn-On/Turn-Off requests directly inside the drawer before submission to CIS.
  * **Accessibility (WCAG 2.2):** Status tags (`Scheduled`, `Dispatched`, `Onsite`, `Completed`, `Held`, `Canceled`, `Pending booking`) use distinct text labels and high-contrast backgrounds.
  * **Telemetry & Reporting:** Audits all service order creations, cancellations, and drawer abandonment events.
- **5. Cross-Industry Reusability Tags:**
  * `#safe-dismissal` `#destructive-de-escalation` `#zero-state-collapse` `#order-triage`

---

## 🧭 Playbook: How to Design Similar Initiatives (Avatar Guidance)
When another designer asks your Avatar how to approach a Utilities initiative, guide them through these specific steps:
1. **Step 1 (Deflect Duplicate Tickets Upfront):** Always surface active outage or maintenance banners above premise details and pivot the primary CTA from *"Report"* to *"Subscribe to Updates"* whenever an incident is already logged.
2. **Step 2 (Protect Against Fear-Based Hesitation):** Never use `"Cancel"` as a drawer close button when working with utility service orders—use **`Close`**, and style `Stop Service` / `Cancel Order` buttons in neutral gray so agents never trigger accidental disconnects.
3. **Step 3 (Cap Multi-Premise Lists & Clarify Zero Balances):** Collapse premises beyond 3 items behind a `Show more` accordion, hide empty note containers, and always display **`No payment due`** instead of blank fields on `$0` balances.
4. **Step 4 (Translate Grid & CIS Codes into Customer Scripts):** Replace internal OMS codes and raw billing intervals with plain-English ETRs and payment summaries that agents can read verbatim on high-stress calls.