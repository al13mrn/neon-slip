# Neon Slip

## Firebase config (hide keys from repository)

This project now loads Firebase config from a local, gitignored file:

1. Copy `firebase-config.example.js` to `firebase-config.local.js`
2. Fill in your Firebase web app config in `firebase-config.local.js`
3. Keep `firebase-config.local.js` uncommitted (it is ignored by `.gitignore`)

If `firebase-config.local.js` is empty, the game falls back to local-only mode.

## Admin mode access

Admin mode no longer uses a client-side password.

Access is granted only when the signed-in Firebase Auth user has a custom claim:

- claim key: `admin`
- claim value: `true`

Example using Firebase Admin SDK:

```js
await admin.auth().setCustomUserClaims(uid, { admin: true });
```

After setting the claim, the user should sign out/in (or refresh token) to activate access.
