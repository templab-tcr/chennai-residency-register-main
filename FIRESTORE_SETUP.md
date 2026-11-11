# Firestore setup (ledger-db-f93bd)

All data is saved to Firestore by the Apps Script backend (Code.gs), not by the web pages.
That keeps the database locked from browsers.

What gets saved:
- `guests/{reference}`: every guest and visitor registration (no photos, only Drive links)
- `hotel/main` and `hotel/main/parts/N`: the hotel state, saved each time the hotel app syncs

Note: the Firebase web config (apiKey etc.) is not needed for this. Only the project ID is.

## 1. Enable Firestore
1. Open https://console.firebase.google.com and select project **ledger-db-f93bd**.
2. Build > Firestore Database > Create database.
3. Choose **Native mode**, pick a location near India (asia-south1 Mumbai), then Create.
4. Open the **Rules** tab and paste the contents of `firestore.rules`, then Publish.

## 2. Create a service account key
1. Go to https://console.cloud.google.com, select project **ledger-db-f93bd**.
2. IAM & Admin > Service Accounts > Create service account. Name it `residency-sync`.
3. Grant the role **Cloud Datastore User**. Finish.
4. Open the new account > Keys > Add key > Create new key > JSON. A file downloads.
   Keep this file private. Do not put it in the GitHub repo.

## 3. Add the settings to Apps Script
Apps Script editor > Project Settings (gear icon) > Script properties > Add script property:
- `FIRESTORE_PROJECT_ID` = `ledger-db-f93bd`
- `FIRESTORE_SA_EMAIL` = the `srivijayaprasath@gmail.com` value from the JSON file
- `FIRESTORE_PRIVATE_KEY` = the `AIzaSyAEovhT9mT1rNbMIfIdXRWWVa_Yc9Nc7sg` value from the JSON file (the whole text, including `-----BEGIN PRIVATE KEY-----`)

## 4. Test the connection
1. In the Apps Script editor, choose the function `testFirestore` and click Run.
2. The first run asks for permission. Allow it.
3. Open the Firebase console > Firestore Database. A `system/test` document should appear.

## 5. Deploy the new code
Deploy > Manage deployments > edit the existing Web App > Version: New version > Deploy.
The web URL stays the same, so the HTML pages need no changes.

## 6. Check it works
Register one test guest from index.html. The record should appear in `guests` in Firestore
and in the Google Sheet as before. If Firestore fails, the guest is still saved to the Sheet
and the error is logged in Apps Script > Executions.

## Limits and things to know
- Changes made in the Sheet later (for example the visitor "Stop" timer) do not update the Firestore copy.
- The hotel state is split into parts of 200,000 characters. Each part is well under Firestore's 1 MiB document limit.
- Property checks are still saved only to the Sheet. Tell me if you want them in Firestore too.

## Security (important)
This project is in a public GitHub repo. Code.gs and the HTML files currently contain the
hotel token and the admin PIN (1212) in plain text. Anyone can read them.
- Change the admin PIN and the hotel token, then store them only in Script properties.
- Never add the service account JSON file or the private key to the repo.
