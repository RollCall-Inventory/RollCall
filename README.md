# RollCall
### Wide Format Paper Inventory Scanner for Print Shops

RollCall is a mobile web app designed for print shop copy centers. Scan the barcode on a paper roll, enter the quantity on hand, and your inventory Google Sheet updates instantly — no app store, no installs, no cost.

---

## Screenshots


| Welcome Screen | Scanner | Quantity Entry | Sheet Updated |
|---|---|---|---|
| <img src="https://github.com/user-attachments/assets/451dcf20-eeb6-4a5e-957e-b63659a1f800" width="200"> | <img src="https://github.com/user-attachments/assets/dc18a4a3-f90f-4204-ab31-dae899a2eac6" width="200"> | <img src="https://github.com/user-attachments/assets/4bc3103f-47bb-4776-8477-a9c5f9f8815e" width="200"> | <img src="https://github.com/user-attachments/assets/80d66e16-9fff-4e54-85a7-478d2f779114" width="200"> |
---

## What It Does

- 📷 **Barcode scanning** — point your iPhone camera at any paper roll and it's identified instantly
- 📊 **Live Google Sheet sync** — quantities update in real time with 🟢 OK / 🟡 Low / 🔴 Order Now status
- 🏷️ **Register new barcodes on the spot** — unknown rolls show a dropdown to assign and save the barcode automatically for next time
- 📋 **Manual select** — full paper list always available even without scanning a barcode
- 💾 **Stays connected** — your Google Sheet URL is saved to your device so setup only happens once

---

## Requirements

- A smartphone or tablet with a modern browser
  - iPhone / iPad — use **Safari** or the **Google app**
  - Android — use **Chrome** or the **Google app**
- A Google account
- 5 minutes for initial setup

RollCall runs entirely in the browser. No app download required.

> ⚠️ **iPhone and iPad users avoid Chrome.** Chrome on iOS does not support camera access for web apps.

---

## Setup Guide

### Step 1 — Copy the Google Sheet Template

Make a copy of the RollCall inventory template:

