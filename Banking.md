---
avatar_id: "fsi-banking-card-management"
avatar_name: "Leo — The FSI Banking Card Servicing Strategist"
industry: "Financial Services (Banking)"
initiative_name: "Card Management — Lock/Unlock, Replace, Deactivate Consolidation"
jira_keys: ["TDI-16600", "CREQ-6064", "CREQ-7588", "CREQ-7484"]
status: "In Design — CREQ-6064 blocked on an engineering decision (reuse BLOCKED status vs. add new LOCKED status in PortX)"
stakeholders:
  design_lead: "Selva Kumar"
  product_manager: "Sandra McGrath"
  engineering_lead: "Lux Lishi Xiao"
  target_customers: ["Hernâni Alves (sponsor)", "Agent-facing Client 360 users"]
abstract_patterns:
  - "#single-entry-point-consolidation"
  - "#status-aware-action-filtering"
  - "#sequence-driven-branching"
  - "#target-selection-before-action"
  - "#locked-field-as-policy-enforcement"
  - "#irreversibility-gated-confirmation"
sibling_initiatives_to_refer:
  - "Subscription Lifecycle Servicing (Reason: Shares status-aware action filtering — only valid lifecycle transitions are ever shown, not grayed out)"
  - "Returns and Exchanges (Reason: Shares sequence-driven branching — the backend's current state decides the outcome, not a UI 'reason' field alone)"
  - "Manual Claim Association (Reason: Shares single-entry-point consolidation — one picker + one panel instead of N parallel mini-flows)"
---

# 🏷️ Financial Services (Banking) — Card Management (Lock/Unlock, Replace, Deactivate)

## 📖 Project Context (The Story)

### 1. The Problem We Needed to Solve (Context)
* **The Opportunity:** The legacy Client 360 UI bundled Deactivate and Replacement into a single screen and buried Lock/Unlock entirely, while Activate/Deactivate lived in a client-level "All actions" menu that had no way to say *which* card it meant once a client had more than one card on file. TDI-16600 and its three CREQs (6064, 7588, 7484) were opened to fix this: add real Lock/Unlock support, let an agent replace a card without being forced to deactivate it first, and lock down the shipping address used for replacements.
* **User & Business Pain Points:** Agents had no safe way to temporarily disable a card (lock) without the more drastic, harder-to-reverse act of deactivating it. "Replace" and "Deactivate" were conflated into one flow, so an agent fixing a worn-out card and an agent handling a stolen card went through the same screen with no distinction in outcome. The shipping address field was editable with nothing but a policy telling agents not to touch it — a training-dependent control, not an enforced one.
* **The Impact:** Left unsolved, this meant inconsistent card-number outcomes for customers (not knowing if a replacement kept their number or not), a real security gap on unused Inactive cards that could sit indefinitely activatable by anyone, and a shipping-address field whose only protection was agent discipline rather than the interface itself.

### 2. How We Solved It (Design Perspective & End-to-End Flows)
* **The Core UX Engine:** Pick the card first, then show one "Manage card" panel whose available tabs are generated entirely from that card's current backend status — never a static list with disabled states. Every action (Lock, Unlock, Activate, Deactivate, Replace) is a tab inside that one panel, so the "which card, what's allowed" logic exists in exactly one place instead of being re-implemented per status or per menu. Irreversible actions get a confirmation step; reversible ones don't. Fields that must never be edited (shipping address) are made genuinely non-editable, not just policy-protected.
* **Detailed User Flows:**
  * **Flow 1 — Standard Active-card replacement (wear and tear):** Agent selects the card from the client's card list, opens "Manage card," lands on the Replace tab. They choose reason "Worn/damaged," the panel shows "Same card number will be reprinted. Original card stays active until the new one is activated," the shipping address field is visible but locked (grayed background, border token, lock icon) and pre-filled from the address on file. No reason-driven dropdown branching is needed beyond this one choice — everything else (ordering, notifying the customer) is identical regardless of reason.
  * **Flow 2 — Lost/Stolen, sequence-dependent path:** Agent picks "Lost/Stolen" as the reason on the same Replace tab. The panel now says a new card number will be issued, not a reprint, and reminds the agent to tell the customer their saved payment methods and autopay will need updating. If the agent instead needs the card fully shut off first, they go to the Deactivate tab, confirm (Deactivate is the one action that requires "are you sure?"), and the backend status flips to Closed; a subsequent Replace off of a Closed card is not offered, because Replace is read directly from current status, not from a remembered "reason."
  * **Flow 3 — Dead-end Canceled/Closed state:** Agent opens "Manage card" on a Closed card and the panel shows no tabs at all — not grayed-out ones. Nothing about a Closed card can be reversed, so showing any action would be a dead click or worse, imply something happened. Absence is the deliberate answer, confirmed in design as the correct state rather than an oversight.

---

## 🧩 Pattern #01: Single Entry Point Consolidation

