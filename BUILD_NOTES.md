# Proposals Pulse Prototype: Build & Iteration Notes

A record of how the interactive prototype was produced with Claude, the decisions made along the way, and what each iteration taught us about the pipeline design.

**Live prototype:** https://brianbackdivisions.github.io/proposals-prototype/
**Repo:** https://github.com/brianbackdivisions/proposals-prototype
**Source file:** `index.html` (single self-contained file)

---

## 1. Starting Point

Context was grounded in pre-existing memory Claude already had loaded:
- Proposals Experience PRD (FUL-8019, $40M field-created proposals target)
- Lifecycle decisions: 8 statuses, source-agnostic Pending Review, qualification and pricing as separate decisions
- Opportunity Compass requirements (FUL-8032)
- WO-to-Proposal attribution PRD
- Key stakeholders and ticket map

This meant Claude could skip the discovery phase and start building with business logic already aligned to how we wanted V1 to behave.

**Initial ask:** build a clickable dashboard with a funnel, drill into proposals, and walk through qualification > pricing > submission states.

---

## 2. Build Approach

One self-contained `index.html` file. Pros:
- No build tooling, no dependencies
- Anyone can open locally or host statically
- Fast to iterate: every change is a single file edit
- Trivial to version in git, deploy to GitHub Pages, share as a link

Cons: the file is now ~1000 lines of mixed HTML/CSS/JS. For anything beyond demo, this would get refactored into components. For a fast-moving prototype meant to drive stakeholder conversations, the tradeoff was worth it.

Data is all synthetic, generated client-side on load, with a realistic distribution across lifecycle stages (~290 proposals). Enough to feel real without needing a backend.

---

## 3. Iteration Log

### v1: Dashboard + detail skeleton
- Funnel with 8 stages showing $ value and count per stage
- Filterable table (initial pass was cosmetic filters)
- Detail page with lifecycle stepper, qualification card, pricing tabs (customer pricing / provider quotes / ERQs)
- State transitions: approve qualification → Awaiting Pricing → save pricing → Pending Submission → submit → Submitted → mark approved/rejected

### v2: Cleaner dashboard, CC-style detail page
Feedback: too many filters, wrong columns, detail page didn't match Control Center patterns.

Changes:
- Removed: preferences, refresh, export, geography, customer interest filters
- Table columns: dropped approval probability, description strength, pricing source
- Table columns: added Created By (with Field/NAT/Provider chip), Assignee, Days in Stage, Created Date, Estimates Received (per FUL-8021)
- Detail page restructured to match CC task panel style:
  - Top to bottom: Scope/info → Pending Review task → Pricing → Submission
  - Task panel header with status and creator chips
  - Field-initiated alert banner
  - Line items matching CC layout (Labor & Trip, Material & Equipment, add lines, provider quote rows)

### v3: Full requirements pass
Research pass through FUL-8021, 8022, 8020, 8025, 8027, 8545, 8560 to make sure the prototype reflected real behavior.

Changes:
- **Sortable Estimates Received column** + Awaiting Pricing funnel sub-count ("X w/ estimates received")
- **Editable description field** with "Enhance with AI" button
- **Qualification → Confirm Relevant Proposal**: Compass Insights box with contract coverage signals (positive/warning/negative), PSA status, historical cancellation patterns, NTE context. Designed as progressive surface: V1 ships with field-submitted context, reserved layout for future Compass signals.
- **Pricing section locked until qualification approved** (grayed out with "Qualify proposal to unlock pricing")
- **Estimates section** with ERQ lifecycle: open (field-started, not posted), sent (on provider board awaiting response), received (ready to apply). Each state gets the right CTA: "Review & Post to Provider", none, or "Use This Estimate".
- **Contract-locked rates** shown with lock icon and "Contract rate" label for customers with rate sheets
- **Proactive field visits** (15% of proposals generated with no NTE, tagged with purple "PROACTIVE" label)
- **Integrated vs non-integrated submission** paths differentiated

### v4: Salesforce opportunity creation at qualification
Reflecting decision #7 (SF opportunity created when proposal exits Pending Review, not at Draft or Submission).

Changes:
- On Approve Opportunity: button shows "Qualifying..." for 1.2s, then page re-renders with new status and a subtle slide-up toast in the corner
- Toast shows the auto-generated SF Opportunity ID with a "View in Salesforce" link
- Persistent blue SF banner in the proposal header for any qualified proposal

Iteration: first version was a blocking modal. Feedback: too intrusive. Rewrote as a slide-up toast card that auto-dismisses after 6 seconds but stays clickable.

### v5: Working filters
Feedback: the filters on the dashboard were decorative.

