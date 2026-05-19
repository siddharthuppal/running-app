# Road to 10K

A progressive web app (PWA) for a structured 10-week running plan from couch to 10K. Progress syncs across devices via Firebase + Google Sign-In.

---

## Setup Firebase & Google Sign-In

### 1. Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and create a project.
2. Add a **Web app** to the project and copy the `firebaseConfig` object shown.
3. Create a **Cloud Firestore** database (start in production mode).
4. Open **Authentication → Sign-in method** and enable **Google**.

### 2. Add your Firebase config

Copy the example file and fill in your values:

```bash
cp firebase-config.example.js firebase-config.js
```

Then edit `firebase-config.js`:

```js
export const firebaseConfig = {
  apiKey: 'YOUR_API_KEY',
  authDomain: 'your-project.firebaseapp.com',
  projectId: 'your-project',
  storageBucket: 'your-project.firebasestorage.app',
  messagingSenderId: 'YOUR_SENDER_ID',
  appId: 'YOUR_APP_ID'
};

export const firebaseOptions = {
  enabled: true,
  appDocumentId: 'road-to-10k'
};
```

> **`firebase-config.js` is git-ignored.** Your API keys are never committed to the repository. Share the `firebase-config.example.js` template instead.

### 3. Set Firestore security rules

In the Firebase console → Firestore → Rules, paste:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/apps/{appId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

This ensures each user can only access their own data.

### 4. Add your domain to Auth authorized domains

In Firebase → Authentication → Settings → Authorized domains, add the domain you're hosting the app on (e.g. `yourdomain.com`). `localhost` is pre-authorized for local development.

---

## How it works

- Progress is saved to `localStorage` immediately on every tap.
- When Firebase is configured (`enabled: true`), the app shows a **Sign in with Google** screen.
- After sign-in, progress syncs to Firestore under `users/{uid}/apps/road-to-10k`.
- A live listener keeps multiple devices in sync in real time.
- Signing out clears the session; signing back in restores cloud progress.

---

## Security notes

- **Credentials**: `firebase-config.js` is `.gitignore`d. Firebase web API keys are safe to expose client-side as long as Firestore rules are set correctly — they identify the project, not grant admin access.
- **Firestore rules**: The rules above enforce that authenticated users can only read/write their own document. No user can access another's data.
- **Content Security Policy**: The app sets a CSP meta tag restricting scripts, styles, fonts, and network requests to trusted origins only.
- **No user input in DOM**: All training plan content is hardcoded; no user-supplied strings are rendered as HTML, eliminating XSS risk.

---

## Running locally

Just open `index.html` via a local HTTP server (required for ES modules and the service worker):

```bash
npx serve .
# or
python3 -m http.server 8080
```
