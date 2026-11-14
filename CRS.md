---
avatar_id: "crs-fsm-architect"
avatar_name: "Lucas — The CRS & Field Service CX Architect"
industry: "Commercial & Residential Services (CRS / Field Service Management)"
initiative_name: "CRS Workspace — FSM Integration, Dynamic Scheduling & Plan Enrollment"
jira_keys: ["CRS-WORKSPACE-01", "CRS-ARRIVAL-02", "CRS-PLAN-03"]
status: "Ready for Dev / Deployed"
stakeholders:
  design_lead: "Selva"
  product_manager: "Cedric / Luis"
  engineering_lead: "Workato & Frontend Integration Team"
  target_customers: ["ServiceTitan Enterprise Accounts", "FieldRoutes Pest & Home Service Accounts", "Multi-Property Residential & Commercial Clients"]
abstract_patterns:
  - "#adaptive-configuration-fallback"
  - "#zero-state-container-collapse"
  - "#api-payload-translation"
  - "#side-panel-drawer-workflow"
  - "#destructive-action-de-escalation"
sibling_initiatives_to_refer:
  - "Utilities: Customer Account & Outage Side-Panel (Reason: Shares zero-state container collapsing, explicit zero-balance fallbacks, and side-panel drawer safety rules)"
---

# 🏷️ Commercial & Residential Services (CRS) — CRS Workspace: FSM Integration, Dynamic Scheduling & Plan Enrollment

## 📖 Project Context (The Story)

### 1. The Problem We Needed to Solve (Context)
- **The Opportunity:** Call center agents handling live home and commercial service calls (HVAC, plumbing, electrical, pest control) needed a unified **CRS Workspace** side panel integrated directly with **ServiceTitan** and **FieldRoutes** to manage locations, recurring service plans, billing, and technician appointments without switching tabs.
- **User & Business Pain Points:**
  * **High Handle Times & Cognitive Load:** Agents faced empty greyed-out text boxes, redundant status badges, raw unformatted API strings, and multi-step modals during live customer calls.
  * **Fear of Accidental Cancellations:** Panel dismissal buttons labeled `"Cancel"` caused agents to hesitate out of fear they would cancel live customer work orders.
  * **API & Backend Constraints:** Certain ServiceTitan Business Units went live without configured arrival window presets, while Workato date calculations (such as `Next Service Date`) created severe screen-loading bottlenecks.
  * **Blind Plan Enrollment:** When upselling recurring service plans, selecting a plan template left the bottom half of the enrollment drawer blank, forcing agents to guess what services were covered.
- **The Impact:** Slowed call resolution times, agent hesitation during live scheduling, high risk of invalid backend API calls (e.g., attempting to hold one-time jobs), and missed subscription upsell opportunities.

### 2. How We Solved It (Design Perspective & End-to-End Flows)
- **The Core UX Engine:** **Progressive Disclosure & Context-Aware Adaptation.** Every UI component inside the narrow side panel adapts dynamically to the underlying FSM provider (ServiceTitan vs. FieldRoutes), account state, and Business Unit configuration—hiding empty containers, stripping redundant database noise, and translating raw API payloads into plain-English agent scripts.
- **Detailed User Flows:**
  * **Flow 1 — Dynamic Arrival Window Scheduling (Preset vs. Custom Fallback):**
    * When an agent opens **Schedule Appointment**, **Emergency Dispatch**, or **Reschedule** and selects a Business Unit *with* configured arrival windows, the UI displays equal-width preset pills (`8 AM - 12 PM`, `12 PM - 4 PM`, `4 PM - 8 PM`) alongside a compact `Preset | Custom` switcher.
    * If the customer requests a specific time range, switching to `Custom` swaps the pills in-place for `Arrival window start` and `Arrival window end` time pickers.
    * If a Business Unit has *no* presets configured in ServiceTitan, the switcher hides automatically and defaults directly to the Start/End time pickers.
  * **Flow 2 — Provider-Adaptive Service Plan Enrollment & Preview:**
    * Opening **Add service plan** omits `Start Date` (auto-defaulted to today by the backend) and `Payment Method` to accelerate live call entry.
    * For **ServiceTitan**, the form surfaces a **Business Unit** selector; for **FieldRoutes**, it swaps Business Unit for a selectable **Service Frequency** dropdown.
    * Selecting a plan template dynamically populates an **Included services (X)** card in the lower drawer, displaying a plain-English callout banner (*"Includes 4 seasonal home system inspections billed monthly."*) and a two-column list pairing cleaned service titles on the left with right-aligned visit frequency tags (`Seasonal`, `Every 4 months`).
  * **Flow 3 — Safe Appointment & Account Triage (Workspace Cards & Drawer):**
    * Location lists exceeding 3 properties collapse behind a `Show more (X)` accordion with redundant `Active` badges stripped.
    * In the Appointment Details drawer, dismissing the view uses **`Close`**, `Hold Series` is hidden on one-time jobs, `Reschedule` is hidden on unbooked reminders (tagged in light purple as **`Pending booking`**), and empty `Job Summary` or `Crew Notes` boxes collapse completely.

