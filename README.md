# Proposals Pulse — Interactive Prototype

Interactive prototype for the Proposals Pipeline V1 experience at DMG, built to communicate design intent to engineering and stakeholders.

**Live:** https://brianbackdivisions.github.io/proposals-prototype/

---

## What's in the Prototype

### Proposals Pulse Dashboard
- Full 8-stage pipeline funnel (Draft → Pending Review → Awaiting Pricing → Pending Submission → Submitted → Approved → Rejected → Cancelled)
- Proposal table with Created By, Days in Stage, Est. Received, Assignee columns
- Filter by Customer, Service Line, Service Type, Assignee, Created By, Status
- Click any stage card to filter the table
- Click any row to open the proposal detail view

### Proposal Detail View
- Lifecycle stepper showing current stage
- Description with AI Enhance button
- Full Estimates + Pricing section with ERQ management, line items, accordion sections
- Salesforce opportunity creation on qualification (toast notification)
- Submit to Service Channel (integrated customers) or manual confirm (non-integrated)
- Cancel with reason taxonomy (Duplicate, Contractual, Customer Preference, etc.)

### Agent Signals at Pending Review *(FUL-8960)*
When a field-initiated proposal enters Pending Review, the system runs two AI agents:
- **Contract In-Scope Checker** — flags proposals that overlap with existing contracts
- **Duplicate Detection** — flags proposals that match active or recent proposals for the same scope/property

**Signal pills in the qualification container:**
| Signal | Trigger | Color |
|---|---|---|
| In Contract Scope · 100% | Full overlap detected | Red |
| Partial Contract Overlap · [%] | 50–99% overlap | Yellow |
| Possible Duplicate · [N] | 1+ duplicates found | Orange |
| No conflicts detected | Clean check | Green |

Clicking a pill opens a detail drawer with the full agent output. If a user qualifies despite a risk signal, a lightweight feedback form captures context to improve the model over time.

---

## How to Demo

1. Open https://brianbackdivisions.github.io/proposals-prototype/
2. Click the **Pending Review** stage card to filter to those proposals
3. Click any row to open the proposal detail
4. Scroll to the qualification card — agent signals appear after ~1.5s loading state
5. Click the red **In Contract Scope** or orange **Possible Duplicate** pill to open the detail drawer
6. Click **Qualify Proposal** with a risk signal present to trigger the feedback flow

---

## Tech
Single `index.html` — vanilla JS, no dependencies. Deployed via GitHub Pages.
