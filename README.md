# Pottery Tracker

A small web app for keeping track of finished pottery: photos of each piece, which glazes and clay went on it and how, and notes. A Glazes tab holds a photo of each glaze and notes on whether she liked it. Works in any browser and can be added to a phone or tablet home screen like an app.

- **The app** is static files hosted free on GitHub Pages.
- **The data** lives in a free Google Firebase project (Firestore database). Everyone who signs in sees and edits the same records, on any phone or tablet.
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
3. Under **Users → Add user**, enter an email and a password for each person who should use the app. Each one is a login; they all share the same records.
4. Go to **Settings → User actions**, untick **Enable create (sign-up)**, and click **Save**. There's no sign-up form in the app either, so only the people added in step 3 can get in.

### 3. Create the database
1. **Build → Firestore Database → Create database**. Pick a location near you (e.g. `us-east1`) and start in **production mode**.
2. Open the **Rules** tab, replace everything with the contents of `firestore.rules` from this repo, and click **Publish**. This lets anyone who is signed in read and change the records, and keeps everyone else out.

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

**Pieces** each have photos (up to 10, plus a photo of the bottom), a name, and these sections, which start closed and show a one-line summary of what's filled in:
- **About the piece:** form (mug, bowl, platter… or type your own), size XS–XL, technique (hand built, slab built, wheel thrown, altered), slab thickness, and marks: best / favorite combo, runner, failed.
- **Status and dates:** thrown, trimmed, bisqued, glazed, finished. The latest date is the piece's status, so unfinished pieces show "Bisqued" and so on.
- **Clay:** clay body, new or recycled, grams used, weight, size before and after firing, and how many ounces it holds.
- **Glazes and decoration:** glazes in the order they went on, each with coats, how (brushed, dipped, poured, squeeze bottle, spatter… or type your own) and where (inside, bottom 2/3…). ↑ moves a glaze earlier. Also the layering technique (frozen pond, overlapping randomly…), a description, decorations (slip, underglaze, refire…), decorative techniques and tools.
- **Firing:** firing type (oxidation, reduction, wood, soda, raku, luster), bisque and glaze temperatures.
- **Price range:** low and high.
- **My comments.**

Each new photo opens on a crop screen first: drag the corners in around the part to keep (or tap **Square** or **Whole photo**), then **Use photo**. On each photo, **★** makes it the cover, **↻** turns it a quarter turn, **×** removes it. On a card, swipe the photo (or tap ‹ ›) to see the others, and tap it to see it full screen. Tap the rest of the card to edit. The top of the tab shows how many pieces there are.

Search finds pieces by any word: name, form, glaze name or color, how it went on, notes. "blue mug" finds mugs with a blue glaze. The filters narrow to one glaze, one form, one mark, or in-progress / finished pieces.

**Glazes & supplies** holds glazes, underglazes, Stroke & Coat, stains, low-fire glazes, slips and clay bodies. Each has a photo (a test tile or swatch), a name, what kind it is and **Did you like it?**, then closed sections for buying (manufacturer, size, price, best place to buy), color and firing (color, fire and bisque temperature, the clay it works on, whether it's a runner; shrinkage for clay bodies) and notes (description, recipe, notes). The buttons at the top show one kind, one rating, or the runners. **Used on N pieces** jumps to those pieces. Deleting one doesn't change the pieces that used it; they keep its name.

Glazes are picked for a piece from a list with their photos. A glaze that isn't in the list yet can be typed in the list's search box; it's added when the piece is saved. **+ New clay…** on a piece adds a clay body the same way.

**Updates** reach her devices by themselves: when the app comes back on screen (and every 15 minutes while open) it checks for a newer version and reloads, but never while a popup is open or she's typing. The version under the title is the date and time it was published.

## What's stored
```
studio/shared/pieces/{id}     name, form, status dates, clay, glazes in order, firing, price, comments, photo ids
studio/shared/glazes/{id}     glazes, underglazes, clay bodies and other supplies, each with a photo id
studio/shared/photos/{id}     one photo each
studio/shared/settings/studio older clay body list (clay bodies are now supplies)
```

**Photos** are shrunk to about 900 px and kept in Firestore in their own records, so they need no paid Firebase Storage plan and work offline. Loading the app reads only the pieces and glazes; each photo is downloaded once per device when it's first shown, then read from the device's copy. A photo is never changed once saved (a rotated photo is saved as a new one), which is what makes that copy safe to reuse.

The free plan allows 50,000 reads and 20,000 writes per day and 1 GB of storage. At about 150 KB per photo, that's room for several thousand photos.

## Backups
**Download backup** on each tab saves the pieces or supplies as a spreadsheet (CSV) file. Doing that now and then gives her a copy outside Firebase. Photos aren't in the file.

## Testing locally
Modules don't load from `file://`, so serve the folder:
```
python -m http.server 8000
```
then open http://localhost:8000. `localhost` is already an authorized domain in Firebase.
