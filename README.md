# Matchday Ledger

Your EA FC co-op stats tracker — shared live data via Firebase, screenshot scanning via Tesseract.js OCR, hosted free on GitHub Pages.

## 1. Add your Firebase config

Open `index.html`, find this block near the top of the `<script>` section:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

Replace it with the real object Firebase gave you (Project settings → Your apps → the web app you registered).

## 2. Deploy to GitHub Pages

1. Push `index.html` (this file's folder) to your GitHub repo — either via the GitHub website's "Add file → Upload files" button, or with git:
   ```
   git init
   git add .
   git commit -m "Matchday Ledger"
   git branch -M main
   git remote add origin https://github.com/<you>/<repo>.git
   git push -u origin main
   ```
2. In the repo, go to **Settings → Pages**.
3. Under "Build and deployment", set **Source** to "Deploy from a branch", branch `main`, folder `/ (root)`.
4. Save. GitHub gives you a URL like `https://<you>.github.io/<repo>/` within a minute or two — that's your permanent link.
5. Any time I hand you an updated `index.html`, just re-upload/push it and the same link updates automatically.

## 3. Sign-in (only you and your buddy, via Google)

The app has a login screen that uses "Sign in with Google" — no separate password to manage. Set this up in Firebase:

1. **Authentication → Sign-in method**, enable **Google**. It'll ask for a "Support email for project" — pick your own Google account.
2. **Authentication → Settings → Authorized domains** — add your GitHub Pages domain (`<you>.github.io`, no `https://`), or Google's sign-in popup will reject the live site.
3. Go to **Firestore Database → Rules**, replace the rule with the one below (put your and your buddy's real emails, lowercase, in the list), and click Publish:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if request.auth != null
           && request.auth.token.email.lower() in ['your_email@gmail.com', 'buddys_email@gmail.com'];
       }
     }
   }
   ```

**Your repo is public, so nothing with your real emails in it belongs there.** This is why the app's code doesn't contain an email list at all — the rule above, which lives only in your private Firebase console (never pushed to GitHub), is the one and only place those addresses need to exist. If someone signs in with a Google account that isn't on that list, Firestore itself refuses to hand over any data, and the app shows a generic "this account doesn't have access" message and signs them back out.

## 4. Firestore security rules (superseded by step 3)

Firebase's "test mode" rules allow anyone with your project ID to read and write your data, and **expire after 30 days** (after which nothing will load). For a two-person hobby tracker with no sensitive data, the practical risk is low, but you have two options:

- **Do nothing extra, just extend the expiry:** in Firebase Console → Firestore Database → Rules, replace the rule with:
  ```
  rules_version = '2';
  service cloud.firestore {
    match /databases/{database}/documents {
      match /{document=**} {
        allow read, write: if true;
      }
    }
  }
  ```
  This never expires, but is still fully open to anyone who finds your project ID.
- **Add a shared PIN check** (a bit more setup, meaningfully more private) — ask me and I'll wire this in.

## What's in this folder

- `index.html` — the whole app
- `zermatt-scene.webp` / `zermatt-scene.jpg` — the login screen's background image (WebP loads first where supported, JPEG is the fallback). **Both must sit in the same folder as `index.html` in your repo** — the page references them by relative path.

Single self-contained app — no build step, no npm install. Open `index.html` locally in a browser to test before deploying (Firestore sync still works over `file://`, though a couple of browsers restrict local file access — GitHub Pages is the real test). The two image files need to be uploaded too, not just the HTML.