Changes:
- Every filter chip opens a real dropdown with unique values pulled from the data
- Click value to filter, click chip again to clear
- Filters stack (Customer + Service Line + Status, etc.)
- Active filter banner shows all applied filters as removable tags
- Live search box filters by ID, ticket, customer, location, description
- Empty state message when zero results
- Outside-click dismisses open dropdowns

### v6: Preview proposal flow (later reverted)
Tried a flow where "Save & Next" became "Preview Proposal" and showed a customer-facing view of the proposal before submitting.

Feedback: too wonky. The save button was buried inside the Labor & Trip section instead of at the bottom of all pricing. Reverted.

What stayed:
- CTA moved to the **very bottom** of the pricing card, below all line items and totals
- Line items restructured to show **Customer** and **Provider** totals side by side in each section header plus margin
- Separate "Provider labor quote" / "Provider material quote" rows highlighted beneath each customer rate table
- Button renamed "Save" and **disabled until provider pricing exists** ("Provider pricing required before saving")

### v7: Submission split
Feedback: one submit button covering both integrated and non-integrated was confusing.

Changes:
- **Integrated customers** (Kroger, Target, Walmart, Home Depot, CVS) get "Submit via ServiceChannel" (or "Submit via Maximo" for Target) with a loading state simulating the API call
- **Non-integrated customers** get "Mark as Submitted" with helper text ("Submit manually via email, customer portal, phone, etc., then confirm below")
- Cancel button moved into the submission card itself instead of a bottom action bar
- Removed duplicate buttons
- Subtle "Mark submitted manually" fallback link for integrated customers (in case the integration is broken or the user went outside it)
- Submission status message reflects actual path: "Submitted via ServiceChannel" vs "Submitted (manually)"

---

## 4. Deployment

**GitHub Pages** under personal account (`brianbackdivisions/proposals-prototype`) because the dmg-innovate org didn't have repo creation permissions for my GitHub account. A simple `pages.yml` workflow auto-deploys on every push to main (~20 seconds).

Sharing options for teammates:
- Direct URL: view and demo
- Raw file URL: paste into Claude to iterate independently
- Clone the repo: edit locally

To move to the org later: get an admin to create `dmg-innovate/proposals-prototype` or grant repo-create rights, then transfer.

---

## 5. What Worked Well

1. **Grounded context upfront.** Having the PRD, key decisions, and ticket map already in Claude's memory meant every iteration landed with the right business logic without having to re-explain.
2. **Single-file prototype.** Zero friction to edit, deploy, share, or hand off for someone else to iterate.
3. **Iterate in small slices.** Each round was one conversation turn focused on a specific flow (filters, SF creation, submission split). Prevented drift and kept scope clear.
4. **Screenshots as feedback.** Sharing a screenshot of the live prototype with annotations (or just describing what felt off in the UI) produced better results than abstract requirements.
5. **Synthetic data with realistic distribution.** Made the funnel look right and gave demo paths for different customer types, integrations, and proactive visits without any backend work.

## 6. What to Watch Out For

1. **Conditional rendering complexity grows fast.** The "Preview Proposal" attempt created nested conditionals across sections that made the file hard to reason about. Reverting was the right call.
2. **Button proliferation.** Several rounds ended with duplicate submit buttons between section cards and a bottom action bar. Eventually consolidated into the section card itself.
3. **Feature flag vs. state.** Things like "proposal is qualified" or "is proactive" or "customer is integrated" drive a lot of UI. Worth naming these clearly in the data model so the template logic stays readable.
4. **Single-file hits a ceiling.** Once this goes past demo into actual design reference, it should be broken into components with a proper state management layer.

---

## 7. What This Prototype Is Good For

- **Executive demos:** click through the full lifecycle in 2 minutes
- **Stakeholder alignment:** show exactly what qualification, pricing, and submission will feel like rather than describing it
- **Design reference for engineering:** component shapes, state transitions, empty states, validation rules are all visible in working form
- **Scenario testing:** switch between integrated/non-integrated customers, proactive vs NTE-based proposals, field-initiated vs NAT-created proposals to pressure-test edge cases

## 8. What It's Not

- Not a design system source of truth (colors, spacing, typography are approximations)
- Not production-ready code (single file, inline styles, no accessibility audit)
- Not connected to real data or APIs (all client-side synthetic data)
- Not a substitute for the PRDs (the Jira tickets still own the requirements)

---

## 9. How to Iterate Further

Anyone with a Claude session can iterate on this:

```
# Option 1: Claude fetches the live source
"Please fetch https://raw.githubusercontent.com/brianbackdivisions/proposals-prototype/main/index.html
and modify [X]."

# Option 2: Clone locally
git clone https://github.com/brianbackdivisions/proposals-prototype.git
# then open a Claude session in the directory
```

Changes pushed to main auto-deploy to the live URL in about 20 seconds.

---

_Built April 2026. Last updated after v7 (submission split + manual fallback)._
