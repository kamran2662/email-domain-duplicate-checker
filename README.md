# Email Domain Duplicate Checker

A GitHub-ready email intake dashboard backed by Google Sheets. It checks **domains**, not full email addresses:
- `support@facebook.com` → domain `facebook.com`
- `sales@facebook.com` → duplicate if `facebook.com` already exists, regardless of the email username.

The Google Apps Script backend uses `LockService` around the check-and-insert operation to prevent simultaneous requests from inserting the same domain twice.

## Project layout

- `dashboard/` — static dashboard (host on GitHub Pages)
- `apps-script/Code.gs` — Google Apps Script API (connects to Google Sheets)
- `dashboard/config.js` — set your deployed Apps Script URL here

## 1. Create the Google Sheet

1. Create a new Google Sheet.
2. Copy its spreadsheet ID from its URL:
   `https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit`
3. Create a tab named `Emails` (or use another name and update `SHEET_NAME` in `Code.gs`).
4. Add these headers in row 1:

   `Domain | Email | Added At`

## 2. Deploy the Google Apps Script backend

1. Open [script.google.com](https://script.google.com/) and create a new project.
2. Replace the contents of `Code.gs` with `apps-script/Code.gs`.
3. In **Project Settings → Script Properties**, add:
   - `SPREADSHEET_ID` = your Google Sheet ID
   - `API_KEY` = a long random secret (at least 32 characters)
4. In `Code.gs`, confirm `SHEET_NAME = 'Emails'`.
5. Run `setupSheet` once from the editor and approve the requested Google Sheets permissions.
6. Select **Deploy → New deployment → Web app**:
   - Execute as: **Me**
   - Who has access: **Anyone**
7. Deploy and copy the Web app URL (ends in `/exec`).

Important: Anyone who obtains the web app URL *and* your API key can submit entries. Keep the API key private. Do not put it in a public GitHub repository.

## 3. Configure the dashboard

1. Open `dashboard/config.js`.
2. Replace `PASTE_YOUR_APPS_SCRIPT_WEB_APP_URL_HERE` with your `/exec` URL.
3. Open `dashboard/index.html` in a browser to test locally, or deploy the `dashboard/` folder using GitHub Pages.
4. When the dashboard loads, paste your API key into the **Admin API key** field. It is saved only in that browser's `sessionStorage` and is not committed to GitHub.

## 4. Publish on GitHub Pages

1. Create a GitHub repository, e.g. `email-domain-checker`.
2. Upload the contents of this project (or at least the `dashboard/` folder plus `README.md`).
3. In GitHub: **Settings → Pages**.
4. Under Build and deployment, choose **Deploy from a branch**.
5. Select `main` and `/dashboard` (or move dashboard files to the repository root and select `/`).
6. Save. GitHub will provide your dashboard URL.

## 5. Use it

- Enter one email and click **Check & Add**.
- A new domain is inserted once.
- If that domain already exists, the dashboard reports **Duplicate** and does not insert it.
- Use **Bulk import** to paste multiple email addresses (one per line). Each line is checked independently.
- Use **Refresh stats** to update the total domain count.
- Export the current sheet contents as CSV from the dashboard.

## Security notes

- The API key is an access gate, not a substitute for restricting access to the dashboard. Anyone with the key can add domains.
- Do not commit the API key or spreadsheet ID to GitHub.
- The Apps Script endpoint is public because GitHub Pages is static hosting. Keep the API key private and rotate it if exposed.
- The sheet stores one row per unique domain. The first email submitted for a domain is retained.
- This is designed for a single-operator workflow. Apps Script locking prevents simultaneous duplicate inserts, but Google Apps Script quotas and Sheets limits still apply.
