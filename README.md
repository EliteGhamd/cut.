# Cut — Fat-Loss Tracker

A single-page web app: 7-day halal meal plan, 21 recipes, and daily tracking for calories, macros, water, and weight. Targets are set to **1,800 kcal / 200g protein**.

Data is stored per user in **Firebase Firestore** (anonymous auth). Before you add Firebase keys, it falls back to on-device storage so it still runs.

---

## 1. Create the Firebase project

1. Go to <https://console.firebase.google.com> → **Add project**.
2. Inside the project, open **Build → Firestore Database → Create database**. Start in **production mode**, pick a region.
3. Open **Build → Authentication → Get started → Sign-in method**, and **enable "Anonymous"**.
4. Open **Project settings (gear icon) → General → Your apps → Web (`</>`)**. Register an app, then copy the `firebaseConfig` object.

## 2. Paste your config

In `index.html`, find the `FB_CONFIG` block near the top of the first `<script>` and replace the `PASTE_…` values with your real config:

```js
const FB_CONFIG = {
  apiKey: "AIza…",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project-id",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abc123"
};
```

> These keys are **safe to commit publicly** — Firebase web keys identify the project, they don't grant access. Access is controlled by the security rules below.

## 3. Set Firestore security rules

In **Firestore → Rules**, paste this and publish. It lets each signed-in user read/write only their own document:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

## 4. Put it on GitHub Pages

```bash
git init
git add index.html README.md
git commit -m "Cut fat-loss app"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/cut-app.git
git push -u origin main
```

Then in the repo: **Settings → Pages → Source: Deploy from a branch → `main` / `root` → Save**.
Your app goes live at `https://YOUR_USERNAME.github.io/cut-app/`.

## 5. Allow your domain in Firebase

In **Authentication → Settings → Authorized domains**, add `YOUR_USERNAME.github.io` so anonymous sign-in works on the live site.

---

## How storage works

- The whole app state (logged meals, water, weight history) is saved as one JSON document at `users/{uid}` in Firestore.
- Saves are debounced (~0.35s) so rapid taps don't spam writes.
- The dot next to the logo shows status: **green** = synced to Firebase, **grey** = saved on this device, **red** = sync error.

## Optional: multi-device sign-in

Anonymous auth gives each *device* its own data. To carry data across devices, enable **Google** sign-in in Authentication and add a sign-in button — the storage code already keys everything by `uid`, so no data-layer changes are needed.

## Adjusting targets

Edit the `T` object in `index.html`:

```js
const T = { cal:1800, p:200, c:135, f:50, water:13 }; // water = 250ml glasses
```