* **1. The Core Problem:**
  * A client can have several cards, so every action (Lock, Replace, Deactivate) needs a "which card" context, and building each action as its own menu item means re-implementing that same card-picker and status logic three or four times over.
* **2. The Winning UX Pattern:**
  * **Rule A (Visuals):** One "Manage card" panel per selected card; tabs inside it are the only place actions appear — no action is ever its own top-level menu entry.
  * **Rule B (Interactions):** Card-level actions (Lock/Unlock/Replace/Deactivate/Activate) are removed from the client-level "All actions" menu entirely, since that menu has no way to express which of several cards it targets; anything that needs a specific-card target lives only where the card is already known. Per-card kebab menus stay deliberately thin — just "See card details" and "Manage card" — rather than listing Lock/Replace/Deactivate directly in the kebab, so there is exactly one implementation of "what's valid for this status," not a duplicate one per entry point.
  * **Rule C (UX Copy):** Panel header is always the specific action name ("Lock card," "Replace card," "Deactivate card," "Activate card"), never a generic "Manage card" label once a tab is open, so the agent's target and intent are always explicit and unambiguous — this was itself a cleanup pass across roughly 15 panel-header nodes in the file.
* **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **Action-first entry (lead with "Lock/Replace/Deactivate" before picking a card):** An action isn't a real instruction until a card is known, so every subsequent tab or message risked being mislabeled or ambiguous about its target; it was initially argued for on dev-speed grounds and adopted as the team's final sync decision despite this tradeoff — a case where delivery speed, not UX purity, won, and that tradeoff is recorded here rather than hidden.
  * ❌ **Keeping Deactivate/Activate inside the global "All actions" menu:** Worked fine when a client had one card; broke down the moment a client had more than one, since the menu item never said which card it meant.
  * ❌ **Hand-built per-status kebab menus (a different menu list for Active, Blocked, Inactive):** Would have meant four separate places encoding the same business rules, which drift apart silently as the product evolves.
* **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** The panel's available tabs are derived from the PortX card-status enum (`ACTIVE, BLOCKED, CLOSED, INACTIVE, LOST_STOLEN, LOST, STOLEN, ATM`) read live at panel-open time, not cached from the card-list view.
  * **Accessibility (WCAG 2.2):** Tab order inside the panel must follow the same left-to-right order as the rendered tabs; focus returns to the triggering kebab/row when the panel closes.
  * **Telemetry & Reporting:** Every panel open logs which card (by status) and which tab was initially active, so action usage can be measured per status without extra manual notes.
* **5. Cross-Industry Reusability Tags:**
  * `#single-entry-point` `#side-panel-workflow` `#target-selection-before-action`

---

## 🧩 Pattern #02: Status-Aware Action Filtering

* **1. The Core Problem:**
  * Showing every action with the invalid ones grayed out still costs the agent a read-and-reject cycle on every single use, and a card's real-world physical state (not just its logical status) determines which actions genuinely make sense.
