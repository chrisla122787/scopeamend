# ScopeAmend

A change-order pricing and signed-approval tool for small remodeling and
renovation contractors. Tagline: **"Never do extra work you don't get paid
for."** Positioning line: **"Price the change. Document the approval.
Protect your profit."**

The problem it solves: a client asks for extra work mid-project, the
contractor does it on a verbal "sure," and the money is never priced, never
approved in writing, and argued about at the end. ScopeAmend prices the
change in minutes and produces a clean change-order document with a client
signature block — before the work starts.

Pricing: **$19/month founding-member pricing** for the first 25 customers,
locked in while the subscription stays active. Standard price is
**$39/month** after the founding spots are gone. The calculator and a
watermarked document preview are free, and so is your first clean PDF;
Pro unlocks unlimited PDFs, multiple projects, the change-order
history/dashboard, and backup export.

Plain static HTML/CSS/JS — no build step, no backend.

## Files

| File | What it is |
|---|---|
| `index.html` | Landing page: promise, how it works, free/Pro features, a static sample change order, founding pricing, privacy + billing/cancellation notes. |
| `app.html` | The tool: change-order calculator with line items, contract roll-up, watermarked document preview, PDF change order (jsPDF via CDN; first one free, then Pro), per-project change-order history with statuses and document versioning (approved versions lock; revisions create V2, V3, …), dashboard totals, backup export/import, CSV export, and per-approved-CO invoice copy. |
| `README.md` | This file. |

## Before launch

- Replace the `STRIPE_LINK` placeholder hrefs with the real Stripe
  subscription payment link for ScopeAmend ($19/month founding price).
  There are two in `index.html` (hero + pricing) and one in `app.html`
  (Pro gate card).
- Contact email in the footer is `chrisla122787@gmail.com`.

## Free vs Pro

**Free (no payment, no account):**

- Full change-order calculator for **one project** — line items, totals,
  and the full contract roll-up, live as you type.
- Watermarked change-order preview rendered on screen (diagonal
  "PREVIEW — ScopeAmend" watermark), showing the work, line amounts, the
  new contract total, and the document ID.
- **One free clean PDF** — the first PDF change order is unwatermarked
  and client-ready, no account or subscription needed (one per browser,
  tracked by a best-effort local counter — `scopeamend.freePdfUsed.v1`;
  after that, PDFs are Pro).

**Pro (subscription):**

- Clean branded PDF change order: company logo + contact info, client
  and project, change-order number and Document ID (e.g. CO-003-V1),
  original contract reference, description + reason, scope exclusions,
  line-item table, change order total, previous contract total, new
  contract total, schedule impact, payment terms, the agreement line
  ("Both parties agree this change is subject to the original contract
  and applicable law"), an "Approved on" line, a two-party approval
  block (client AND contractor signature + printed name + date), and
  the legal disclaimer printed discreetly at the foot of the document.
- Multiple projects, each with its own numbered change-order history.
- Statuses (Draft / Sent / Approved / Declined / Paid), a per-change-order
  "client approved" checkbox, duplicate, and dashboard totals.
- "Copy for Invoice" per approved change order (plain-text summary:
  document ID, project, client, description, amount, date) and a CSV
  export of all change orders (project, document ID, date, description,
  status, amount).
- Backup export (JSON) / import-restore.

**The customer document never shows internal numbers.** Unit costs and
markup percentages exist only in the private editor and the "Your numbers"
panel. The preview and the PDF show each line's description and line
amount only — exactly how standard change orders present pricing.

## Document versioning & locking

Every change order carries a **Document ID**: `CO-003-V1` — the
change-order number plus a version. It appears on the preview and the
PDF, and in the CSV export and invoice copy.

- A change order whose status is **Approved or Paid is locked**: its
  fields and line items can't be edited in the editor (the editor shows
  a LOCKED banner). This protects the version the client actually
  signed.
