# AMORE — Firebase Setup Guide

Follow these exact steps to connect your AMORE store to Firebase.
This enables persistent product storage, secure admin login, and image uploads.

---

## Step 1 — Create a Firebase Project

1. Go to [https://console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project** → name it `amore-perfume` → Continue
3. Disable Google Analytics (optional) → **Create project**

---

## Step 2 — Enable Authentication

1. In the Firebase Console sidebar, click **Authentication** → **Get started**
2. Go to the **Sign-in method** tab
3. Click **Email/Password** → Enable it → **Save**
4. Go to the **Users** tab → **Add user**
   - Email: `admin@amore.com` (or your preferred admin email)
   - Password: choose a strong password (min 8 characters)
   - Click **Add user**

> ⚠️ Never expose this email/password in any public code or repository.

---

## Step 3 — Create Firestore Database

1. Click **Firestore Database** → **Create database**
2. Choose **Start in production mode** → **Next**
3. Select a region close to Egypt (e.g., `europe-west1`) → **Enable**

---

## Step 4 — Set Firestore Security Rules

1. In Firestore, go to the **Rules** tab
2. Replace the existing rules with the contents of `firestore.rules` in this project:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Products: public READ, authenticated admin WRITE only
    match /products/{productId} {
      allow read: if true;
      allow create, update, delete: if request.auth != null
        && request.auth.token.email_verified == true;
    }

    // Orders: admin only
    match /orders/{orderId} {
      allow read, write: if request.auth != null;
    }

    // Block everything else
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

3. Click **Publish**

---

## Step 5 — Enable Firebase Storage

1. Click **Storage** → **Get started**
2. Choose **Start in production mode** → **Next** → **Done**
3. Go to the **Rules** tab in Storage, replace with:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {

    // Product images: public read, admin write only
    match /products/{fileName} {
      allow read: if true;
      allow write: if request.auth != null
        && request.resource.size < 5 * 1024 * 1024
        && request.resource.contentType.matches('image/.*');
    }

    match /{allPaths=**} {
      allow read, write: if false;
    }
  }
}
```

4. Click **Publish**

---

## Step 6 — Register a Web App & Get Config

1. In Firebase Console, click the ⚙️ gear icon → **Project settings**
2. Scroll to **Your apps** → click the `</>` (Web) icon
3. Register app name: `AMORE Web Store` → **Register app**
4. Copy the `firebaseConfig` object shown — it looks like:

```js
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "amore-perfume.firebaseapp.com",
  projectId: "amore-perfume",
  storageBucket: "amore-perfume.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

---

## Step 7 — Add Config to Your Files

Open **BOTH** of these files and replace the placeholder `firebaseConfig`:

### `index.html` — find:
```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  ...
};
```

### `admin.html` — find the same block (same replacement)

Paste your **real** `firebaseConfig` values in both files.

---

## Step 8 — Seed Initial Products (Optional)

After connecting Firebase, open the Admin Console at `admin.html`, log in,
and use **Add New Perfume** to create your initial product catalogue.

Alternatively, you can seed products via the Firebase Console:
1. Go to Firestore → products collection → **Add document**
2. Auto-generate ID
3. Add fields: `name_en`, `name_ar`, `brand`, `price`, `stock`, `inStock`, `image`, `isFeatured`, `isBestSeller`, `category`, `categoryIds` (array), `createdAt` (timestamp)

---

## Step 9 — Deploy (Optional)

To host your site on Firebase Hosting:

```bash
npm install -g firebase-tools
firebase login
firebase init hosting
# Public directory: . (current folder)
# Single-page app: No
# Overwrite index.html: No
firebase deploy
```

Your site will be live at `https://YOUR_PROJECT_ID.web.app`

---

## Security Checklist

- [x] Admin account created via Firebase Console only (no public signup)
- [x] Firestore rules restrict writes to authenticated users
- [x] Storage rules enforce file size (5MB) and image type limits
- [x] Firebase config keys are safe to include client-side (they identify your project, not grant admin access)
- [x] Admin password stored securely in Firebase Auth (not in code)
- [ ] Optional: Enable App Check for additional abuse protection

---

## Files Modified

| File | Changes |
|------|---------|
| `index.html` | Firebase integration, real-time product loading, new AMORE logo |
| `admin.html` | NEW — full admin login + dashboard with Firebase Auth |
| `firestore.rules` | NEW — Firestore security rules |
| `FIREBASE_SETUP.md` | NEW — this setup guide |

---

## Support

WhatsApp: +20 11 49770420
