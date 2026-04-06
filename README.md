# Neon Slip

## Firebase config (hide keys from repository)

This project now loads Firebase config from a local, gitignored file:

1. Copy `firebase-config.example.js` to `firebase-config.local.js`
2. Fill in your Firebase web app config in `firebase-config.local.js`
3. Keep `firebase-config.local.js` uncommitted (it is ignored by `.gitignore`)

If `firebase-config.local.js` is empty, the game falls back to local-only mode.

## Admin mode access

Admin mode does **not** use a client-side password.

Access is controlled in Realtime Database at:

- path: `adminAccess/{uid}`
- value: `true`

How it works:

- the default owner bootstrap is the verified account `al116im24@ast.kevibham.org` (enforced by DB rules)
- once signed in, that owner account is auto-granted in `adminAccess/{uid}`
- any current admin can grant another user by UID from the in-game Admin panel

No Firebase custom-claim or Admin SDK step is required.
