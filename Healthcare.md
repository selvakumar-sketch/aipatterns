---
avatar_id: "healthcare-avatar"
avatar_name: "Aria — The Healthcare & Life Sciences Navigator"
industry: "Healthcare & Life Sciences (HLS)"
initiative_name: "Post-Call Adverse Event Review & Potentially Reportable Event (PRE) Capture"
jira_keys: ["DES-5016", "DES-4656"]
status: "In Design (Q4 FY27)"
stakeholders:
  design_lead: "Hernâni Alves"
  product_manager: "Matt Yeoh / Luís Catarino"
  engineering_lead: "Felix"
  target_customers: ["Genentech", "Sentrex"]
abstract_patterns:
  - "#adverse-event-review"
  - "#post-call-adjudication"
  - "#potentially-reportable-events"
  - "#compliance-wrap-up"
  - "#non-proactive-ai"
sibling_initiatives_to_refer:
  - "Insurance: Manual Claim Association (DES-5008 - Scoped 1-to-1 notes mapping)"
  - "Retail: Return & Refund Status (TDI-13220 - Shared wrap-up page layouts)"
---

# 🏷️ Healthcare & Life Sciences — Post-Call Adverse Event Review

## 📖 Project Context (The Story)

### 1. The Problem We Needed to Solve (Context)
* **The Opportunity:** Pharmaceutical manufacturers run crucial Patient Support Programs (PSPs) in the US and Canada to guide patients through specialty drug enrollments, prior authorizations, and financial assistance.
* **User & Business Pain Points:** Legally, these companies are mandated by governing bodies (like the FDA or EMA) to report any **Adverse Events** (side effects) mentioned during a conversation. Previously, agents had to manually write down side effects, which was slow, easily missed, and prone to compliance audit failures.
* **The Impact:** Missing a single reported side-effect in a call log can result in massive legal compliance fines and patient safety risks. Standard contact center wrap-up states lacked the flexible UX capabilities to capture and structure these **Potentially Reportable Events (PRE)**, resulting in a "hacked" one-off HTML solution that was unmaintainable. 

### 2. How We Solved It (Design Perspective & End-to-End Flows)
* **The Core UX Engine:** To protect call center flow and respect compliance (some clients like Genentech previously avoided recording calls entirely), we chose a **Non-Proactive, Post-Call AI Adjudication model**. Instead of disruptive real-time alerts, Copilot silently transcribes the call, extracts side-effect mentions, and presents them as structured suggestions during the Wrap-Up phase for the agent to review, adjust, and confirm.
* **Detailed User Flows:**
  * **Flow 1 — Pre-Emit Adverse Event Review (No Event Found):**
    * When a call concludes and the AI detects no safety or side-effect topics, the standard Wrap-Up disposition panel renders with a green **`✓ No Adverse Events Detected`** badge, letting the agent complete standard wrap-up instantly.
  * **Flow 2 — Potentially Reportable Event (PRE) Flagged:**
    * If a side-effect is detected (even if the call's primary purpose was administrative), a warning banner appears at the top of the Wrap-Up screen: `"Potentially Reportable Event (PRE) Detected: Please verify the side effects mentioned by the patient."` with a primary **`[ Adjudicate PRE ]`** button.
  * **Flow 3 — Multi-Event Adjudication & Side-Panel Form:**
    * Clicking the button opens a dedicated **`Adverse Event Review`** side panel listing the extracted side effects (e.g., *"Nausea"*, *"Headaches"*), the matching transcript snippet as proof, and editable fields for Severity, Duration, and Reporter Type. 
    * The agent must explicitly check **`[ Confirm as Adverse Event ]`** or click **`[ Dismiss / False Positive ]`** before submitting the interaction.

---

## 🧩 Pattern #01: Post-Call Compliance Adjudication (PRE Capture)

* **1. The Core Problem:** 
  * Agents need an efficient, bulletproof way to review, verify, and log legally mandated side-effect reports (PREs) during wrap-up without interrupting live patient empathy or misrouting safety data to the FDA/EMA.
* **2. The Winning UX Pattern:** 
  * **Non-Proactive Post-Call Batching:** Put the AI in a silent observation mode during the live call, and surface all safety findings as structured **Adjudication Cards** only in the Wrap-Up state.
  * **Explicit Split Panel Verification:** Separate standard wrap-up codes on the left from the **`Adverse Event Review Panel`** on the right. The panel displays:
    1. The AI-flagged Symptom (e.g., *"Migraine"*).
    2. The exact **Transcript Snippet** showing when/how the patient described the symptom.
    3. Mandatory primary actions: **`[ Confirm & Log ]`** and **`[ Dismiss as False Positive ]`**.
  * **Dynamic Progress Banner:** Display an active count banner: `"Safety Review: Adjudicated 1 of 2 flagged events"` so the agent cannot close the interaction with unresolved compliance flags.
* **3. Why Other Options Failed (The "Graveyard"):** 
  * ❌ **Proactive Real-Time Call Alerts:** Rejected because showing popups to the agent mid-call was highly distracting, disrupted patient empathy, and legally forced the system to record and analyze 100% of calls live, violating strict client privacy rules.
  * ❌ **Pre-Filled Auto-Submission:** Rejected because AI transcription is not 100% accurate; auto-submitting unverified side effects to the FDA/EMA would lead to corrupted safety databases and massive audit penalties.
* **4. Technical & Edge-Case Guardrails:** 
  * **Zero-Recording Compliance:** Support text-based Chat, SMS, and Email pipelines where transcripts are processed instantly post-interaction, even for clients who have voice recording disabled.
  * **Telemetry & Audit Trail:** Every single agent decision (e.g., dismissing an AI suggestion as a false positive) must log a structured event (`agent_dismissed_pre`) with a required text reason box to maintain a legal audit trail.
  * **Unsaved Work Safety Lock:** If the agent attempts to force-close the browser or end ACW (After Call Work) before completing the safety adjudication, show a system-level modal warning: `"Unresolved Safety Events: Your progress will be lost and legal reporting flags may be missed. [ Keep Reviewing ] [ Discard & Exit ]"`.
* **5. Cross-Industry Reusability Tags:** 
  * `#post-call-adjudication` `#compliance-wrap-up` `#transcript-verification` `#human-in-the-loop` *(Directly reusable for **Insurance** during Fraud/Claims triage reviews at wrap-up, and **FinTech/Banking** when auditing suspicious transaction alerts flagged during a session).*

---

## 🧭 Playbook: How to Design Similar Initiatives (Avatar Guidance)
When another designer asks Aria how to approach a **Compliance or Regulatory Review** flow in any industry, guide them through these 4 steps:
1. **Step 1 (Determine Proactivity):** Always default to post-call/post-session adjudication instead of real-time alerts. Real-time alerts add cognitive load, whereas post-interaction reviews preserve focus and empathy.
2. **Step 2 (Provide the "Receipts" / Proof):** Never show an AI-flagged warning without showing the **exact transcript snippet or data source** that triggered it. Users must be able to verify the AI's logic instantly without digging.
3. **Step 3 (Audit the Rejections):** Legally, dismissing a flagged safety/compliance risk is just as important as confirming it. Force the user to provide a quick reason when dismissing, and audit that click.
4. **Step 4 (Enforce the Exit Lock):** Prevent the user from navigating away, wrapping up, or auto-timing out if there are unresolved compliance flags on their screen.
