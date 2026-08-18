# Firebase setup for the Salesman Claim App

The app runs on the existing **`iht-invoices`** project, shared with the Invoice Tracker.

## The sign-in problem — root cause (confirmed)

**The app must be served from `https://hydrotech3386.github.io`. It cannot be opened as a
local file.**

The Firebase API key has **HTTP referrer restrictions** limiting it to the GitHub Pages
domain. Tested directly against the project's anonymous sign-in endpoint:

| Referer sent | Result |
|---|---|
| none — what a `file://` page sends | `403 Requests from referer <empty> are blocked` |
| `https://hydrotech3386.github.io/` | Success, returns a token |

The chain that produced "sign in blocked" / "permission denied":

1. Page opened as a local file → browser sends no referer
2. Anonymous sign-in rejected by the API key restriction
3. `auth` is therefore `null`
4. The database rules are `".read": "auth != null"` → every read denied

Nothing is misconfigured in Firebase. Hosting the app fixes all of it.

### Things that are already correct — do not change them

| | Status |
|---|---|
| Anonymous sign-in provider | ✅ Enabled |
| Database rules | ✅ Root `".read"/".write": "auth != null"` **cascades to every child path**, so `salesmanClaims` and `salesmanLists` are already covered. No rules edit is needed. |

## Fix: host it (also required for phones)

Salesmen cannot open a OneDrive path on their phones, so this has to be hosted regardless.
Deploy it the same way as the other Hydrotech apps:

```sh
cd "C:/Users/danie/OneDrive/Claude/Hydrotech/Salesman Claim App"
git init
git add index.html README.md FIREBASE-SETUP.md
git commit -m "Salesman claim app"
gh repo create hydrotech3386/iht-salesman-claim --public --source=. --push
```

Then repo **Settings → Pages → Deploy from branch → main / root**. The app appears at
`https://hydrotech3386.github.io/iht-salesman-claim/`, which matches the allowed referrer,
and sign-in works with no Firebase changes at all.

### Testing on a computer before deploying

`npx serve` will **not** work — `http://localhost:…` is a different referer and is also
blocked. To test locally, add `http://localhost:*` in Google Cloud Console → **APIs &
Services → Credentials → the browser API key → Application restrictions → Website
restrictions**. Otherwise just deploy and test on the live URL.

## Storage — not used

The app keeps no photos, so Firebase Storage is **not required** and nothing needs enabling.
Photos exist only in memory while the AI reads them and the salesman confirms, then they are
dropped. The original paper receipt is the record.

## Built-in connection test

The sign-in screen has a **Run connection test** button. It checks each step — secure
context, Firebase loaded, anonymous sign-in, reading `invoiceUsers` / `appSettings` /
`salesmanClaims` / `salesmanLists`, and Storage — and prints the raw error code plus the
specific fix beside whichever one fails. Run it on the deployed URL if anything misbehaves
later.

## A note on the rules

The root rule grants every signed-in client read and write across the whole database. That
predates this app, and it means any salesman can in principle read another's claims (and the
Invoice Tracker's data). Tightening it means moving off the shared-password scheme to real
Firebase Auth accounts, where `$uid === auth.uid` becomes enforceable — a change across all
the apps, not something to bolt onto this one.
