# 🏷️ Insurance — AI Patterns Package

**Persona Identity:** Clara — The Insurance Claims Architect

**Core Domain Rules:**
* **Guidewire Compliance (No Blind Auto-Linking):** Never automatically write notes to the first claim in a list when a customer has multiple claims. Only auto-confirm a claim if the customer has **exactly 1 claim** on their account.
* **Strict 1-to-1 Claim Notes:** Every claim discussed in an interaction must receive its own separate, distinct note (`"Only this claim receives these notes"`). Notes cannot be shared across multiple claims.
* **IVR Input is Context Only:** A claim number entered in the IVR is just a hint (`Shared by customer` badge). The adjuster must verify and manually confirm it before logging notes.
* **Channel-Agnostic Copy:** Always use *"conversation"* or *"interaction"* (never voice-only words like *"call"* or *"caller"*) so the design works across Voice, SMS, Chat, and Email.

---

## 📖 Project Context: Manual Claim Association (`DES-5008` / `TDI-17797`)

### 1. The Problem We Needed to Solve
In the Insurance Workspace (Member 360), claims adjusters handle interactions with policyholders (and third-party independent agents) who may have **zero, one, or multiple claims** on their account. 
Previously, the system defaulted to writing wrap-up notes to the **first claim in the list**, and there was only a single global Wrap-Up notes box. This caused three major issues:
1. **Compliance Risk:** If a customer had 2+ claims, notes were accidentally logged to the wrong claim in Guidewire ClaimCenter.
2. **Wrong IVR Inputs:** Customers often key in the wrong claim number in the IVR or ask about multiple claims on a single call.
3. **Multi-Claim Note Conflict:** The standard Wrap-Up box could not split and send separate notes to separate claims.

### 2. How We Solved It (Design Perspective & End-to-End Flows)
Instead of cluttering the Claims Card with inline checkboxes or forcing adjusters to do everything at Wrap-Up, we moved **Claim Confirmation and 1-to-1 Note-Taking into the Member 360 `Claim details` Side Panel**, guided by a **Dynamic Top Banner**:

* **Flow 1 — Unknown Customer / No Account Found (1 Claim):**
  * If the customer isn't matched automatically, the empty state shows `"No account found for this customer"` with two clear entry points: **`[Search by claim #]`** and **`[Search account]`** (filling up to 2 fields).
  * Once matched, if the customer has **only 1 claim**, the system automatically marks it **`Confirmed`** and the top banner prompts: `"Notes saved for 0 of 1 claim... [Write notes]"`.
* **Flow 2 — Customer Enters the Wrong Claim in IVR (2+ Claims):**
  * The claim entered in the IVR is tagged with a **`Shared by customer`** badge (not auto-confirmed).
  * The top banner asks the adjuster to verify it: `"The customer shared claim CL 3320-007... Check it's the one you're discussing [Check claim]"`.
  * In the Side Panel, the adjuster sees two buttons: **`[Confirm claim]`** or **`[Different claim]`**. Clicking **`[Different claim]`** lets them switch to the right claim (`CL 3342 002`) and confirm it.
* **Flow 3 — Customer Doesn't Know Their Claim Number (3+ Claims):**
  * When multiple claims exist and none was shared in the IVR, all claims start as **`Not confirmed`**.
  * The top banner prompts: `"Which claim are you discussing? Choose it, check the details, confirm it, then write your note [Choose a claim]"`. The adjuster reviews the list in the Side Panel and confirms the right one.
* **Flow 4 — Multiple Claims Discussed in One Conversation (1-to-1 Notes & Wrap-Up):**
  * After saving notes for Claim #1 (`✓ Notes saved`), the adjuster clicks **`Add another claim`** in the Side Panel to confirm Claim #2 and write a separate note for it.
  * The top banner tracks progress (`"Notes saved for 1 of 2 claim"` $\rightarrow$ `"All your claim notes are saved. Nothing else to do here. You can wrap up."`).
  * **At Wrap-Up:** The disposition panel shows a read-only **`Notes by claim`** summary (grouped by claim `#`) above an editable **`General notes`** box (`"Notes about a claim go in that claim's side panel. These are not sent to a claim"`).

---

## 🧩 Pattern #01: Multi-Item Association & 1-to-1 Scoped Note Logging

* **1. The Core Problem:** Adjusters need a compliant way to associate one or more specific claims with an active conversation and log separate 1-to-1 notes for each claim without sending notes to the wrong record in Guidewire.
* **2. The Winning UX Pattern:** 
  * **Default-Select by Count:** Auto-confirm when the customer has **1 claim**; require manual selection in the Side Panel when the customer has **2+ claims**.
  * **Read-Only Card Badges + Side-Panel Actions:** Keep the main Claims Card clean using read-only status pills (`Shared by customer`, `Confirmed`, `✓ Notes saved`), and house all confirmation buttons (`[Confirm claim]`, `[Different claim]`, `[Remove from conversation]`) and the **`Notes for this claim`** editor inside the **`Claim details` Side Panel**.
  * **Step-by-Step Progress Banner:** Use a dynamic banner at the top of the workspace (`[Choose a claim]` $\rightarrow$ `[Check claim]` $\rightarrow$ `[Write notes]` $\rightarrow$ `"All your claim notes are saved"`) so the adjuster always knows what to do next.
  * **Split Wrap-Up Summary:** Display saved claim notes as a read-only **`Notes by claim`** list at Wrap-Up, completely separated from **`General notes`**.
* **3. Why Other Options Failed (The "Graveyard"):**
  * **Inline Checkboxes on the Claims Card:** Rejected because checkboxes clutter the card UI and block future claim-level inline actions (e.g., updating claim status, sending info, or matching Banking card action patterns).
  * **Unknown-Number Search Inside the Claims Card:** Rejected because customer identification belongs at the workspace/account level via **Member Search** (`[Search by claim #]` / `[Search account]`), not inside an individual widget.
  * **Writing Claim Notes in the Standard Wrap-Up Box:** Rejected because a single Wrap-Up text box cannot map separate 1-to-1 notes to multiple claims in Guidewire.
  * **Auto-Confirming IVR Claims:** Rejected because customers frequently enter the wrong claim number in the IVR; human confirmation is required for compliance.
* **4. Technical & Edge-Case Guardrails:**
  * **Third-Party Callers:** Must support independent agents calling on behalf of a policyholder (showing the policyholder's claims while logging the caller in Conversation Details).
  * **Closed Claims:** Adjusters must be able to select and confirm `Closed` claims when customers call with questions about past payouts.
  * **Note Attachment Format & Metadata:** Confirm `.txt`/`.md` attachment support with backend engineering (Felix) and ensure confirming a claim logs the `interaction_id` $\leftrightarrow$ `claim_number` link even if no text note is written.
  * **Omnichannel Copy Check:** Always use `"Remove from conversation"` and `"Confirmed for this conversation"` (replace any leftover `"Remove from call"` strings).
* **5. Cross-Industry Reusability Tags:** `#multi-select` `#side-panel` `#1-to-1-notes` `#ivr-context-verify` `#audit-log` *(Directly reusable for **Banking** when associating one or more credit/debit cards or disputed transactions to an interaction, and **Retail** when logging notes across multiple orders/returns).*
