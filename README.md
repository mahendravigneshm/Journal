# Field Journal — setup

A personal journal that installs on your phone like an app and backs itself up to
your own Google Drive. No Anthropic API involved — it's a plain HTML/JS site that
talks directly to Google from your browser. Entries are stored locally on your
phone first (works instantly, no setup), and synced to Drive once you connect it.

There are two one-time setup steps: **hosting** the files somewhere with a stable
URL, and **connecting your own Google Drive**. Both take about 10 minutes combined.

---

## 1. Host the files

These are static files — any free static host works. Two easy options:

**Netlify Drop (fastest, no account strictly required)**
1. Go to https://app.netlify.com/drop
2. Drag this whole folder onto the page.
3. You'll get a URL like `https://random-name-123.netlify.app`. That's your app's
   permanent address (create a free Netlify account first if you want to keep the
   same URL long-term instead of a throwaway one).

**GitHub Pages (if you already use GitHub)**
1. Create a new repository, upload all the files in this folder to it.
2. Repo Settings → Pages → set source to the `main` branch, root folder.
3. Your URL will be `https://<your-username>.github.io/<repo-name>/`.

Either way — **write down the exact URL**, you'll need it in the next step.

---

## 2. Connect your own Google Drive

This makes the app able to read/write **only files it creates itself** in your
Drive (the `drive.file` scope) — it can't see your other files.

1. Go to https://console.cloud.google.com/ and create a new project (top-left
   project dropdown → New Project). Any name is fine.
2. **Enable the API**: search bar → "Google Drive API" → open it → **Enable**.
3. **Configure the consent screen**: left sidebar → APIs & Services → OAuth
   consent screen.
   - User type: **External**
   - Fill in app name (e.g. "Field Journal"), your email as support + developer
     contact.
   - On the Scopes step, add: `https://www.googleapis.com/auth/drive.file`
   - On the Test users step, add **your own Google account email**.
   - Leave it in **Testing** status — you don't need to publish or verify it for
     personal use.
4. **Create the credential**: left sidebar → APIs & Services → Credentials →
   **Create Credentials → OAuth client ID**.
   - Application type: **Web application**
   - Authorized JavaScript origins: paste the exact URL from Step 1 (no trailing
     slash), e.g. `https://random-name-123.netlify.app`
   - Create → you'll get a **Client ID** ending in `.apps.googleusercontent.com`.
     Copy it.
5. Open your app at the hosted URL → **Settings** tab → paste the Client ID →
   **Save Client ID** → **Connect & back up now**. Google will ask you to sign in
   and approve access the first time; after that it's remembered.

If you ever change hosts or URLs, add the new URL to the same OAuth client's
Authorized JavaScript origins list — the Client ID itself doesn't need to change.

---

## 3. Install it on your phone

**Android (Chrome):** open your URL → Chrome will usually show an "Install app"
banner; if not, tap the ⋮ menu → **Install app** (or **Add to Home screen**).

**iPhone (Safari):** open your URL in Safari (not Chrome — iOS requires Safari
for this) → tap the Share icon → **Add to Home Screen**.

Either way you get a real icon on your home screen that opens full-screen, no
browser chrome. It works offline for writing entries; Drive backup needs a
connection.

A couple of iOS quirks worth knowing: the Google sign-in step works most
reliably the first time if you do it in Safari itself before adding the icon to
your home screen. And iOS occasionally clears site data for installed web apps
that go unused for a week or more — your Drive backup is exactly the safety net
for that; use **Restore latest backup from Drive** on Settings if a device ever
comes up empty.

---

## What's actually happening, technically

- Every entry is written to `localStorage` on your device immediately — that's
  the fast path and it works with zero setup.
- After each save, the app makes a best-effort silent attempt to push a copy to
  Drive. If you're not connected yet, it just skips quietly rather than popping
  a login window uninvited.
- The Drive backup writes two files to your Drive root: `journal-data.json` (the
  real backup, used by "Restore") and `Field Journal.md` (a readable export you
  can open directly in Drive or Docs).
- Nothing passes through Anthropic, Claude, or any third-party server — it's
  your phone talking to `googleapis.com` directly, using credentials only you
  hold.
