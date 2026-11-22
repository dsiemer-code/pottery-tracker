# Pottery Tracker

A small web app for keeping track of finished pottery: photos of each piece, which glazes and clay went on it and how, and notes. A Glazes tab holds a photo of each glaze and notes on whether she liked it. Works in any browser and can be added to a phone or tablet home screen like an app.

- **The app** is static files hosted free on GitHub Pages.
- **The data** lives in a free Google Firebase project (Firestore database), tied to her sign-in. Her phone and tablet see the same records.
- **Offline:** the app keeps a copy on the device. Changes made with no signal are saved on the device and sync once it's back online. She does need to be online the first time she signs in on a device.

Until `firebase-config.js` is filled in, the app runs in **demo mode** with sample data and saves nothing. That's a good way to try it first.

## Setup (about 15 minutes, one time)

### 1. Create the Firebase project
Make a **new** Firebase project for this app rather than reusing the Plushie Ledger one. It keeps the logins and data apart and costs nothing.

1. Go to https://console.firebase.google.com and sign in with a Google account.
2. **Create a project** (name it e.g. `pottery-tracker`). Google Analytics can be turned off.
3. Stay on the free **Spark** plan. No card is needed.

### 2. Turn on sign-in
1. **Build → Authentication → Get started**.
2. Under **Sign-in method**, enable **Email/Password**.
3. Under **Users → Add user**, enter her email and a password. This is her login.
4. There's no sign-up form in the app, so nobody else can create an account.

### 3. Create the database
1. **Build → Firestore Database → Create database**. Pick a location near you (e.g. `us-east1`) and start in **production mode**.
2. Open the **Rules** tab, replace everything with the contents of `firestore.rules` from this repo, and click **Publish**. This makes sure each person can only see their own data.

### 4. Connect the app
1. **Project settings** (gear icon) → **General** → **Your apps** → click the **`</>`** (Web) icon. Register the app (no Firebase Hosting needed).
2. Copy the `firebaseConfig` values it shows into `firebase-config.js`.
   These values are meant to be public. The sign-in and the rules are what protect the data.

### 5. Put it on GitHub Pages
1. Create a repository and upload all the files in this folder (keep them at the top level).
2. Repo **Settings → Pages** → Source: **Deploy from a branch**, branch `main`, folder `/ (root)`. Save.
3. After a minute the site is live at `https://dsiemer-code.github.io/pottery-tracker/` (if the repo is named `pottery-tracker`).
4. Back in Firebase: **Authentication → Settings → Authorized domains → Add domain** and add `dsiemer-code.github.io`. Without this, sign-in is refused on the live site.

### 6. Put it on her phone and tablet
Open the GitHub Pages link on each device and sign in.
- **iPhone / iPad (Safari):** Share → **Add to Home Screen**.
- **Android (Chrome):** ⋮ menu → **Add to Home screen** / **Install app**.

## Using it

**Pieces** are finished pots. Each one has a name, the date it was finished, up to 10 photos, the clay body, the glazes used, and notes. For each glaze there's a **How** line for how it went on (e.g. "2 dips", "brushed 3 coats, rim only"). On a phone, **Add photos** offers the camera or the photo library. On each photo, **★** makes it the cover (the one on the card), **↻** turns it a quarter turn, and **×** removes it. Tap a photo on a card to see it full screen; tap the rest of the card to edit.

Glazes are picked from a list with their photos. A glaze that isn't in the Glazes tab yet can be typed in the list's search box and added as a new glaze; it's added to the Glazes tab when the piece is saved. **+ New clay…** works the same way for clay.

Search finds pieces by name, notes, clay, glaze, or how a glaze was applied. **All glazes** narrows the list to pieces with one glaze.

**Glazes** each have a photo (a test tile or a pot), a brand or recipe, **Did you like it?** (Love it / It's OK / Not for me) and notes. The buttons at the top show only the glazes she loved, or only the ones she didn't. **Used on N pieces** jumps to those pieces. Deleting a glaze doesn't change the pieces that used it; they keep its name.

**Updates** reach her devices by themselves: when the app comes back on screen (and every 15 minutes while open) it checks for a newer version and reloads, but never while a popup is open or she's typing. The version under the title is the date and time it was published.

**Clay bodies** are a short list at the bottom of the Glazes tab, used for the clay choice on each piece.

## What's stored
```
users/{her-account-id}/pieces/{id}     name, date finished, clay, glazes (glaze, name, how), notes, photo ids
users/{her-account-id}/glazes/{id}     name, brand or recipe, liked it, notes, photo id
users/{her-account-id}/photos/{id}     one photo each
users/{her-account-id}/settings/studio clay body list
```

**Photos** are shrunk to about 900 px and kept in Firestore in their own records, so they need no paid Firebase Storage plan and work offline. Loading the app reads only the pieces and glazes; each photo is downloaded once per device when it's first shown, then read from the device's copy. A photo is never changed once saved (a rotated photo is saved as a new one), which is what makes that copy safe to reuse.

The free plan allows 50,000 reads and 20,000 writes per day and 1 GB of storage. At about 150 KB per photo, that's room for several thousand photos.

## Backups
**Download backup** on each tab saves the pieces or glazes as a spreadsheet (CSV) file. Doing that now and then gives her a copy outside Firebase. Photos aren't in the file.

## Testing locally
Modules don't load from `file://`, so serve the folder:
```
python -m http.server 8000
```
then open http://localhost:8000. `localhost` is already an authorized domain in Firebase.
