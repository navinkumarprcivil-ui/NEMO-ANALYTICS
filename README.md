# Nemo Aqua Store — Business Analytics

A single-file finance, GST, inventory and margin-analytics dashboard for an
aquarium / aquatics retail business. Everything runs in the browser; your data
is stored locally in the browser (localStorage) — no server, no database.

## What's inside

| File | Purpose |
|------|---------|
| `index.html` | The entire app — fully self-contained, works offline. This is all Vercel serves. |
| `vercel.json` | Vercel static-hosting config. |
| `package.json` | Lets you run it locally with `npm start`. |
| `.gitignore` | Standard ignores. |

> **Note:** `index.html` is a compiled bundle. The editable source lives in the
> parent project as `Aqua Finance.dc.html`. Re-export the bundle whenever you
> change the source.

## Features
- Dashboard: real cost-per-order, packing %, DOA loss %, zone-wise margin
- Margin Playbook driven by real order data
- Monthly / FY Profit & Loss with Excel + PDF export
- Inventory & stock value with low-stock reorder flags
- Auto-categorised expenses (vendor → category memory)
- Month-over-month revenue / expense / margin trends
- GST 3B summary with Excel + PDF export
- Monthly backup reminder (one-click full Excel workbook)

## Run locally
```bash
npm start        # serves on http://localhost:3000
```
Or just open `index.html` in any browser.

---

## Deploy to GitHub

```bash
cd deploy
git init
git add .
git commit -m "Nemo Aqua Store dashboard"
git branch -M main
git remote add origin https://github.com/<your-username>/nemo-aqua-store.git
git push -u origin main
```

## Deploy to Vercel

**Option A — from the dashboard (easiest)**
1. Go to https://vercel.com/new
2. Import your `nemo-aqua-store` GitHub repo.
3. Framework preset: **Other**. Build command: *(leave empty)*. Output dir: *(leave empty / `.`)*.
4. Click **Deploy**. Done.

**Option B — Vercel CLI**
```bash
npm i -g vercel
cd deploy
vercel          # follow prompts, accept defaults
vercel --prod   # promote to production
```

Because the app is a static `index.html`, no build step is required.

## Deploy to Firebase (Hosting + secure rules)

The repo ships ready-to-use Firebase config:

| File | Purpose |
|------|---------|
| `firebase.json` | Hosting + which rules files to deploy |
| `.firebaserc` | Your project id (replace the placeholder!) |
| `firestore.rules` | Firestore security rules — **owner-only** |
| `storage.rules` | File-attachment rules (images/PDF, ≤10 MB, owner-only) |
| `database.rules.json` | Realtime DB rules — owner-only |

### One-time setup
```bash
npm i -g firebase-tools
firebase login
# put your real project id in .firebaserc (or run: firebase use --add)
```

### Deploy
```bash
cd deploy
firebase deploy                     # hosting + all rules
# or deploy just the rules:
firebase deploy --only firestore:rules,storage,database
```

### The security model (important)
Every business stores its data under **`/stores/{uid}/...`**, where `{uid}` is
the signed-in user's Firebase Auth UID. The rules guarantee:

- **No public access.** Unauthenticated requests are denied everywhere.
- **Owner isolation.** A user can only read/write data under their own `uid` —
  never another store's.
- **Shape checks.** Expenses/orders must carry a numeric `amount`, inventory a
  `name` + numeric `qty`, so a tampered client can't write garbage.
- **Attachments** (expense bills) must be an image or PDF, under 10 MB.

> You **must** enable **Firebase Authentication** (e.g. Email/Password or Google)
> for these rules to work — they rely on `request.auth.uid`. Without auth enabled,
> every request is denied.

> The app is **wired to Firebase**. On load it requires Google sign-in; only
> admins (you + anyone you add in Settings) can read/write the shared store at
> `/stores/main`. Data syncs live across all admins and is cached locally for
> offline viewing.

> **Before sign-in works**, in the Firebase console go to **Authentication →
> Sign-in method**, enable **Google**, then under **Settings → Authorized
> domains** add the domain you deploy to (e.g. `your-app.vercel.app` and
> `your-app.web.app`). `localhost` is allowed by default.

### Adding your co-admin
1. Send them the deployed URL; they click **Continue with Google** once.
2. They'll see a “Not authorised” screen showing their **Google UID** — they copy it to you.
3. You open **Settings → Store admins**, paste their UID, click **Add**.
4. They reload and now have full access. You can remove a co-admin anytime from the same screen.

## Important: your data
Data is stored in **your Firebase project** and shared live across all admins,
with a local cache for offline viewing. Use the in-app **monthly backup** button
to download an Excel copy as an extra safeguard.
