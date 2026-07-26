# The Reading Ledger

A personal reading tracker — TBR queue, library, yearly stats, series progress, reading goals, and now shareable with friends — backed by Firebase (email/password sign-in + Firestore) and hosted for free on GitHub Pages.

There's no build step. It's plain HTML + a React component transformed in the browser by Babel, loaded from CDNs. You never need to run `npm install`.

## Files

- `index.html` — the page shell; loads React, Babel, and Firebase from CDNs, then fetches and transforms `app.jsx`
- `firebase-config.js` — your Firebase project's public config + `firebase.initializeApp(...)`
- `app.jsx` — the entire app (UI, Firestore data layer, auth, friends)
- `firestore.rules` — a copy of the security rules for reference (the live rules are set in the Firebase console — **republish these any time this file changes**)
- `favicon.ico`, `icon-16.png`, `icon-32.png`, `icon-180.png`, `icon-192.png`, `icon-512.png`, `manifest.json` — browser tab icon + "Add to Home Screen" icon/behavior

## One-time setup

1. Create a Firebase project → enable **Firestore Database** → enable **Authentication → Email/Password** sign-in provider.
2. Register a **Web app** in Project Settings → copy the `firebaseConfig` values into `firebase-config.js` (already done for you here).
3. In **Firestore → Rules**, publish the contents of `firestore.rules`.

## Deploy to GitHub Pages

1. Create a GitHub repo and push all the files above to the `main` branch (root of the repo — no subfolder).
2. In the repo: **Settings → Pages** → Source: **Deploy from a branch** → Branch: **main**, folder **/ (root)** → Save.
3. GitHub will publish it at `https://<your-username>.github.io/<repo-name>/`. First deploy can take a minute or two.

That's it — open the URL on your phone, tablet, or laptop, sign in (or create an account), and it's the same data everywhere.

## Testing locally before you push

Opening `index.html` directly by double-clicking it (a `file://` URL) will *not* work — browsers block the `app.jsx` fetch from local files. Instead, serve the folder over HTTP:

```bash
# from inside the project folder
npx serve .
# or
python3 -m http.server 8000
```

Then visit the printed `localhost` URL.

## How your data is stored

Everything lives in Firestore under `users/{your-uid}/`:
- `library/books` — your entire book list (TBR, reading, read, DNF) — private, only you can ever read this
- `library/goals` — your yearly reading goals — private
- `library/seriesOverrides` — manual series DNF flags / looked-up total-book-counts — private
- `library/publicSummary` — a friends-readable subset: currently-reading, finished books, and yearly stats. Regenerated automatically from your library any time it changes. Your TBR queue and quotes are never included here.
- the top-level `users/{uid}` doc — your display name, email, and invite code (readable by any signed-in user, so friends can see your name)
- `friends/{friendUid}` — one doc per friend you've added (mutual — adding someone writes a matching doc on both accounts)

A brand-new account (yours originally, or a friend's) starts with an **empty** shelf — nothing is auto-seeded anymore.

## Sharing with friends

Anyone can go to your same GitHub Pages URL and create their own account — they'll get their own completely separate, empty ledger, not yours. From there:

1. Each person finds their own invite code on the **Friends** tab.
2. Share that code with a friend (text, however) — they enter it in their own **Friends** tab under "Add a friend."
3. Adding is mutual and immediate — no approval step. You'll then each see the other's currently-reading, finished books, and yearly stats under Friends → View shelf.
4. Your TBR queue, quotes, and series notes are never shared — only what's in the "public summary" (see above) ever leaves your own account.

## Updating the app later

If you come back to Claude for more changes, just say so — I'll hand you an updated `app.jsx` (or `firestore.rules`, if data-access rules need to change) to drop in over the old one. `firebase-config.js` and `index.html` shouldn't need to change again.

## Limitations worth knowing

- **Rules complexity.** The friends feature relies on Firestore security rules (not just app-side checks) to keep your TBR list private — this is the correct, tamper-proof way to do it, but it does mean the rules file is more involved than a purely personal single-user setup. If you ever ask for more sharing features (e.g. seeing a friend's TBR too), the rules will need matching updates.
- **No notifications.** There's no "your friend just finished a book" push alert — you have to open their shelf to see updates. Doable to add later if you want it (would need a small activity-log data structure).
- **Invite codes are permanent** — never reused, but there's no way to regenerate your own code from within the app yet if you ever wanted to invalidate it (would need to be added).
- **Display names aren't unique** — nothing stops two friends from picking the same name; it's a label for the UI only, not an identifier.

## Cost

Firebase's free "Spark" tier includes 50K document reads and 20K writes per day, and 1 GiB storage — for you and a handful of friends this is essentially unlimited headroom. The one extra cost the friends feature adds is a single additional document read per rule check when a friend opens your shelf (the `exists()` check in the rules) — negligible at this scale.
