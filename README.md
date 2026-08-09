# Body & Nutrition Tracker

A single-file React app (`index.html`) — no build step, no `npm install`. It runs
straight from GitHub Pages by loading React, lucide-react and recharts from a
CDN (esm.sh) in the browser, and using in-browser Babel to transpile the JSX.

- **Hosting:** GitHub Pages
- **Data:** Firebase Firestore, one document per user per day
- **Sign-in:** Firebase email/password auth (your data is private to your account)
- **AI features:** your own Anthropic API key, entered once in Settings and stored only in your browser's `localStorage` — never sent anywhere but directly from your browser to Anthropic

---

## One-time setup

### 1. Firebase console

Go to [console.firebase.google.com](https://console.firebase.google.com) → your `body-tracker-5347e` project.

1. **Authentication → Sign-in method → Email/Password → Enable → Save.**
2. **Firestore Database → Create database** (if you haven't already) → start in **production mode** → pick a region.
3. **Firestore Database → Rules** → paste the contents of [`firestore.rules`](./firestore.rules) → **Publish**.

That's it on the Firebase side — no API keys or secrets to configure there; the app talks to Firebase directly over its public REST API, and the rules above are what actually keep your data private.

### 2. GitHub Pages

Repo → **Settings → Pages** → Source: **Deploy from a branch**, Branch: **main**, Folder: **/ (root)**. Save. The site goes live at:

```
https://anna36476.github.io/body-tracker/
```

Any push to `main` redeploys automatically within a minute or two.

### 3. First run

1. Open the site above.
2. Click **"No account yet? Create one"**, enter an email + password (this is your own account, stored in your Firebase project — nobody else can use it).
3. Click the **gear icon** (top right) → paste an Anthropic API key (get one at [console.anthropic.com/settings/keys](https://console.anthropic.com/settings/keys)) → **Save**.
4. Start logging meals.

Sign in again on any other device/browser with the same email + password and your data follows you (it lives in Firestore, not the browser). The Anthropic API key is per-browser, though — you'll need to re-enter it on each device.

---

## Making changes

Edit `index.html` directly (it's the whole app), commit, push to `main`:

```bash
git add index.html
git commit -m "Update app"
git push
```

GitHub Pages picks it up automatically. There's no build/compile step to run.

---

## How it works, briefly

- **No bundler:** the browser loads React/lucide-react/recharts as ES modules straight from esm.sh (see the `<script type="importmap">` in `index.html`), and `@babel/standalone` transpiles the inline JSX at load time. Fine for an app this size; not something you'd want for a large production app (it re-transpiles on every visit).
- **Auth:** plain calls to Firebase's Identity Toolkit REST API (`accounts:signUp` / `accounts:signInWithPassword` / token refresh) — no Firebase SDK needed.
- **Storage:** plain calls to the Firestore REST API. Every save is one document at `/users/{uid}/data/{key}` holding a single JSON-string field, mirroring the app's original local-storage-style `get/set/list(prefix)` calls.
- **AI calls:** go straight from your browser to `api.anthropic.com` with the key you entered, using the `anthropic-dangerous-direct-browser-access` header Anthropic provides for exactly this kind of client-only app. The key never touches Firebase or GitHub.

---

## Troubleshooting

**"Email/password sign-in isn't enabled yet"** — you skipped step 1.1 above; enable it in the Firebase console.

**Data doesn't load / a permission error in the console** — Firestore rules aren't published yet (step 1.3), or you're signed into a different account than the one that saved the data.

**"Add your Anthropic API key in Settings first"** — click the gear icon, paste a key from console.anthropic.com.

**Changes don't show up on the live site** — GitHub Pages can take a minute or two after a push; check the repo's **Actions** tab for the Pages deployment status.

**Anthropic API calls failing with a CORS or 401 error** — double check the key was pasted correctly (no extra whitespace) and that it's active in the Anthropic console.
