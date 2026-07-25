# The Reading Ledger

A personal reading tracker — TBR queue, library, yearly stats, series progress, and reading goals — backed by Firebase (Google sign-in + Firestore) and hosted for free on GitHub Pages.

There's no build step. It's plain HTML + a React component transformed in the browser by Babel, loaded from CDNs. You never need to run `npm install`.

## Files

- `index.html` — the page shell; loads React, Babel, and Firebase from CDNs
- `firebase-config.js` — your Firebase project's public config + `firebase.initializeApp(...)`
- `app.jsx` — the entire app (UI, Firestore data layer, auth)
- `firestore.rules` — a copy of the security rules for reference (the live rules are set in the Firebase console)

## One-time setup (you've likely already done this)

1. Create a Firebase project → enable **Firestore Database** → enable **Authentication → Google** sign-in provider.
2. Register a **Web app** in Project Settings → copy the `firebaseConfig` values into `firebase-config.js` (already done for you here).
3. In **Firestore → Rules**, publish the contents of `firestore.rules`.

## Deploy to GitHub Pages

1. Create a GitHub repo and push these four files to the `main` branch (root of the repo — no subfolder).
2. In the repo: **Settings → Pages** → Source: **Deploy from a branch** → Branch: **main**, folder **/ (root)** → Save.
3. GitHub will publish it at `https://<your-username>.github.io/<repo-name>/`. First deploy can take a minute or two.
4. **Important:** in the Firebase console, go to **Authentication → Settings → Authorized domains** and add `<your-username>.github.io` (Google sign-in will refuse to work from a domain it doesn't recognize).

That's it — open the URL on your phone, tablet, or laptop, sign in with Google, and it's the same data everywhere.

## Testing locally before you push

Opening `index.html` directly by double-clicking it (a `file://` URL) will *not* work — browsers block the `app.jsx` fetch from local files. Instead, serve the folder over HTTP:

```bash
# from inside the project folder
npx serve .
# or
python3 -m http.server 8000
```

Then visit the printed `localhost` URL. You'll also need to add `localhost` to Firebase's **Authorized domains** (same screen as above) to sign in while testing locally.

## How your data is stored

Everything lives in Firestore under `users/{your-uid}/library/`:
- `books` — your entire book list (TBR, reading, read, DNF)
- `goals` — your yearly reading goals

The very first time you sign in, it seeds those documents from the book list built into `app.jsx` (from your spreadsheet import). After that, all edits — adding books, marking things read, reordering your TBR queue, adjusting goals — write straight to Firestore, and Firestore's realtime listeners push updates to any other device you have the page open on.

## Updating the app later

If you come back to Claude for more changes, just say so — I'll hand you an updated `app.jsx` to drop in over the old one. `firebase-config.js` and `index.html` shouldn't need to change again.

## Cost

Firebase's free "Spark" tier includes 50K document reads and 20K writes per day, and 1 GiB storage — for a single-person reading tracker this is essentially unlimited headroom.
