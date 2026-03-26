# Firebase Setup Guide — Neon Slip

---

## Already have the existing `neon-slip` project? Here's what to enable

The `firebaseConfig` in `index.html` is already fully filled in for the `neon-slip` project.
You only need to make sure the following are enabled in the Firebase console
(**console.firebase.google.com → project: neon-slip**):

| What | Where in the console | Status |
|------|----------------------|--------|
| **Email/Password auth** | Build → Authentication → Sign-in method → Email/Password → Enable | Required |
| **Realtime Database** | Build → Realtime Database (already created with the URL in the config) | Required |
| **Database rules** | Deploy via CI (add `FIREBASE_TOKEN` secret) or paste `database.rules.json` manually into the Rules tab | Required |
| **Firebase Storage** | Build → Storage → Get started (same region as DB) | Enables profile photo uploads |

That's it — the `storageBucket` field is now set to `neon-slip.appspot.com` so profile photos
upload to Firebase Storage automatically once Storage is enabled.

### Getting the CI auto-deploy working

Add a `FIREBASE_TOKEN` secret to your GitHub repository so the workflow can deploy
`database.rules.json` automatically on every push to `main`:

```sh
npm install -g firebase-tools
firebase login:ci   # prints a token — copy it
```

GitHub repo → **Settings → Secrets and variables → Actions → New repository secret**
- **Name:** `FIREBASE_TOKEN`
- **Value:** the token printed above

---

## Setting up a brand-new Firebase project (different account)

Follow these steps to set up a brand-new Firebase project that replicates the original
Neon Slip backend exactly (Realtime Database rules, Authentication, CI auto-deploy).

---

## 1. Create a Google Account & Firebase Project

1. Go to <https://accounts.google.com> and create a new Google account.
2. Go to <https://console.firebase.google.com> and sign in.
3. Click **Add project** → give it any name (e.g. `neon-slip`).
4. Disable Google Analytics if you don't need it, then click **Create project**.

---

## 2. Enable Authentication

1. In the Firebase console sidebar, click **Build → Authentication**.
2. Click **Get started**.
3. Enable the following sign-in providers (these are the ones the game uses):

   | Provider | Notes |
   |----------|-------|
   | **Email/Password** | Enable both Email/Password and Email link (passwordless) |
   | **Anonymous** | Allows guests to post scores without an account |

4. Click **Save** after enabling each provider.

---

## 3. Create the Realtime Database

1. In the sidebar, click **Build → Realtime Database**.
2. Click **Create database**.
3. Choose the closest region (e.g. **europe-west1** for Europe or **us-central1** for US).
4. Select **Start in locked mode** — you'll deploy proper rules in the next step.
5. Click **Enable**.

---

## 4. Deploy the Database Security Rules

The rules are already in [`database.rules.json`](./database.rules.json).
They enforce the following structure:

| Path | Read | Write | Notes |
|------|------|-------|-------|
| `leaderboard` | Anyone | Authenticated (new entries only) | Score validated 0–10 000 000, name ≤24 chars |
| `users/$uid` | Owner only | Owner only | Private user data |
| `publicProfiles` | Any authenticated user | Owner only | Indexed on `displayNameLower` |
| `friends/$uid` | Owner only | Any authenticated user | |
| `friendRequests/$uid` | Owner only | Any authenticated user | |
| `friendRequestsSent/$uid` | Owner only | Any authenticated user | |
| `neonSpeechThreads` | Any authenticated user | Any authenticated user | Indexed on `createdBy` |
| `neonSpeechMemberships/$uid` | Owner only | Any authenticated user | |
| `neonSpeech/$threadId` | Any authenticated user | Any authenticated user | |

### Option A — Automatic (CI) ✅ Recommended

This is the easiest option once you've done it once.

1. Install the Firebase CLI locally:
   ```sh
   npm install -g firebase-tools
   ```
2. Log in and get a CI token:
   ```sh
   firebase login:ci
   ```
   Copy the token that's printed (it looks like `1//0g...`).

3. In your GitHub repository, go to **Settings → Secrets and variables → Actions → New repository secret**:
   - Name: `FIREBASE_TOKEN`
   - Value: paste the token from step 2

4. That's it — every time you push a change to `database.rules.json` or `firebase.json` on `main`, the
   [`.github/workflows/deploy-firebase-rules.yml`](.github/workflows/deploy-firebase-rules.yml) workflow
   automatically deploys the rules.

### Option B — Manual (paste in console)

1. Open the Firebase console → **Realtime Database → Rules** tab.
2. Delete everything in the editor.
3. Open [`database.rules.json`](./database.rules.json) and paste its entire contents.
4. Click **Publish**.

### Option C — Manual (Firebase CLI)

```sh
npm install -g firebase-tools
firebase login
firebase deploy --only database
```

---

## 5. Update Your Project ID in the Repository

After creating your project, replace `YOUR_FIREBASE_PROJECT_ID` in [`.firebaserc`](./.firebaserc)
with your actual project ID (visible in the Firebase console under **Project settings → General**):

```json
{
  "projects": {
    "default": "your-actual-project-id"
  }
}
```

---

## 6. Update the Firebase Config in `index.html`

1. In the Firebase console, go to **Project settings → Your apps**.
2. If no web app exists yet, click **Add app → Web** and register it.
3. Copy the `firebaseConfig` object shown.
4. Open `index.html` and find the block starting at:
   ```js
   const firebaseConfig = {
   ```
5. Replace the entire object with your new values:
   ```js
   const firebaseConfig = {
     apiKey: "YOUR_NEW_API_KEY",
     authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
     databaseURL: "https://YOUR_PROJECT_ID-default-rtdb.REGION.firebasedatabase.app",
     projectId: "YOUR_PROJECT_ID",
     storageBucket: "YOUR_PROJECT_ID.appspot.com",
     messagingSenderId: "YOUR_SENDER_ID",
     appId: "YOUR_APP_ID"
   };
   ```

> **Note:** `storageBucket` enables profile photo uploads to Firebase Storage. Leave it as `""` only if you skip step 7.

---

## 7. (Optional) Enable Firebase Storage

If you want profile picture / avatar uploads:

1. In the Firebase console, click **Build → Storage → Get started**.
2. Choose the same region as your database.
3. Fill in `storageBucket` in `index.html` with the value from **Storage → Files** (e.g. `your-project.appspot.com`).

---

## Quick Checklist

- [ ] New Google account created
- [ ] Firebase project created
- [ ] Email/Password authentication enabled
- [ ] Anonymous authentication enabled
- [ ] Realtime Database created (same region preferred)
- [ ] Database rules deployed (Option A, B, or C above)
- [ ] `.firebaserc` updated with your project ID
- [ ] `index.html` `firebaseConfig` updated with your new config values
- [ ] (CI) `FIREBASE_TOKEN` secret added to GitHub → Actions secrets
- [ ] Pushed to `main` — workflow deployed rules automatically ✅