👉 **[RollCall Google Sheet Template](https://docs.google.com/spreadsheets/d/1NK6T7wVnGJrK1A2k0PtHJ72AS1R-x-47_nPIOApGqGk/copy)**

Once copied, keep the first sheet tab named **Inventory**. This is where RollCall will write your quantity updates.

---

### Step 2 — Set Up Google Apps Script

Google Apps Script acts as the bridge between RollCall and your Google Sheet. This is a one-time setup.

1. In your Google Sheet, go to **Extensions → Apps Script**
2. Delete any existing code in the editor
3. Paste the following script in full:

```javascript
const SHEET_NAME = "Inventory";

function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getSheetByName(SHEET_NAME);
    const values = sheet.getDataRange().getValues();

    for (let r = 0; r < values.length; r++) {
      for (let c = 0; c < values[r].length; c++) {
        const cell = String(values[r][c]).trim();
        const searchName = String(data.name || '').trim();
        if (cell === searchName || cell.includes(searchName)) {
          sheet.getRange(r + 1, 4).setValue(data.qty);
          return ContentService
            .createTextOutput(JSON.stringify({ status: "ok", row: r + 1 }))
            .setMimeType(ContentService.MimeType.JSON);
        }
      }
    }
    return ContentService
      .createTextOutput(JSON.stringify({ status: "not_found" }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch(err) {
    return ContentService
      .createTextOutput(JSON.stringify({ status: "error", message: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

4. Click **Deploy → New deployment**
5. Configure the deployment as follows:

| Setting | Value |
|---|---|
| Type | Web app |
| Execute as | **Me** |
| Who has access | **Anyone** |

6. Click **Deploy**
7. Copy the **Web app URL** — you will need this in Step 3

> ⚠️ **Common mistakes:**
> - "Execute as" must be set to **Me** — not "User accessing the web app"
> - "Who has access" must be **Anyone** — not "Anyone with Google account"
> - After any future code changes, always select **New version** when redeploying — saving the script file alone does not update the live URL

---

### Step 3 — Connect RollCall to Your Sheet

1. Open **[RollCall](https://maxien1.github.io/rollcall)** in Safari on your iPhone
2. Tap the URL field and paste your Apps Script Web app URL
3. Tap **Let's Go**
4. Allow camera access when prompted

Your URL is saved to your device. Every time you open RollCall after this it will go straight to the scanner — no setup needed.

---

### Step 4 — Register Your Barcodes

The first time you scan each paper roll, RollCall won't recognize the barcode yet. Here's what to do:

1. Scan the barcode on the packaging
2. When the **New Barcode Scanned** screen appears, select the correct paper type from the dropdown
3. Tap **Register & Set Quantity**
4. Enter the current quantity and tap **Update Google Sheet**

From that point on, scanning that roll will identify it automatically. You only need to register each barcode once per device.

---

## Paper Types Included

| Section | Paper Type | Max Stock |
|---|---|---|
| Blueprint | 20 lb Bond (24") | 4 |
| Blueprint | 20 lb Bond (36") | 4 |
| 24″ Rolls | Universal Heavyweight | 2 |
| 24″ Rolls | Super Heavyweight | 2 |
| 24″ Rolls | Photo Gloss | 2 |
| 24″ Rolls | Photo Satin | 2 |
| 24″ Rolls | Matte Polypropylene | 2 |
| 24″ Rolls | Scrim Vinyl | 2 |
| 36″ Rolls | Universal Heavyweight | 4 |
| 36″ Rolls | Super Heavyweight | 4 |
| 36″ Rolls | Photo Gloss | 4 |
| 36″ Rolls | Photo Satin | 4 |
| 36″ Rolls | Matte Polypropylene | 4 |
| 36″ Rolls | Scrim Vinyl | 4 |
| 36″ Rolls | Wrapping Paper (30") | 4 |

To add or remove paper types, edit the `MASTER_PAPERS` array in `index.html`.

---

## Hosting Your Own Copy

Want to run your own version — for a different store, a different paper lineup, or just your own setup? Fork this repo and host it for free on GitHub Pages.

1. Click **Fork** at the top right of this page
2. Go to your forked repo → **Settings → Pages**
3. Under Source, select **Deploy from branch**
4. Set branch to **main** and folder to **/ (root)**
5. Click **Save**
6. Your app will be live at `https://[your-username].github.io/rollcall`

No servers. No hosting fees. No maintenance. GitHub Pages is free for public repos.

---

## Troubleshooting

**"Connection error" when updating the sheet**
- Confirm Apps Script is deployed with **Execute as: Me** and **Who has access: Anyone**
- If you recently edited the script, make sure you deployed a **New version** — the live URL does not update automatically when you save

**Camera won't start**
- RollCall must be opened over HTTPS — GitHub Pages handles this automatically
- On iPhone, use **Safari** — Chrome on iOS does not support camera access for web apps
- If prompted, allow camera permissions. If you previously denied them, go to **Settings → Safari → Camera** and set it to Allow

**Paper shows as "not found" in the sheet**
- The paper name sent by the app must match what is in your Google Sheet exactly
- Check for extra spaces, different capitalization, or special characters

**Barcodes not saving between sessions**
- Make sure you are not using Safari in Private Browsing mode — localStorage does not persist in private tabs
- Clearing browser history or data in Safari will remove saved barcodes and your Apps Script URL

---

## Tech Stack

| Component | Technology |
|---|---|
| App | HTML / CSS / JavaScript (single file) |
| Barcode scanning | [QuaggaJS](https://github.com/serratus/quaggaJS) |
| Hosting | GitHub Pages (free) |
| Backend | Google Apps Script |
| Database | Google Sheets |

---

## suggestions & Feedback

Have an idea for a new feature, a paper type to add, or something that isn't working right for your shop? We'd love to hear from you.

📬 **[rollcall.inventory@gmail.com](mailto:rollcall.inventory@gmail.com)**

---

## License

MIT — free to use, modify, and share.
