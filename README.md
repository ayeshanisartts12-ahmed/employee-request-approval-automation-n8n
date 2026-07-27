# Employee Request Approval Automation (n8n)

An n8n workflow that automates the employee → manager request approval process (e.g. Work From Home, Leave, etc.), with Google Sheets as the request log and Gmail for notifications.

## 🔄 How it works

1. **Employee submits a request** → a webhook (`Receive New Request`) accepts a POST with:
   - `employee_name`
   - `employee_email`
   - `manager_email`
   - `request_type`
   - `details`
2. The data is mapped into clean fields (`Edit Fields`) and **logged to Google Sheets** with `Status = Pending`.
3. The newly logged row is fetched back (to get its `row_number`).
4. **The manager receives an email** with the request details and two buttons: ✅ Approve / ❌ Reject. Each button is a link to a dedicated webhook, carrying the row number and request info as query params.
5. Clicking a button:
   - Updates that row's `Status` in Google Sheets (`Approved` or `Rejected`)
   - Shows the manager a simple confirmation page
6. A **Google Sheets Trigger** watches for row updates. When `Status` changes:
   - If `Approved` → employee gets an "approved" email
   - If `Rejected` → employee gets a "rejected" email

## 🧩 Nodes used

| Node | Purpose |
|---|---|
| `Receive New Request` (Webhook) | Entry point for new requests |
| `Edit Fields` (Set) | Normalizes incoming data |
| `Log Request as Pending` (Google Sheets) | Appends new row |
| `Fetch Logged Row` (Google Sheets) | Retrieves row number for the new entry |
| `Email to Manager for Decision` (Gmail) | Sends approve/reject email |
| `Manager Clicks Approve` / `Manager Clicks Reject` (Webhook) | Handles button clicks |
| `Mark Status as Approved` / `Mark Status as Rejected` (Google Sheets) | Updates row status |
| `Confirm to Manager (Success Page)` (Respond to Webhook) | Confirmation message |
| `Google Sheets Trigger` | Watches for status changes |
| `If` | Branches on Approved vs Rejected |
| `Email Employee — Approved` / `Email Employee — Rejected` (Gmail) | Notifies employee of outcome |

## ⚙️ Setup requirements

To use this workflow, you'll need:
- An **n8n instance** (cloud or self-hosted)
- A **Google Sheet** with these columns: `Employee_Name`, `Employee_Email`, `Manager_Email`, `Request_type`, `Request_Details`, `Status`, `Date/Time`
- **Google Sheets OAuth2** credentials connected in n8n
- **Gmail OAuth2** credentials connected in n8n
- Update all webhook URLs in the email templates to match **your own n8n instance URL**

> ⚠️ Before importing: open the JSON and replace the placeholder credential IDs, spreadsheet ID, and instance URL with your own. n8n will prompt you to reconnect credentials on import regardless.

## 🐛 Known issues (fix before production use)

- [ ] **Hardcoded recipient email** — the manager/employee email nodes currently send to a fixed address instead of `{{ $json.Manager_Email }}` / `{{ $json.Employee_Email }}`. Update the `sendTo` field in all three Gmail nodes.
- [ ] **Wrong greeting variable** — approval/rejection emails greet the user with `{{ $json.Request_type }}` instead of `{{ $json.Employee_Name }}`.
- [ ] **Broken field reference** — the rejected-employee email uses `{{ $json.Request.Details }}` (typo) instead of `{{ $json.Request_Details }}`.
- [ ] **No authentication on approve/reject links** — anyone with the link can approve/reject a request. Consider adding a signed token or secret query param if this goes beyond internal/demo use.
- [ ] **Sanitize before publishing publicly** — remove/replace real email addresses, spreadsheet IDs, and instance subdomains if this repo is public.

## 📝 Notes

- The workflow assumes each request creates exactly one row and relies on `row_number` to match updates — don't manually reorder/delete rows in the sheet while requests are pending.
- `pinData` in the JSON includes sample/mock data for testing the Google Sheets Trigger node in the n8n editor; it does not affect production runs.

## 📄 License

MIT (or update to your preferred license)
