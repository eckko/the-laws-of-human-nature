# Saving progress to an account (cloud sync)

Optional. Without it, progress lives only in the reader's browser. With it,
readers can sign in (Google, via Firebase) and their progress follows them
across devices. Only progress is synced (memory levels, review dates, days
practised, last five results), never the questions.

## Files

- `sync-config.js`: which database and its settings (the file to edit).
- `progress-merge.js`: combines two copies of progress (no database code).
- `cloud-sync.js`: sign-in bar, when to save, merging (no database code).
- `providers/firebase.js`: Google sign-in + Firestore adapter.
- `providers/firebase-firestore.rules`: security rules to publish in Firebase.
- `providers/example-in-browser.js`: smallest adapter; copy it for a new DB.

## Turn sync off

In `sync-config.js` set `provider: "none"`. To remove it completely, also
delete `js/sync/`, `css/cloud-sync.css`, their lines in `index.html` and the
empty `#cloud-sync-bar` div.

## Set up Firebase (one-time)

1. Firebase console: create a project (free Spark plan is enough).
2. Build > Authentication > Sign-in method > Google > Enable.
3. Authentication > Settings > Authorized domains: add your GitHub Pages
   domain (for example `eckko.github.io`).
4. Build > Firestore Database > Create database (production mode).
5. Firestore > Rules: paste `providers/firebase-firestore.rules`, Publish.
6. Project settings > Your apps > Web app: copy `firebaseConfig` into the
   `firebase` block of `sync-config.js` (already done for this project).

Data is stored at `users/{uid}/books/{book id}` with fields `progress` (JSON
text) and `updatedAt`. One Firebase project serves every book on the domain.

## Behaviour

- Merge: per question the most recently answered record wins; days and recent
  results are combined.
- Reset/Load stamp `resetAt`, so older copies elsewhere cannot undo them.
- Sign out saves, then clears this book's progress from the browser.
- Offline or database down: practice continues, saved in the browser.

## Different database

Copy `providers/example-in-browser.js` to `providers/<name>.js`, implement
`start`, `signIn`, `signOut`, `readProgress`, `writeProgress`, keep the
`registerProvider("<name>", ...)` call, then set `provider: "<name>"` in
`sync-config.js` with a `<name>: {...}` settings block.