---

## 🧩 Pattern #[01]: Configuration-Driven Adaptive Inputs (Preset vs. Custom Fallback)

- **1. The Core Problem:**
  * Enterprise FSM clients frequently launch with partially configured Business Units (some having preset arrival windows and others having none), requiring the scheduling UI to seamlessly support both preset selection and manual time-range entry without cluttering the drawer.
- **2. The Winning UX Pattern:**
  * **Rule A (Visuals — Strict Button Hierarchy):** Assign distinct visual weights using only three design-system button styles: **Filled** (`#512DA8`) for the active preset slot, **Outlined** (white fill, slate border, equal flex width) for unselected presets, and a compact rectangular segmented tab (`Preset | Custom` in `#F1F5F9` with a white active tab) aligned to the section header.
  * **Rule B (Interactions — In-Place Swapping & Auto-Fallback):** Toggling `Preset | Custom` replaces the preset pill row in-place with `Arrival window start` and `Arrival window end` clock inputs—never stacking both controls simultaneously. When an unconfigured Business Unit is selected, hide the switcher and render the Start/End time inputs directly.
  * **Rule C (UX Copy & Iconography):** Strictly pair time pickers with clock icons and `HH:MM AM/PM` formatting (never calendar icons), and apply the exact same interaction model across *Schedule Appointment*, *Emergency Dispatch*, and *Reschedule*.
- **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **Stacking `+ Custom` Pill + Start/End Inputs Simultaneously:** Leaving preset pills visible above a highlighted `+ Custom` button and Start/End fields created competing active states, duplicated "Use standard windows" links, and forced vertical scrolling.
  * ❌ **Full-Width Toggle Switch Element:** Adding a standalone toggle switch row consumed valuable vertical real estate in the narrow 400px side panel and added unnecessary cognitive friction.
- **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** Handles Business Units returning empty `arrivalWindows: []` arrays from ServiceTitan without throwing validation errors or blocking appointment booking.
  * **Accessibility (WCAG 2.2):** Segmented tabs expose `role="tablist"` and `aria-selected` states; time inputs enforce keyboard-navigable time selection and clear focus rings.
  * **Telemetry & Reporting:** Logs whether the agent booked via a BU preset ID or a custom start/end timestamp override for operational dispatch auditing.
- **5. Cross-Industry Reusability Tags:**
  * `#adaptive-inputs` `#preset-vs-custom` `#fallback-state` `#side-panel-form`

---

## 🧩 Pattern #[02]: API Payload Translation & Plain-English Preview Cards

- **1. The Core Problem:**
  * Raw FSM subscription APIs return technical interval structures (`interval: "0"`, `interval_unit: "SEASONAL"`) and redundant tier suffixes (`"Gold Cooling Inspection – Monthly"`) that confuse agents on live calls when left unformatted or hidden.
- **2. The Winning UX Pattern:**
  * **Rule A (Visuals — Two-Column Scannable List without Custom Icons):** When a plan template is selected, populate the empty lower drawer space with an **Included services (X)** card featuring a subtle purple info callout banner at the top, cleaned service names left-aligned, and neutral `Tag` pills right-aligned under a `Visit frequency` header with subtle row dividers.
  * **Rule B (Interactions — Deterministic Payload Parsing):** Automatically translate API interval pairs into human-readable cadences (`interval_unit === "SEASONAL"` $\rightarrow$ `Seasonal`; `interval === "4"` + `MONTH` $\rightarrow$ `Every 4 months`) and strip trailing billing-tier suffixes (`– Monthly`, `– Annual`) from service titles so billing cadence and technician visit cadence never contradict each other.
  * **Rule C (UX Copy — Agent Script Banner):** Provide a 1-sentence plain-English summary inside the info callout (*"Includes 4 seasonal home system inspections billed monthly."*) that agents can read directly to customers on the phone.
- **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **Displaying Raw Service Names Alongside Frequency Tags:** Showing `"Gold Cooling Inspection – Monthly"` next to a `"Seasonal"` tag made agents pause and question whether the visit occurred monthly or seasonally.
  * ❌ **Inline Ragged Tags & Custom Emojis:** Placing tags immediately after variable-length service titles created a jagged, hard-to-scan layout, while custom trade emojis were unsupported by the core Figma design system.
