# Christmas Holiday Potluck Sign-Up 🎄

A single-file, mobile-first potluck sign-up page. No build step, no dependencies to install. Open `index.html` and it works.

Out of the box it runs in **Local Demo Mode** (data lives only in your own browser). Follow the 4 steps below to make it live for every guest.

## Customize the event

Open `index.html` and edit the `EVENT_CONFIG` object at the top of the `<script>` section: title, date, time, location, host note, and how many dishes you want per category (`targetCounts`).

> Link previews (WhatsApp, iMessage) read the `<meta property="og:...">` tags near the top of the file, not the JavaScript. If you change the title or date, update those two tags too.

## Guests bringing more than one dish

In the RSVP form, **+ Add another dish** adds a second (up to six) dish card, each with its own category, name, dietary tags and note. The family's headcount is entered once and counted once. Each dish appears under its own category with an "Also bringing:" line linking it to the family's other dishes, and the "spots filled" counters count dishes. Removing one dish (trash icon) leaves the rest of the RSVP alone. Removing a family's only dish removes the whole entry.

**Returning guests:** as someone types their name, guests who already RSVP'd are suggested, with their headcount and dishes. Tapping a suggestion switches the form to "Add another dish" for that RSVP: the headcount is hidden and unchanged, and the new dish is added to their existing entry. A name that isn't picked from the list is a new guest with their own headcount. If someone types a name that exactly matches an existing RSVP without picking it, they get a one-tap "Add a dish to … instead" nudge (it's a nudge, not a rule). If two guests share a name, both are listed with their own headcount so people can tell them apart.

Entries saved by earlier versions of this page (one dish each) still display and are upgraded the next time they're edited.

## Host view: full guest list + CSV

Add `?host=1` to the end of your page link, for example `https://<your-username>.github.io/<repo-name>/?host=1`, and a **Host view** panel appears under the RSVP button:

- **View all guests** shows every RSVP in one alphabetical list with headcount, category, dish, dietary tags and notes, plus total people.
- **Download CSV** saves a spreadsheet (`potluck-guests-YYYY-MM-DD.csv`) that opens in Excel, Numbers or Google Sheets. It has one row per dish; a family bringing several dishes gets several rows, with their headcount on the first row only, so adding up the "Total people" column never counts anyone twice.

Share the normal link (without `?host=1`) with guests. This is a convenience, not a password: anyone who adds `?host=1` can see the panel, but it only shows what's already on the page, so nothing extra is exposed.

## Step 1 · Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and sign in with a Google account.
2. Click **Create a project**, name it (e.g. `christmas-potluck`), and turn Google Analytics **off**. The free Spark plan is plenty.

## Step 2 · Create the Realtime Database and set its rules

1. In the left menu choose **Build → Realtime Database → Create Database**. Pick a region and start in **locked mode**.
2. Open the **Rules** tab, replace everything with the rules below, and click **Publish**:

```json
{
  "rules": {
    "potluck_rsvps": {
      ".read": true,
      "$id": {
        ".write": true,
        ".validate": "newData.hasChildren(['name','adults','kids','dishes','createdAt'])",
        "name":   { ".validate": "newData.isString() && newData.val().length > 0 && newData.val().length <= 60" },
        "adults": { ".validate": "newData.isNumber() && newData.val() >= 1 && newData.val() <= 20" },
        "kids":   { ".validate": "newData.isNumber() && newData.val() >= 0 && newData.val() <= 20" },
        "dishes": {
          ".validate": "newData.numChildren() >= 1 && newData.numChildren() <= 6",
          "$i": {
            ".validate": "newData.hasChildren(['category'])",
            "category": { ".validate": "newData.val() == 'appetizer' || newData.val() == 'main' || newData.val() == 'dessert' || newData.val() == 'other'" },
            "dish":     { ".validate": "newData.isString() && newData.val().length <= 80" },
            "note":     { ".validate": "newData.isString() && newData.val().length <= 140" },
            "tags":     { "$t": { ".validate": "newData.isString() && newData.val().length <= 20" } },
            "$other":   { ".validate": false }
          }
        },
        "createdAt": { ".validate": "newData.isNumber()" },
        "updatedAt": { ".validate": "newData.isNumber()" },
        "$other":    { ".validate": false }
      }
    }
  }
}
```

Each RSVP (one family, counted once in the headcount) holds 1 to 6 dishes. These rules let anyone with the link add, edit, and delete individual entries (that's how a no-login potluck list works), while rejecting malformed or oversized data and blocking a single request from wiping the whole list. Avoid Firebase's "test mode" rules, because they expire after 30 days.

## Step 3 · Connect the app

1. In **Project settings (gear icon) → General → Your apps**, click the **Web** icon `</>` and register an app (skip Firebase Hosting).
2. Copy the `firebaseConfig` values it shows you into `EVENT_CONFIG.firebaseConfig` in `index.html`.
3. Make sure `databaseURL` is filled in (it looks like `https://your-project-default-rtdb.firebaseio.com`). If it's missing, copy it from the top of the Realtime Database **Data** tab.

The amber "Demo Mode" banner disappears and a green **● Live** badge appears once the connection works.

## Step 4 · Host it on GitHub Pages

1. Create a new public repository on GitHub and upload `index.html` (and this README) to the `main` branch.
2. Go to **Settings → Pages**, set **Source** to *Deploy from a branch*, choose `main` / `/ (root)`, and save.
3. After about a minute your site is live at `https://<your-username>.github.io/<repo-name>/`. Paste that link into your WhatsApp group.

## Notes

- **Firebase keys in the page are normal.** Web config values identify your project and are not secrets; your database rules are what protect the data. Don't put admin credentials anywhere in this file.
- **Anyone with the link can edit or delete any entry.** If a guest messes up, you can fix or clear entries directly in the Firebase console under **Realtime Database → Data → `potluck_rsvps`**.
- **Testing locally first?** Just double-click `index.html`. To reset the demo data, clear this site's storage in your browser (or delete the `potluck_rsvps_demo_v1` key).
