# Neon Slip

## Ownership

Original game and website by **Ali Imran**.  
Copyright &copy; Ali Imran 2025-2026. All rights reserved.

The game, branding, visuals, and site presentation are protected.  
See [license.html](license.html) and [terms.html](terms.html) for usage restrictions.

## Firebase config (hide keys from repository)

This project loads Firebase config in two layers:

1. `firebase-config.js` (committed): production defaults used by deployed builds
2. `firebase-config.local.js` (gitignored): optional local override for development

Local override setup:

1. Copy `firebase-config.example.js` to `firebase-config.local.js`
2. Fill in your Firebase web app config in `firebase-config.local.js`
3. Keep `firebase-config.local.js` uncommitted (it is ignored by `.gitignore`)

If your local override has an empty `apiKey`, Firebase is not initialized (auth/cloud features disabled).  
If `apiKey` is set but `databaseURL` is empty, auth still works but leaderboard/cloud database features stay local-only.

Security note:
- Firebase web API keys are client identifiers (not secrets), but should still be restricted in Firebase/Google Cloud console settings (allowed referrer domains + API restrictions).
- Ensure Firebase Authentication authorized domains include your production host(s) (for this repo: `neonslip.me`, and `www.neonslip.me` if used).
- Keep Realtime Database rules restrictive (this repo deploys `database.rules.json` via workflow).

## Leaderboard authentication

The leaderboard uses Firebase Authentication UIDs as database keys, including when
the visible player name comes from CrazyGames. Enable the **Anonymous** provider in
Firebase Console under **Authentication > Sign-in method** so players without a
separate Firebase account can submit scores. Keep `database.rules.json` deployed;
it only permits a player to write the leaderboard row matching their own `auth.uid`.

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
