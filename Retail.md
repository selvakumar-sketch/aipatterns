---
avatar_id: "retail-cx-avatar"
avatar_name: "[TBC — avatar name]"
industry: "Retail & Consumer Goods"
initiative_name: "Returns and Exchanges in the Customer 360 Workspace"
jira_keys: ["TDI-15150", "TDI-13740", "DES-4920", "DES-5010"]
status: "In Design (DES-4920 due 2026-11-20)"
stakeholders:
  design_lead: "Hernâni Alves"
  product_manager: "Nicole Fernandes"
  product_designer: "Selva Kumar"
  engineering_lead: "[TBC]"
  target_customers: ["[TBC]"]
abstract_patterns:
  - "#valid-actions-only"
  - "#item-level-state"
  - "#one-story-two-statuses"
  - "#side-panel-workflow"
sibling_initiatives_to_refer:
  - "Retail: Order Status Labelling (Reason: one status per order with a reason line; same status wording and attention rules)"
  - "FSI Banking: Card Management, replace / lock / unlock / deactivate (Reason: action availability depends on object state and role; same permission and status pattern)"
---

# 🏷️ Retail — Returns and Exchanges in the Customer 360 Workspace

> **Draft for review.** Written from the design work so far (DES-4920, the Retail prototype, Nicole's PDD and exchanges prototype). Items marked **[TBC]** are not decided or not known. Nothing here is a Talkdesk commitment until Product and Engineering confirm it.
>
> **Industry note:** Returns and exchanges is a Retail initiative (TDI-15150 / TDI-13740). The banking equivalent in your work is Card Management, which is listed as a sibling above, not documented here.

## 📖 Project Context (The Story)

### 1. The Problem We Needed to Solve (Context)
* **The Opportunity:** Retail customers are planning major improvements to returns and exchanges. The Workspace needs an agent-side flow that starts from an order and covers returns, refunds and exchanges. The exchange scope is the open point (see Flow 3).
* **User & Business Pain Points:**
  * Refund and Cancel are offered on every order, including delivered or already refunded ones.
  * An agent opens several tabs to learn whether anything was returned, refunded or replaced.
  * Return state, payment state and delivery state are mixed together, so "Refunded" and "In progress" are easy to misread.
  * Partial returns (some items, some quantities) are hard to see.
* **The Impact:** Wrong or blocked actions during a live call, longer handle time, refunds offered when they are not possible, and agents who stop trusting what the screen says.

### 2. How We Solved It (Design Perspective & End-to-End Flows)
* **The Core UX Engine:** Start from the order, show only what is true for that order right now, and keep each kind of state (return, payment, delivery) in its own place. Every action either works or is locked with a plain reason. Complex detail lives in a side panel so the summary cards stay clean.
* **Detailed User Flows:**
  * **Flow 1 — Start from the order, delivered and eligible:**
    * The order side panel shows the one primary action that fits, for example "Start a return" on a delivered order. Other valid actions sit beside it; everything else is in "More".
    * Tabs: Overview, Items (n), Returns and refunds (n). An amber dot on the last tab means a return is open.
  * **Flow 2 — Action not allowed (blocked state):**
    * The action is not offered as a button. It appears in "More", locked, with the reason in plain words, for example "Cancel order: Already delivered." or "Cancel order: A return is in progress on this order."
    * If a return is already open, "View return" becomes the primary action and "Start a return" is locked with "A return is already in progress."
  * **Flow 3 — Return and exchange together (complex state):**
    * Target design: the returned item, the replacement item or variant, the reason and the financial result (even, upsell, downsell) are shown as one story. Return status and replacement shipment status stay separate.
    * **[TBC]** Whether agent-side exchange creation is in scope for the Workspace. The Exchanges PDD marks it MUST (TDI-13734 / 13741); the Packaged Agent PDD marks it WON'T HAVE.
    * Manage and track: hold and release a return, cancel a return. **[TBC]** exact states from the store system.

---

## 🧩 Pattern #01: Valid Actions Only, Blocked Actions Locked With a Reason

* **1. The Core Problem:**
  * Agents are shown actions that cannot work for the order, and learn that only after trying.
* **2. The Winning UX Pattern:**
  * **Rule A (Visuals):** Show valid actions as buttons (one primary, at most one more). Put the rest in a "More" menu. Locked items show a lock icon and a one-line reason.
  * **Rule B (Interactions):** Availability is decided from order state (delivered, shipped, paid or refunded, return open), not from a fixed list. A locked item cannot be activated but can be read.
  * **Rule C (UX Copy):** Reason is a short sentence that names the cause: "Already delivered.", "Payment is already refunded.", "Nothing here can be returned right now." No error codes, no store jargon.
* **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **Same actions on every order:** Refund and Cancel showed on delivered and refunded orders; agents could start flows that were never valid.
  * ❌ **Greyed-out buttons with no reason:** the agent cannot tell why, so they ask a colleague or retry.
  * ❌ **Hiding blocked actions completely:** the agent cannot tell whether the action exists, and cannot explain to the customer why it is unavailable.
* **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** Rules need order, payment, shipment and return state from the store system. **[TBC]** which of these the order endpoints already return.
  * **Accessibility (WCAG 2.2):** More is a menu button (`aria-haspopup`, `aria-expanded`). Locked items use `aria-disabled` and keep their reason as readable text, not only a tooltip. Menu closes on Escape. **[TBC]** focus return to the trigger.
  * **Telemetry & Reporting:** **[TBC]** log blocked-action views and reasons to learn which rules agents hit most.
* **5. Cross-Industry Reusability Tags:**
  * `#valid-actions-only` `#locked-with-reason` `#more-menu` `#permission-and-state`
  * Closest sibling: FSI Banking card management (lock, replace, deactivate depend on card state and role).

---

## 🧩 Pattern #02: Item-Level Return State (Partial Returns)

* **1. The Core Problem:**
  * A whole-order status cannot show that two items were returned, one partly, and three were kept.
* **2. The Winning UX Pattern:**
  * **Rule A (Visuals):** In the Items tab, items are grouped by shipment. A returned item shows "Return in progress · 1 of 1" (returned quantity of ordered quantity) under its name.
  * **Rule B (Interactions):** The Returns and refunds tab lists each return with its reference, reason, items and quantities, fees and expected refund, followed by the refunds issued.
  * **Rule C (UX Copy):** Quantities are always "n of m". Money shows fees and expected refund as separate lines.
* **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **One order-level status only:** hides which items are affected.
  * ❌ **Separate full pages for returns:** forces tab switching during a call; the side panel keeps the order in view.
* **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** Needs return line items with quantities. **[TBC]** availability and freshness.
  * **Accessibility (WCAG 2.2):** Tabs use `role="tablist"` / `role="tab"` with `aria-selected`; the open-return dot has an accessible label.
  * **Telemetry & Reporting:** **[TBC]**
* **5. Cross-Industry Reusability Tags:**
  * `#item-level-state` `#partial-fulfilment` `#side-panel` `#line-items`

---

## 🧩 Pattern #03: One Story, Two Statuses (Return and Replacement)

* **1. The Core Problem:**
  * In an exchange, the return of the old item and the shipment of the new one are two processes; mixing them into one status confuses the agent and the customer.
* **2. The Winning UX Pattern:**
  * **Rule A (Visuals):** Show returned item, replacement and money difference together in one card. Show return status and replacement shipment status as two labelled values, each with the store's own wording.
  * **Rule B (Interactions):** The financial result (even, upsell, downsell) is calculated and shown before the agent confirms. **[TBC]** who collects or refunds the difference.
  * **Rule C (UX Copy):** Each status says what it is about ("Return status: …", "Delivery status: …"). Use the store's status names, not invented ones.
* **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **All statuses as separate unlabelled tags:** customers' feedback was that they are hard to read and mix payment, delivery and return.
  * ❌ **One combined "exchange status":** needs Talkdesk to invent or calculate a status; the PDD rules this out.
* **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** Exchange events and fulfilment events come from the store system (per Nicole's exchanges prototype, via Shopify return and fulfilment events). **[TBC]** exact fields.
  * **Accessibility (WCAG 2.2):** Do not rely on colour alone to show an amber or red state; keep a text label.
  * **Telemetry & Reporting:** **[TBC]**
* **5. Cross-Industry Reusability Tags:**
  * `#one-story-two-statuses` `#store-wording` `#financial-summary`

---

## 🧩 Pattern #04: Order Side Panel as the Workflow Surface

* **1. The Core Problem:**
  * Agents need order detail, items, returns and actions without leaving the customer view.
* **2. The Winning UX Pattern:**
  * **Rule A (Visuals):** Header with order number and date, the order's statuses, then actions, then tabs. Summary cards (Customer details, Orders list) stay light.
  * **Rule B (Interactions):** "View return" and "Ref. details" open the panel on the right tab (Returns and refunds, Items). Item references can be copied one by one or all at once.
  * **Rule C (UX Copy):** Tab names are plain nouns: Overview, Items, Returns and refunds.
* **3. Why Other Options Failed (The "Graveyard"):**
  * ❌ **Four separate tabs for one order:** too many clicks for one question.
  * ❌ **Pop-ups for returns:** too small for return detail. **[TBC]** the PDD asks for a larger panel at small and medium widths.
* **4. Technical & Edge-Case Guardrails:**
  * **Integration Limits:** **[TBC]** panel width and layout behaviour at small and medium widths (Phase 2, TDI-18086).
  * **Accessibility (WCAG 2.2):** Panel is a dialog with a visible close button; Escape closes it. **[TBC]** focus trap and return-focus rules.
  * **Telemetry & Reporting:** **[TBC]**
* **5. Cross-Industry Reusability Tags:**
  * `#side-panel-workflow` `#tabs` `#ref-details-copy`

---

## 🧭 Playbook: How to Design Similar Initiatives (Avatar Guidance)
When another designer asks how to approach an initiative like this, guide them through these steps:
1. **Step 1 (Check state, not just object):** List the states of the object (delivered, shipped, paid, refunded, return open). Decide which actions are valid in each, and write the reason for every blocked one before drawing screens.
2. **Step 2 (Protect the summary cards):** Keep summary cards short. Put detail, items, returns and complex actions in the side panel.
3. **Step 3 (Keep statuses separate):** Do not merge return, payment and delivery into one status. Use the store's own names. Let the admin rename them, not Talkdesk.
4. **Step 4 (Map the data early):** Confirm with Engineering which return, refund, shipment and item-quantity fields the endpoints already return, and which exchange states exist. Hide what is missing instead of guessing.
5. **Step 5 (Check the sibling patterns):** Compare with Order Status Labelling (one status per order with a reason) and FSI Card Management (actions that depend on state and role) before inventing something new.