* **2. The Winning UX Pattern:**
  * **Rule A (Visuals):** The tab list itself changes per status — invalid actions are removed from the panel entirely, never shown disabled.
  * **Rule B (Interactions):** Active → Lock, Replace, Deactivate. Blocked → Unlock, Replace, Deactivate (being blocked only restricts use, not the physical card's condition — it can still be lost, damaged, or the customer may still want to close it, so everything except Lock itself still applies). Inactive → Activate, Replace, Deactivate (a never-activated card can still be damaged or lost in the mail, so Replace must exist; and a customer may simply never want it, so Deactivate must exist too — otherwise an unused, never-activated card sits indefinitely activatable by anyone, a real security gap). Closed → no tabs at all, since nothing is reversible from there.
  * **Rule C (UX Copy):** The Deactivate tab keeps one consistent label across every status (it's always "permanently end this card," never reversed in direction), while Lock/Unlock and Activate/Deactivate swap their own labels to reflect direction; the helper text, not the button label, is what carries the difference in consequence per status (e.g., "can't be undone" vs. "will never be able to be activated").
* **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **Showing all four actions grayed out based on status:** Still forces a read-and-reject click every time; a genuinely invalid action should be impossible to click, not merely discouraged.
  * ❌ **Dropping Deactivate from Inactive cards ("there's no reason to show it"):** Seemed logical at first glance but ignores that an unused card issued in error, a duplicate, or tied to a closed account needs a way to be permanently shut down — without it, that card remains activatable indefinitely.
* **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** CREQ-6064 (Lock/Unlock) is still blocked on an unresolved engineering decision — whether Lock reuses the existing `BLOCKED` status or a new `LOCKED` status is added to PortX. The design currently uses "Blocked" everywhere; every Blocked-labeled screen in this file should be treated as provisional until that decision lands.
  * **Accessibility (WCAG 2.2):** Removing tabs dynamically must not break tab-list ARIA relationships; the active tab's `aria-selected` state must be recalculated whenever the tab set changes after a status refresh.
  * **Telemetry & Reporting:** Log the full set of tabs offered (not just the one chosen) per panel-open, so filtering-rule correctness can be audited against real status distributions later.
* **5. Cross-Industry Reusability Tags:**
  * `#status-driven-ui` `#no-disabled-states` `#security-by-removal`

---

## 🧩 Pattern #03: Sequence-Driven Branching Over UI-Decided Branching

* **1. The Core Problem:**
  * Whether a replacement keeps the same card number or issues a new one, and whether a card ends up deactivated, must be driven by the banking core's actual current-status read at the moment of the call — never by a UI-level "reason" field that could drift out of sync with backend truth.
* **2. The Winning UX Pattern:**
  * **Rule A (Visuals):** Replace is one screen with a reason dropdown (e.g., "Worn/damaged" vs. "Lost/Stolen"), not two separate flows — the only thing that differs between reasons is the number-vs-reprint outcome and the stolen/lost status transition; everything else (ordering, address, notification) is identical.
  * **Rule B (Interactions):** Replace never changes card status by itself. A new card number requires Deactivate-then-Replace, in that order; a same-number reprint requires Replace alone, on an Active card. The sequence of calls, not any checkbox, is what the banking core reads to decide the outcome.
  * **Rule C (UX Copy):** The number-vs-reprint distinction is spoken out loud at the moment the agent picks the reason, and again on the success screen, specifically because it has a real operational consequence for the customer (a new number requires updating saved payment methods and autopay) that must never be silently implied.
* **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **A UI checkbox or toggle deciding "same number" vs. "new number":** Risked disagreeing with what the banking core would actually do, since the real determinant is the card's status at the moment Replace is called, not a flag set earlier in the UI.
  * ❌ **Two fully separate Replace flows (one for wear-and-tear, one for lost/stolen):** Would have duplicated the address-handling, ordering, and notification logic twice for a difference that is really just one dropdown choice and its downstream consequence.
  * ❌ **Confirmation step on every action (Lock, Unlock, Replace, Activate, Deactivate):** Lock/Unlock/Activate/Replace are all recoverable outcomes; only Deactivate is genuinely irreversible, so only Deactivate earns the extra "are you sure?" friction — adding it everywhere else would slow agents down with no safety benefit.
* **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** The shipping address field is rendered as a genuinely locked field (disabled-input visual tokens, lock icon) rather than an editable field with a "do not edit" instruction, per CREQ-7484 — confirmed against the Replace API, which requires the address and fails the call if it's empty; there is no admin toggle to override this, matching the existing mailing-address-on-file convention. An open item flagged but not yet confirmed with the PM: a bundled "Card replacement" toggle was re-added inside the Deactivate panel per a later team sync, meaning there are now two parallel paths that can initiate a replacement (the standalone Replace tab, and this toggle) — worth confirming as intentional rather than an accidental duplication.
  * **Accessibility (WCAG 2.2):** The locked address field must still be reachable by keyboard and announced as read-only (not simply omitted) via `aria-readonly`, so screen-reader agents don't assume it's missing entirely.
  * **Telemetry & Reporting:** Every Replace call logs the reason selected and the resulting number-vs-reprint outcome as separate fields, so any future mismatch between UI reason and backend outcome is visible in reporting rather than only discoverable by a customer complaint.
* **5. Cross-Industry Reusability Tags:**
  * `#sequence-over-ui-flag` `#backend-is-source-of-truth` `#locked-field-as-policy-enforcement`

---

## 🧭 Playbook: How to Design Similar Initiatives (Avatar Guidance)
When another designer asks your Avatar how to approach an initiative like this, guide them through these specific steps:
1. **Step 1 (Find the Shared Target Entity):** Does every action in scope need to know "which X" (which card, which subscription, which claim)? If so, that's your single entry point — build one picker + one panel, not N parallel menu items.
2. **Step 2 (Read Status, Don't Guess It):** Get the real enum of backend states before designing any tab or menu filtering. Map every state to the literal set of actions that are valid from it — including the states where nothing should be shown at all (a dead end is a valid, deliberate design, not a gap).
3. **Step 3 (Find the One Irreversible Action):** Of all the actions in the set, which one can't be undone? That's the only one that earns a confirmation step. Don't apply friction uniformly — it trains agents to click through warnings without reading them.
4. **Step 4 (Trace What Actually Decides the Outcome):** When two similar actions produce different downstream results (same number vs. new number; refund vs. store credit), find out whether that's a UI choice or something the backend decides from current state + call sequence. If it's the backend, the UI's job is to narrate the consequence clearly, not to encode the decision itself.
5. **Step 5 (Turn Policy Into Construction, Not Instruction):** If a field must never be edited for compliance reasons, make it structurally non-editable (disabled-input tokens, lock icon) rather than editable-with-a-warning. A rule enforced by the interface never depends on training.
6. **Step 6 (Reuse Existing Visual Language):** Status colors, badges, and labels should match what's already live elsewhere in the product (e.g., green already means "Active" on account badges) rather than inventing a new private color system for this one feature.