- To change a locked change order, use **Create revised version** (the
  editor banner button, or the history row's Revise button). It creates
  the next version (V2, V3, …) as a fresh Draft; the approved version
  stays in the history untouched, and the project's CO number counter
  does not advance.
- Money rule for versions: each change-order **number** counts once
  toward approved totals — the highest-version Approved/Paid version of
  that number. A Draft revision doesn't disturb the approved version's
  amount; once the revision is approved, its amount replaces the older
  version's in the roll-up and the dashboard.

## Legal disclaimer

This exact text — "ScopeAmend provides document preparation and change-order management tools. It does not provide legal advice or guarantee the enforceability of any document. Users are responsible for ensuring that their agreements comply with applicable laws, licensing requirements, and original contract terms." — appears in three places:
the landing page (Terms policy card and the footer), the app next to
the PDF button, and as discreet footer text on every generated PDF.

## ⚠️ MVP access note (read before launch)

Pro unlocking is **client-side only** for launch validation:

- The "I already subscribed — unlock Pro" button in `app.html` simply sets
  a `localStorage` flag (`scopeamend.pro.v1 = "1"`) in the visitor's own
  browser. There is **no real payment verification** — no server checks the
  Stripe subscription, and anyone who finds the button can unlock Pro
  without paying.
- This is a deliberate MVP trade-off to validate demand cheaply on a $0
  static stack (the landing page says so in plain words: unlocking is on
  the honor system during opening validation).
- Before scaling past validation, replace this with a real entitlement
  check (e.g. a small serverless function that verifies the Stripe
  subscription by email before issuing an unlock token).

## The change-order math

All math is in `app.html` (`lineTotal()`, `coTotals()`, `rollup()`):

```
lineTotal        = qty × unitCost × (1 + markup% / 100)
lineCost         = qty × unitCost                     (internal only)
changeOrderTotal = Σ lineTotal
markupInCO       = changeOrderTotal − Σ lineCost      (internal only)

approvedToDate   = Σ totals of the project's change orders with status
                   Approved or Paid (the change order currently being
                   edited is excluded — it is counted separately below)
previousTotal    = originalContract + approvedToDate
newContractTotal = previousTotal + thisChangeOrderTotal
pctChange        = (approvedToDate + thisChangeOrderTotal)
                   ÷ originalContract × 100
```

Each line item also carries a type tag — Labor / Materials / Other — for
the contractor's own organization. It does not affect the math and does
not appear on the client document.

Change-order numbers auto-increment **per project**: each project stores
its own `nextCo` counter (CO-001, CO-002, …), and saving a new change
order or duplicating one consumes the next number. Updating a saved
change order keeps its number.

Worked example 1 — line math (hand-verified):

- Line: qty 2 × unit cost $100 × (1 + 25% markup) = **$250.00** line total.
- Internal cost of that line: $200.00; internal markup: $50.00.
- The client document shows only: description + **$250.00**.

Worked example 2 — contract roll-up (hand-verified):

- Original contract: $40,000
- CO-001 approved: $1,250 (e.g. qty 1 × $1,000 × 1.25)
- CO-002 approved: $800 (e.g. qty 4 × $160 × 1.25)
- Editor open on a fresh change order totalling $0
- Approved to date: $2,050 · Previous total: $42,050
- **New contract total: $42,050** · Change vs original: **+5.1%**

## Data & privacy

Everything lives only in the visitor's own browser (`localStorage`):

| Key | Contents |
|---|---|
| `scopeamend.projects.v1` | All projects, each with its change orders (fields, lines, statuses, approved flags, dates). |
| `scopeamend.settings.v1` | Company name/phone/email, logo (data URL), and the currently selected project id. |
| `scopeamend.pro.v1` | Pro unlock flag (see MVP note above). |
| `scopeamend.freePdfUsed.v1` | Set to `"1"` once the one free PDF has been generated in this browser. Best-effort only — clearing browser data resets it, like everything else here. |

Nothing is sent to a server. The app tells users plainly: "Your data is
stored only in this browser on this device — export a backup regularly."
Backup export downloads one JSON file with settings + all projects;
import restores it (and rejects files that aren't ScopeAmend backups).

Change-order statuses: Draft, Sent, Approved, Declined, Paid. The
"client approved" checkbox and the status stay in sync: ticking it moves
a Draft/Sent/Declined change order to Approved; setting a status of
Approved or Paid ticks it. Dashboard totals (across **all** projects):

- **Outstanding** = Σ totals of change orders with status Sent
  (sent to the client, not yet approved).
- **Approved — added to contracts** = Σ totals of Approved + Paid.
- **Change orders (all projects)** = total count.

## Deploy

Any static host works — copy the three files (or at least the two HTML
files) to the host as-is:

- **Netlify / Vercel:** drag-and-drop the folder, or connect the repo; no
  build command, publish directory = this folder.
- **Cloudflare Pages:** same — framework preset "None", no build command.
- **GitHub Pages:** fine for demos/preview only — GitHub Pages terms
  prohibit primarily commercial SaaS use, so don't run the paid product
  from there.

The only external request the app makes is the jsPDF CDN script in
`app.html` (cdnjs). If it fails to load, everything except the PDF button
still works and the user gets a clear message.
