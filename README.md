# Field Records

Installable web app (PWA) for tracking field inputs, harvest, sales and expenses.
A second app, **Grain Drying** (`/drying/`), logs grain dryer meter readings from photos —
see [Grain Drying](#grain-drying) below.
Data syncs to Firebase Realtime Database under `users/<uid>/fieldRecords`, and a copy
is kept on the device so the app opens and takes edits without signal. Unsynced edits
upload automatically when the connection comes back.

## Layout

```
public/              ← everything that gets deployed
  index.html         the app
  sw.js              service worker (offline support)
  manifest.json      install metadata
  drying/            Grain Drying app (index.html, sw.js, manifest.json)
  icon-*.png, icon.svg
tools/generate-icons.html   regenerates the PNG icons if you change the design
database.rules.json         recommended Realtime Database rules
firebase.json, .firebaserc  Firebase Hosting config
```

## Deploy

```bash
npm install -g firebase-tools
firebase login
firebase deploy --only hosting
```

> **Heads-up:** `.firebaserc` points at the `shopping-list-app-d077b` project. If the
> shopping list app is already hosted at `shopping-list-app-d077b.web.app`, this deploy
> **replaces it**. To keep both, create a second site and deploy there instead:
>
> ```bash
> firebase hosting:sites:create field-records-<something-unique>
> firebase target:apply hosting fieldrecords field-records-<something-unique>
> ```
> then add `"target": "fieldrecords"` inside `"hosting"` in `firebase.json`.

After a change to `index.html`, bump `CACHE` in `public/sw.js` (e.g. `v2` → `v3`) so
installed copies drop their old cache.

## Database rules

`database.rules.json` limits each account to its own `users/<uid>` data. It is **not**
deployed automatically, because the Realtime Database is shared with the shopping list
app and deploying would replace that app's rules too. Merge it into the existing rules
in the Firebase console (Realtime Database → Rules).

## Install on a phone

- **Android (Chrome):** open the site → ⋮ menu → *Add to Home screen* / *Install app*.
- **iPhone (Safari):** open the site → Share → *Add to Home Screen*.

## Grain Drying

`https://<your-site>/drying/` is a separate app you install on its own (open that address,
then *Add to Home Screen*). It uses the same sign-in accounts as Field Records.

**How it's used:** tap **📷 Take Picture** and photograph the electric meter, the gas meter
and the dryer screen. Each photo is read by Claude and the numbers fill in
by themselves. Type who it's for and tap **Save START** or **Save END** — that saves the
reading. The app highlights the likely one: if that person has a job started and not
ended, it suggests END. For clock-style electric dials, Claude reports where each pointer
is and the app applies the usual carry rule (a pointer only counts as past a number once
the dial to its right has passed 0).
The **Log** tab pairs each start with its end and shows, per job and per person:
electric and gas used (end − start, since those meters count up), and **job hours**,
**batch count** and burner hours (the end reading, since those are zeroed at the start of
each job). Dryer hours (lifetime) is recorded too. Hours are kept as hours:minutes, the
way the dryer screen shows them. It exports a CSV for a spreadsheet.

Without signal, readings (with photos) are kept on the phone and upload by themselves
later; the numbers have to be typed in, since reading photos needs the internet.

### One-time setup

1. **Database rules.** Merge the `grainDrying` block from `database.rules.json` into the
   Realtime Database rules in the Firebase console. Unlike Field Records (one copy per
   account), the drying log is shared by everyone on the member list.
2. **Add the users.** Each of the 3 people signs in to `/drying/` once (creating an
   account if needed). The app shows a *Waiting for access* screen with their ID. In the
   Firebase console → Realtime Database → Data, add at the top level:

   ```
   grainDryingMembers
     <their ID>: true
   ```

   then they tap **Try Again**.
3. **Claude API key.** Create a key at console.anthropic.com (add a little credit;
   each photo costs about a cent) and paste it in **Settings** in the app once. It's
   stored in the shared log, so the other users don't need to enter it — and anyone on
   the member list can see it, so only add people you trust and set a monthly spend
   limit on the key.
4. If an electric or gas meter face says "multiply by …", enter that in **Settings**.

Data lives under `grainDrying/readings` (numbers) and `grainDrying/photos` (one
compressed JPEG per photo, loaded only when a reading is opened).

