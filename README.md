# Salesman Monthly Claim App

Mobile web app for Hydrotech salesmen to record outstation trips and expenses during the
month, then generate the **Claim Form** and **Daily Mileage Claim** for submission.

**Live:** https://hydrotech3386.github.io/iht-salesman-claim/

## How photos are used

Photos are how the app *reads* a receipt or an odometer — they are never a record.

1. Salesman writes the customer name and/or the reason **by hand on the paper receipt**
2. Photographs it
3. The AI reads the printed date and amount **and the handwriting**
4. The salesman checks what it read and corrects anything wrong
5. On confirm, the extracted figures are saved and **the photo is discarded**

Nothing is uploaded and nothing is kept on the phone. The original annotated receipt is
what gets stapled to the printed claim form, exactly as before.

This is why the app needs **no Firebase Storage**, no upload queue, and no attachment pages
in the print pack.

## Signing in

Salesmen use **the same login as the Invoice Tracker** — the app reads the shared
`invoiceUsers` table on the `iht-invoices` Firebase project, so no new accounts are needed.
Accounts marked `pending` or `suspended` are refused, same as in the Invoice Tracker.

| Role | What they get |
|---|---|
| `salesman` (and any other role) | Their own claims, full edit |
| `management` | A staff picker over everyone's claims, **read-only** |

Management sees a read-only banner and no edit controls, but can still print any salesman's
claim pack. Approval stays on the signed paper form.

## How data is stored

| What | Where |
|---|---|
| Trips, amounts, descriptions | Realtime Database — `salesmanClaims/{uid}` |
| Customer / area / toll / reason suggestions | `salesmanLists` (shared by all salesmen) |
| Offline cache | `localStorage` on the phone |
| Photos | **Nowhere** — held in memory during verification, then dropped |

Claims sync to the account, so a salesman who changes phone signs in and finds their month
intact, including the customer and area names anyone has typed before.

Out of signal, entries save to the phone and the status strip under the header says
*"Offline — changes saved on this phone"*. They sync when signal returns.

## The six tabs

**Mileage** — one entry per outstation trip: date, area, start/end odometer, and one line
per customer visited with **the reason for that visit**. On the Daily Mileage Claim the
customers are numbered in the Customer column and their reasons line up in Remarks. Start
reading pre-fills from the previous trip's end reading. Tick *This trip has toll* to record
from-toll, to-toll, and one-way / two-way.

*Odometer:* photograph it and tap **✨ Read the odometer from the photo**. Two photos on one
trip fill start and end (lower = start); one photo fills whichever box is empty. Blurred or
glared digits come back with an amber "please check" warning. The photo is dropped when the
trip is saved.

**Toll** — automatically lists every trip ticked as having toll. Enter the one-way charge
per trip; two-way is doubled. Key the monthly statement total to get a match/mismatch check
against the trips claimed.

**Parking / F&E / Others** — tap **📷 Scan receipts**, photograph one or several, and each
becomes a line on the review screen with its date, amount and description already filled in
from the receipt and your handwriting. Correct anything, drop anything that isn't a claim,
then add them all at once. **✎ Enter manually** is always available if the AI can't read it
or the key isn't configured.

**Claim** — running summary, then generates both forms.

## Generating the claim

Claim tab → **🖨 Print / Save as PDF** → in the print dialog choose *Save as PDF*.

1. **Claim Form** — mileage total × rate, then TOL CLAIM / PARKING / FOODS &
   ENTERTAINMENT / OTHERS with per-line dates and amounts and a section subtotal
2. **Daily Mileage Claim** — 11 rows per page, extra pages added automatically past 11 trips

The paper form has a fixed number of numbered lines (7 for F&E, 6 for Others). Extra items
are merged into the last line as *"Other N items (see attached receipts)"* — the total stays
correct. Staple the original receipts to the printed pack.

## AI setup

Uses the Claude API key already stored in the Invoice Tracker
(`appSettings/claudeApiKey` on the same Firebase project) — the invoice scanner uses the
same key. Model is `claude-opus-5` with a strict JSON schema, called from the browser.

If no key is set, the scan buttons explain that and manual entry carries on as normal.

## Settings (⚙)

Employee's name, vehicle, **mileage rate** (default `0.40`/km per the form template),
export/import backup, sign out, and clear-current-month.

## Deployment

Hosted on GitHub Pages at `hydrotech3386.github.io`, which is required — the Firebase API
key is restricted to that domain, so the app cannot sign in when opened as a local file.
See FIREBASE-SETUP.md.

After any change: `git add -A && git commit -m "..." && git push`.

## Template note

`Claim Form Template.pdf` prints the company name as "Ipon Hydrotech" — a typo in the
original. The app prints "Ipoh" on both forms.
