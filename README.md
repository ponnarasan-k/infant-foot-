# Operations Hub

One page for the daily running of a small device manufacturer: follow-up of pending work, purchase
orders and approvals, stock, production orders, vendors, NPD projects, statutory dates, expense
allocation and payslips. It works on a desktop browser and on a phone.

The whole hub is a single file, `ops-hub.html`. It runs in two places:

| Where | How it is opened | Records are kept | Sharing and roles | "With Claude" features | Google Drive |
|---|---|---|---|---|---|
| Claude artifact | the published link on claude.ai | in the artifact's shared database | yes, through the page's Share menu | yes | yes |
| This computer | `npm start`, then http://localhost:8080 | in `data/db.json` on this computer | no login, one user | hidden | hidden, upload files instead |

The two copies do not share records. Pick one as the working copy.

## Run it on this computer

Needs [Node.js](https://nodejs.org) 18 or newer.

```
git clone <this repository>
cd operations-hub
npm install        # optional: lets spreadsheet, PDF and PowerPoint features work offline
npm start
```

Open http://localhost:8080. Stop the server with Ctrl+C.

To look around with example records for every department and stage:

```
npm run start:sample
```

Sample records load only into an empty hub, each is marked "sample", and "Remove sample data" on
the Today page clears them.

Settings, as environment variables:

| Variable | Default | Meaning |
|---|---|---|
| `PORT` | `8080` | port to listen on |
| `HOST` | `127.0.0.1` | address to listen on. The default answers this computer only |
| `OPS_HUB_DATA` | `./data` | folder for `db.json` and attached files |

On macOS and Linux put the setting before the command: `PORT=8090 npm start`. On Windows set it
first: `set PORT=8090` then `npm start` (Command Prompt), or `$env:PORT=8090; npm start` (PowerShell).

Back up the hub by copying the `data` folder. It is in `.gitignore` and must stay out of git.

### Using it from a phone or another computer

Starting with `HOST` set to `0.0.0.0` makes the hub reachable from other devices on the same network at
`http://<this computer's address>:8080`. The local hub has **no login**: anyone who can reach that
address can read and change every record. Use it only on a network you trust, or use the Claude
artifact, which has sign-in and roles.

## What is in the hub

- **Today**: overdue, due today, blocked, approvals waiting, compliance in 7 days, stock to
  reorder; department progress for the month; open and overdue items by day.
- **Pending**: tasks with owner, department, priority, status (Not yet started, Open, In progress,
  Pending, Blocked, Done, Dropped), start and due dates, percent complete and what it is waiting on.
- **Meetings**: notes for daily and weekly meetings, turned into follow-up tasks.
- **Purchase**: nine steps from requisition to payment, with PO type, priority, requester, buyer,
  receiver, quote, release, commitment and payment dates, and attached PO documents.
- **Approvals**: purchase, production and payment approvals in one queue, with a decision log.
- **Stock**: item master with mandatory code, category, description, unit, location, reorder level
  and buffer stock; receipts, issues and counts.
- **Production**: production orders from request through release, material issue, in process and
  quality check to completion, including emergency orders.
- **Vendors**: approval status, certificates, re-evaluation dates and the items each one supplied.
- **NPD projects**: project record, gates, requirements with attached documents.
- **Compliance**: statutory and internal due dates. The hub prepares and tracks; a person files.
- **Expenses**: reads a monthly bookkeeping export and allocates each line to a department.
- **Payroll**: reads a salary sheet and produces payslips as one PDF.
- **Monthly status**: per-department view for a month, with a review mark and a CSV download.
- **Weekly report**: copy as text, or download as PowerPoint or Word.
- **Lists and options**: departments, priority levels, PO types and stock categories can be extended.

### Salary sheets stay in the browser

Payroll reads the salary sheet in the browser and builds the payslips there. Nothing from the
sheet is written to the hub's records, in either copy. Do not commit salary sheets; `.gitignore`
blocks spreadsheets, CSV and PDF files outside `test/fixtures` for this reason.

## Files

```
ops-hub.html        the hub: styles, markup and script in one file (published as the Claude artifact)
local/server.js     serves the hub on this computer and keeps its records; no packages needed
local/runtime.js    stands in for the Claude artifact runtime (records, attachments, downloads)
seed/gen.js         sample records for every department and stage
test/hub.js         checks every module at desktop and phone size against a mock runtime
test/local.js       starts the local server and checks saving, restart, attachments and downloads
```

`ops-hub.html` has no `<html>`, `<head>` or `<body>` tags: the Claude artifact host and
`local/server.js` each wrap it in a page. Inside the file the script asks `window.claude.use(name)`
for `db`, `user`, `sample`, `downloads`, `assets` and `mcp`, and hides a feature when its
capability comes back empty. That is the only link between the hub and where it runs.

## Tests

The tests drive a real browser with [Playwright](https://playwright.dev):

```
npm install
npm install --no-save playwright
npx playwright install chromium
npm test
```

`test/hub.js` runs 204 checks and `test/local.js` runs 23. Screenshots and generated files go to
`test/out`.

## Known limits

- The local copy has no login, no roles and no sharing. Two browser tabs stay in step within a few
  seconds; if both change the same record, the later save wins.
- Attachments can be PDF, image, text or CSV files, up to 25 MB each on the local copy.
- Rules for PF, TDS, professional tax and similar are whatever the uploaded sheet contains. The hub
  does not decide them; confirm with your CA.
- `npm audit` reports advisories in `xlsx` 0.18.5 (the last version on npm; a crafted spreadsheet
  can misbehave when read) and in install-time packages pulled in by `pdfjs-dist`, `jspdf` and
  `pptxgenjs` that the hub never runs. Open only spreadsheets and PDFs from sources you trust.
  The PDF reader runs with script evaluation switched off.