- **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** Ignores unpopulated `duration` and `duration_type` API fields; dynamically adapts form inputs by showing **Business Unit** for ServiceTitan and **Service Frequency** for FieldRoutes while omitting `Start Date` (backend defaults to current date).
  * **Accessibility (WCAG 2.2):** High-contrast neutral tags and explicit column headers (`Included services` / `Visit frequency`) ensure screen readers announce the relationship between each service and its cadence.
  * **Telemetry & Reporting:** Tracks plan template selection events and enrollment completion rates by provider (`ServiceTitan` vs. `FieldRoutes`).
- **5. Cross-Industry Reusability Tags:**
  * `#api-translation` `#subscription-preview` `#plain-english-ux` `#multi-provider-form`

---

## 🧩 Pattern #[03]: Zero-State Collapse & Destructive Action De-Escalation

- **1. The Core Problem:**
  * Displaying empty read-only containers, high-contrast destructive buttons, and ambiguous dismissal labels in a dense workspace slows down call handling and causes agents to fear triggering unintended backend mutations.
- **2. The Winning UX Pattern:**
  * **Rule A (Visuals — Zero-State Collapsing & Explicit Fallbacks):** Completely hide `Job Summary`, `Crew Notes`, and `Included Benefits` boxes when backend arrays/strings are empty. Conversely, for critical financial fields, display explicit zero-state text (`-` for `Next Bill` with no plan; `No payment due` in muted gray for `$0` balances) so agents know data loaded properly.
  * **Rule B (Interactions — Context-Aware Action Gating):** Hide `Hold Series` on one-time jobs, hide `Reschedule` on unbooked plan reminders (distinguished via a light purple **`Pending booking`** badge), and collapse property lists beyond 3 locations behind a `Show more (X)` accordion.
  * **Rule C (UX Copy & Hierarchy — Safe Dismissal & Neutral Secondary Actions):** Rename drawer dismissal buttons from `"Cancel"` to **`Close`**, and style `Hold plan` and `Cancel plan` as neutral gray secondary buttons so they never visually overpower primary scheduling or payment CTAs.
- **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **Disabled/Greyed-Out Empty Text Boxes:** Leaving empty `Crew Notes` and `Job Summary` containers visible wasted vertical drawer height and led agents to believe fields were broken or locked.
  * ❌ **Labeling Drawer Dismissal as "Cancel" & Solid Red/Purple Hold/Cancel Buttons:** Agents avoided closing drawers out of fear of canceling live jobs, while prominent colored Hold/Cancel plan buttons drew accidental visual focus away from primary booking workflows.
  * ❌ **Computing `Next Service Date` via Workato:** Calculating next visit dates on the fly caused severe API latency; swapping the field to **`Service Frequency`** eliminated the bottleneck entirely.
- **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** Backend queries pre-filter inactive service locations to match ServiceTitan behavior, making per-card `Active` badges redundant.
  * **Accessibility (WCAG 2.2):** All 7 status badges (`Scheduled`, `Dispatched`, `Onsite`, `Completed`, `Held`, `Canceled`, `Pending booking`) adhere to WCAG contrast standards using distinct fill/text pairings rather than color alone.
  * **Telemetry & Reporting:** Captures drawer open/close durations, `Show more` expansion clicks, and context-gated action usage.
- **5. Cross-Industry Reusability Tags:**
  * `#zero-state-collapse` `#safe-dismissal` `#visual-hierarchy` `#status-badge-system`

---

## 🧭 Playbook: How to Design Similar Initiatives (Avatar Guidance)
When another designer asks your Avatar how to approach an initiative like this, guide them through these specific steps:
1. **Step 1 (Audit for False Affordances & Fear Triggers):** Never label a side-panel dismissal button `"Cancel"` when the underlying domain object (an appointment, order, or claim) can itself be canceled—always use **`Close`**, and neutralize destructive secondary actions (`Hold`, `Cancel Plan`) so primary workflows lead the visual hierarchy.
2. **Step 2 (Collapse Empty Containers, Clarify Zero-Balance States):** In narrow side panels, hide empty optional text blocks (`Notes`, `Summaries`, `Discounts`) completely to preserve vertical real estate, but use explicit confirmation copy (`No payment due`, `-`) on financial metrics so agents never mistake a `$0` balance for a failed API call.
3. **Step 3 (Design for Missing Backend Configurations):** Assume enterprise tenants will go live with incomplete settings (like missing arrival windows). Build an automatic fallback from preset pills to manual inputs, and use an in-place segmented switcher (`Preset | Custom`) rather than stacking duplicate controls.
4. **Step 4 (Translate Raw API Strings into Agent Scripts):** Never surface raw database concatenations or placeholder integers (`interval: 0`, `– Monthly` next to `Seasonal`) directly to frontline agents. Strip contradictory suffixes, right-align cadence tags for rapid scanning, and include a 1-line summary banner agents can read verbatim on live calls.