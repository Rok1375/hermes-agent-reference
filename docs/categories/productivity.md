# Category: productivity

> Deep-dive for the **9 skills** in the `productivity` category. Each section embeds the full `SKILL.md` body so this doc is self-contained.

**Skills in this category:**

- [`airtable`](../skills/productivity/airtable.md) - Airtable REST API via curl. Records CRUD, filters, upserts.
- [`google-workspace`](../skills/productivity/google-workspace.md) - Gmail, Calendar, Drive, Docs, Sheets via gws CLI or Python.
- [`maps`](../skills/productivity/maps.md) - Geocode, POIs, routes, timezones via OpenStreetMap/OSRM.
- [`nano-pdf`](../skills/productivity/nano-pdf.md) - Edit PDF text/typos/titles via nano-pdf CLI (NL prompts).
- [`notion`](../skills/productivity/notion.md) - Notion API + ntn CLI: pages, databases, markdown, Workers.
- [`ocr-and-documents`](../skills/productivity/ocr-and-documents.md) - Extract text from PDFs/scans (pymupdf, marker-pdf).
- [`petdex`](../skills/productivity/petdex.md) - Install and select animated petdex mascots for Hermes.
- [`powerpoint`](../skills/productivity/powerpoint.md) - Create, read, edit .pptx decks, slides, notes, templates.
- [`teams-meeting-pipeline`](../skills/productivity/teams-meeting-pipeline.md) - Operate the Teams meeting summary pipeline via Hermes CLI — summarize meetings, inspect pipeline status, replay jobs, ma

---

## airtable

- **Name:** `airtable`
- **Version:** 1.1.0
- **Author:** community
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/productivity\airtable/SKILL.md`

**Description:**

> Airtable REST API via curl. Records CRUD, filters, upserts.

**Full SKILL.md body:**

```markdown
---
name: airtable
description: Airtable REST API via curl. Records CRUD, filters, upserts.
version: 1.1.0
author: community
license: MIT
platforms: [linux, macos, windows]
prerequisites:
  env_vars: [AIRTABLE_API_KEY]
  commands: [curl]
metadata:
  hermes:
    tags: [Airtable, Productivity, Database, API]
    homepage: https://airtable.com/developers/web/api/introduction
---

# Airtable — Bases, Tables & Records

Work with Airtable's REST API directly via `curl` using the `terminal` tool. No MCP server, no OAuth flow, no Python SDK — just `curl` and a personal access token.

## Prerequisites

1. Create a **Personal Access Token (PAT)** at https://airtable.com/create/tokens (tokens start with `pat...`).
2. Grant these scopes (minimum):
   - `data.records:read` — read rows
   - `data.records:write` — create / update / delete rows
   - `schema.bases:read` — list bases and tables
3. **Important:** in the same token UI, add each base you want to access to the token's **Access** list. PATs are scoped per-base — a valid token on the wrong base returns `403`.
4. Store the token in `${HERMES_HOME:-~/.hermes}/.env` (or via `hermes setup`):
   ```
   AIRTABLE_API_KEY=pat_your_token_here
   ```

> Note: legacy `key...` API keys were deprecated Feb 2024. Only PATs and OAuth tokens work now.

## API Basics

- **Endpoint:** `https://api.airtable.com/v0`
- **Auth header:** `Authorization: Bearer $AIRTABLE_API_KEY`
- **All requests** use JSON (`Content-Type: application/json` for any POST/PATCH/PUT body).
- **Object IDs:** bases `app...`, tables `tbl...`, records `rec...`, fields `fld...`. IDs never change; names can. Prefer IDs in automations.
- **Rate limit:** 5 requests/sec/base. `429` → back off. Burst on a single base will be throttled.

Base curl pattern:
```bash
curl -s "https://api.airtable.com/v0/$BASE_ID/$TABLE?maxRecords=5" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" | python3 -m json.tool
```

`-s` suppresses curl's progress bar — keep it set for every call so the tool output stays clean for Hermes. Pipe through `python3 -m json.tool` (always present) or `jq` (if installed) for readable JSON.

## Field Types (request body shapes)

| Field type | Write shape |
|---|---|
| Single line text | `"Name": "hello"` |
| Long text | `"Notes": "multi\nline"` |
| Number | `"Score": 42` |
| Checkbox | `"Done": true` |
| Single select | `"Status": "Todo"` (name must already exist unless `typecast: true`) |
| Multi-select | `"Tags": ["urgent", "bug"]` |
| Date | `"Due": "2026-04-01"` |
| DateTime (UTC) | `"At": "2026-04-01T14:30:00.000Z"` |
| URL / Email / Phone | `"Link": "https://…"` |
| Attachment | `"Files": [{"url": "https://…"}]` (Airtable fetches + rehosts) |
| Linked record | `"Owner": ["recXXXXXXXXXXXXXX"]` (array of record IDs) |
| User | `"AssignedTo": {"id": "usrXXXXXXXXXXXXXX"}` |

Pass `"typecast": true` at the top level of a create/update body to let Airtable auto-coerce values (e.g. create a new select option on the fly, convert `"42"` → `42`).

## Common Queries

### List bases the token can see
```bash
curl -s "https://api.airtable.com/v0/meta/bases" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" | python3 -m json.tool
```

### List tables + schema for a base
```bash
curl -s "https://api.airtable.com/v0/meta/bases/$BASE_ID/tables" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" | python3 -m json.tool
```
Use this BEFORE mutating — confirms exact field names and IDs, surfaces `options.choices` for select fields, and shows primary-field names.

### List records (first 10)
```bash
curl -s "https://api.airtable.com/v0/$BASE_ID/$TABLE?maxRecords=10" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" | python3 -m json.tool
```

### Get a single record
```bash
curl -s "https://api.airtable.com/v0/$BASE_ID/$TABLE/$RECORD_ID" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" | python3 -m json.tool
```

### Filter records (filterByFormula)
Airtable formulas must be URL-encoded. Let Python stdlib do it — never hand-encode:
```bash
FORMULA="{Status}='Todo'"
ENC=$(python3 -c 'import sys, urllib.parse; print(urllib.parse.quote(sys.argv[1], safe=""))' "$FORMULA")
curl -s "https://api.airtable.com/v0/$BASE_ID/$TABLE?filterByFormula=$ENC&maxRecords=20" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" | python3 -m json.tool
```

Useful formula patterns:
- Exact match: `{Email}='user@example.com'`
- Contains: `FIND('bug', LOWER({Title}))`
- Multiple conditions: `AND({Status}='Todo', {Priority}='High')`
- Or: `OR({Owner}='alice', {Owner}='bob')`
- Not empty: `NOT({Assignee}='')`
- Date comparison: `IS_AFTER({Due}, TODAY())`

### Sort + select specific fields
```bash
curl -s "https://api.airtable.com/v0/$BASE_ID/$TABLE?sort%5B0%5D%5Bfield%5D=Priority&sort%5B0%5D%5Bdirection%5D=asc&fields%5B%5D=Name&fields%5B%5D=Status" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" | python3 -m json.tool
```
Square brackets in query params MUST be URL-encoded (`%5B` / `%5D`).

### Use a named view
```bash
curl -s "https://api.airtable.com/v0/$BASE_ID/$TABLE?view=Grid%20view&maxRecords=50" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" | python3 -m json.tool
```
Views apply their saved filter + sort server-side.

## Common Mutations

### Create a record
```bash
curl -s -X POST "https://api.airtable.com/v0/$BASE_ID/$TABLE" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"fields":{"Name":"New task","Status":"Todo","Priority":"High"}}' | python3 -m json.tool
```

### Create up to 10 records in one call
```bash
curl -s -X POST "https://api.airtable.com/v0/$BASE_ID/$TABLE" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "typecast": true,
    "records": [
      {"fields": {"Name": "Task A", "Status": "Todo"}},
      {"fields": {"Name": "Task B", "Status": "In progress"}}
    ]
  }' | python3 -m json.tool
```
Batch endpoints are capped at **10 records per request**. For larger inserts, loop in batches of 10 with a short sleep to respect 5 req/sec/base.

### Update a record (PATCH — merges, preserves unchanged fields)
```bash
curl -s -X PATCH "https://api.airtable.com/v0/$BASE_ID/$TABLE/$RECORD_ID" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"fields":{"Status":"Done"}}' | python3 -m json.tool
```

### Upsert by a merge field (no ID needed)
```bash
curl -s -X PATCH "https://api.airtable.com/v0/$BASE_ID/$TABLE" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "performUpsert": {"fieldsToMergeOn": ["Email"]},
    "records": [
      {"fields": {"Email": "user@example.com", "Status": "Active"}}
    ]
  }' | python3 -m json.tool
```
`performUpsert` creates records whose merge-field values are new, patches records whose merge-field values already exist. Great for idempotent syncs.

### Delete a record
```bash
curl -s -X DELETE "https://api.airtable.com/v0/$BASE_ID/$TABLE/$RECORD_ID" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" | python3 -m json.tool
```

### Delete up to 10 records in one call
```bash
curl -s -X DELETE "https://api.airtable.com/v0/$BASE_ID/$TABLE?records%5B%5D=rec1&records%5B%5D=rec2" \
  -H "Authorization: Bearer $AIRTABLE_API_KEY" | python3 -m json.tool
```

## Pagination

List endpoints return at most **100 records per page**. If the response includes `"offset": "..."`, pass it back on the next call. Loop until the field is absent:

```bash
OFFSET=""
while :; do
  URL="https://api.airtable.com/v0/$BASE_ID/$TABLE?pageSize=100"
  [ -n "$OFFSET" ] && URL="$URL&offset=$OFFSET"
  RESP=$(curl -s "$URL" -H "Authorization: Bearer $AIRTABLE_API_KEY")
  echo "$RESP" | python3 -c 'import json,sys; d=json.load(sys.stdin); [print(r["id"], r["fields"].get("Name","")) for r in d["records"]]'
  OFFSET=$(echo "$RESP" | python3 -c 'import json,sys; d=json.load(sys.stdin); print(d.get("offset",""))')
  [ -z "$OFFSET" ] && break
done
```

## Typical Hermes Workflow

1. **Confirm auth.** `curl -s -o /dev/null -w "%{http_code}\n" https://api.airtable.com/v0/meta/bases -H "Authorization: Bearer $AIRTABLE_API_KEY"` — expect `200`.
2. **Find the base.** List bases (step above) OR ask the user for the `app...` ID directly if the token lacks `schema.bases:read`.
3. **Inspect the schema.** `GET /v0/meta/bases/$BASE_ID/tables` — cache the exact field names and primary-field name locally in the session before mutating anything.
4. **Read before you write.** For "update X where Y", `filterByFormula` first to resolve the `rec...` ID, then `PATCH /v0/$BASE_ID/$TABLE/$RECORD_ID`. Never guess record IDs.
5. **Batch writes.** Combine related creates into one 10-record POST to stay under the 5 req/sec budget.
6. **Destructive ops.** Deletions can't be undone via API. If the user says "delete all Xs", echo back the filter + record count and confirm before firing.

## Pitfalls

- **`filterByFormula` MUST be URL-encoded.** Field names with spaces or non-ASCII also need encoding (`{My Field}` → `%7BMy%20Field%7D`). Use Python stdlib (pattern above) — never hand-escape.
- **Empty fields are omitted from responses.** A missing `"Assignee"` key doesn't mean the field doesn't exist — it means this record's value is empty. Check the schema (step 3) before concluding a field is missing.
- **PATCH vs PUT.** `PATCH` merges supplied fields into the record. `PUT` replaces the record entirely and clears any field you didn't include. Default to `PATCH`.
- **Single-select options must exist.** Writing `"Status": "Shipping"` when `Shipping` isn't in the field's option list errors with `INVALID_MULTIPLE_CHOICE_OPTIONS` unless you pass `"typecast": true` (which auto-creates the option).
- **Per-base token scoping.** A `403` on one base while another works means the token's Access list doesn't include that base — not a scope or auth issue. Send the user to https://airtable.com/create/tokens to grant it.
- **Rate limits are per base, not per token.** 5 req/sec on `baseA` and 5 req/sec on `baseB` is fine; 6 req/sec on `baseA` alone will throttle. Monitor the `Retry-After` header on `429`.

## Important Notes for Hermes

- **Always use the `terminal` tool with `curl`.** Do NOT use `web_extract` (it can't send auth headers) or `browser_navigate` (needs UI auth and is slow).
- **`AIRTABLE_API_KEY` flows from `${HERMES_HOME:-~/.hermes}/.env` into the subprocess automatically** when this skill is loaded — no need to re-export it before each `curl` call.
- **Escape curly braces in formulas carefully.** In a heredoc body, `{Status}` is literal. In a shell argument, `{Status}` is safe outside `{...}` brace-expansion context — but pass dynamic strings through `python3 urllib.parse.quote` before splicing into a URL.
- **Pretty-print with `python3 -m json.tool`** (always present) rather than `jq` (optional). Only reach for `jq` when you need filtering/projection.
- **Pagination is per-page, not global.** Airtable's 100-record cap is a hard limit; there is no way to bump it. Loop with `offset` until the field is absent.
- **Read the `errors` array** on non-2xx responses — Airtable returns structured error codes like `AUTHENTICATION_REQUIRED`, `INVALID_PERMISSIONS`, `MODEL_ID_NOT_FOUND`, `INVALID_MULTIPLE_CHOICE_OPTIONS` that tell you exactly what's wrong.

```

---

## google-workspace

- **Name:** `google-workspace`
- **Version:** 1.1.0
- **Author:** Nous Research
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/productivity\google-workspace/SKILL.md`

**Description:**

> Gmail, Calendar, Drive, Docs, Sheets via gws CLI or Python.

**Full SKILL.md body:**

```markdown
---
name: google-workspace
description: "Gmail, Calendar, Drive, Docs, Sheets via gws CLI or Python."
version: 1.1.0
author: Nous Research
license: MIT
platforms: [linux, macos, windows]
required_credential_files:
  - path: google_token.json
    description: Google OAuth2 token (created by setup script)
  - path: google_client_secret.json
    description: Google OAuth2 client credentials (downloaded from Google Cloud Console)
metadata:
  hermes:
    tags: [Google, Gmail, Calendar, Drive, Sheets, Docs, Contacts, Email, OAuth]
    homepage: https://github.com/NousResearch/hermes-agent
    related_skills: [himalaya]
---

# Google Workspace

Gmail, Calendar, Drive, Contacts, Sheets, and Docs — through Hermes-managed OAuth and a thin CLI wrapper. When `gws` is installed, the skill uses it as the execution backend for broader Google Workspace coverage; otherwise it falls back to the bundled Python client implementation.

## References

- `references/gmail-search-syntax.md` — Gmail search operators (is:unread, from:, newer_than:, etc.)

## Scripts

- `scripts/setup.py` — OAuth2 setup (run once to authorize)
- `scripts/google_api.py` — compatibility wrapper CLI. It prefers `gws` for operations when available, while preserving Hermes' existing JSON output contract.

## First-Time Setup

The setup is fully non-interactive — you drive it step by step so it works
on CLI, Telegram, Discord, or any platform.

Define a shorthand first:

```bash
GSETUP="python ${HERMES_HOME:-$HOME/.hermes}/skills/productivity/google-workspace/scripts/setup.py"
```

### Step 0: Check if already set up

```bash
$GSETUP --check
```

If it prints `AUTHENTICATED`, skip to Usage — setup is already done.

### Step 1: Triage — ask the user what they need

Before starting OAuth setup, ask the user TWO questions:

**Question 1: "What Google services do you need? Just email, or also
Calendar/Drive/Sheets/Docs?"**

- **Email only** → They don't need this skill at all. Use the `himalaya` skill
  instead — it works with a Gmail App Password (Settings → Security → App
  Passwords) and takes 2 minutes to set up. No Google Cloud project needed.
  Load the himalaya skill and follow its setup instructions.

- **Email + Calendar** → Continue with this skill, but use
  `--services email,calendar` during auth so the consent screen only asks for
  the scopes they actually need.

- **Calendar/Drive/Sheets/Docs only** → Continue with this skill and use a
  narrower `--services` set like `calendar,drive,sheets,docs`.

- **Full Workspace access** → Continue with this skill and use the default
  `all` service set.

**Question 2: "Does your Google account use Advanced Protection (hardware
security keys required to sign in)? If you're not sure, you probably don't
— it's something you would have explicitly enrolled in."**

- **No / Not sure** → Normal setup. Continue below.
- **Yes** → Their Workspace admin must add the OAuth client ID to the org's
  allowed apps list before Step 4 will work. Let them know upfront.

### Step 2: Create OAuth credentials (one-time, ~5 minutes)

Tell the user:

> You need a Google Cloud OAuth client. This is a one-time setup:
>
> 1. Create or select a project:
>    https://console.cloud.google.com/projectselector2/home/dashboard
> 2. Enable the required APIs from the API Library:
>    https://console.cloud.google.com/apis/library
>    Enable: Gmail API, Google Calendar API, Google Drive API,
>    Google Sheets API, Google Docs API, People API
> 3. Create the OAuth client here:
>    https://console.cloud.google.com/apis/credentials
>    Credentials → Create Credentials → OAuth 2.0 Client ID
> 4. Application type: "Desktop app" → Create
> 5. If the app is still in Testing, add the user's Google account as a test user here:
>    https://console.cloud.google.com/auth/audience
>    Audience → Test users → Add users
> 6. Download the JSON file and tell me the file path
>
> Important Hermes CLI note: if the file path starts with `/`, do NOT send only the bare path as its own message in the CLI, because it can be mistaken for a slash command. Send it in a sentence instead, like:
> `The JSON file path is: /home/user/Downloads/client_secret_....json`

Once they provide the path:

```bash
$GSETUP --client-secret /path/to/client_secret.json
```

If they paste the raw client ID / client secret values instead of a file path,
write a valid Desktop OAuth JSON file for them yourself, save it somewhere
explicit (for example `~/Downloads/hermes-google-client-secret.json`), then run
`--client-secret` against that file.

### Step 3: Get authorization URL

Use the service set chosen in Step 1. Examples:

```bash
$GSETUP --auth-url --services email,calendar --format json
$GSETUP --auth-url --services calendar,drive,sheets,docs --format json
$GSETUP --auth-url --services all --format json
```

This returns JSON with an `auth_url` field and also saves the exact URL to
`~/.hermes/google_oauth_last_url.txt`.

Agent rules for this step:
- Extract the `auth_url` field and send that exact URL to the user as a single line.
- Tell the user that the browser will likely fail on `http://localhost:1` after approval, and that this is expected.
- Tell them to copy the ENTIRE redirected URL from the browser address bar.
- If the user gets `Error 403: access_denied`, send them directly to `https://console.cloud.google.com/auth/audience` to add themselves as a test user.

### Step 4: Exchange the code

The user will paste back either a URL like `http://localhost:1/?code=4/0A...&scope=...`
or just the code string. Either works. The `--auth-url` step stores a temporary
pending OAuth session locally so `--auth-code` can complete the PKCE exchange
later, even on headless systems:

```bash
$GSETUP --auth-code "THE_URL_OR_CODE_THE_USER_PASTED" --format json
```

If `--auth-code` fails because the code expired, was already used, or came from
an older browser tab, it now returns a fresh `fresh_auth_url`. In that case,
immediately send the new URL to the user and have them retry with the newest
browser redirect only.

### Step 5: Verify

```bash
$GSETUP --check
```

Should print `AUTHENTICATED`. Setup is complete — token refreshes automatically from now on.

### Notes

- Token is stored at `~/.hermes/google_token.json` and auto-refreshes.
- Pending OAuth session state/verifier are stored temporarily at `~/.hermes/google_oauth_pending.json` until exchange completes.
- If `gws` is installed, `google_api.py` points it at the same `~/.hermes/google_token.json` credentials file. Users do not need to run a separate `gws auth login` flow.
- To revoke: `$GSETUP --revoke`

## Usage

All commands go through the API script. Set `GAPI` as a shorthand:

```bash
GAPI="python ${HERMES_HOME:-$HOME/.hermes}/skills/productivity/google-workspace/scripts/google_api.py"
```

### Gmail

```bash
# Search (returns JSON array with id, from, subject, date, snippet)
$GAPI gmail search "is:unread" --max 10
$GAPI gmail search "from:boss@company.com newer_than:1d"
$GAPI gmail search "has:attachment filename:pdf newer_than:7d"

# Read full message (returns JSON with body text)
$GAPI gmail get MESSAGE_ID

# Send
$GAPI gmail send --to user@example.com --subject "Hello" --body "Message text"
$GAPI gmail send --to user@example.com --subject "Report" --body "<h1>Q4</h1><p>Details...</p>" --html
$GAPI gmail send --to user@example.com --subject "Hello" --from '"Research Agent" <user@example.com>' --body "Message text"

# Reply (automatically threads and sets In-Reply-To)
$GAPI gmail reply MESSAGE_ID --body "Thanks, that works for me."
$GAPI gmail reply MESSAGE_ID --from '"Support Bot" <user@example.com>' --body "Thanks"

# Labels
$GAPI gmail labels
$GAPI gmail modify MESSAGE_ID --add-labels LABEL_ID
$GAPI gmail modify MESSAGE_ID --remove-labels UNREAD
```

### Calendar

```bash
# List events (defaults to next 7 days)
$GAPI calendar list
$GAPI calendar list --start 2026-03-01T00:00:00Z --end 2026-03-07T23:59:59Z

# Create event (ISO 8601 with timezone required)
$GAPI calendar create --summary "Team Standup" --start 2026-03-01T10:00:00-06:00 --end 2026-03-01T10:30:00-06:00
$GAPI calendar create --summary "Lunch" --start 2026-03-01T12:00:00Z --end 2026-03-01T13:00:00Z --location "Cafe"
$GAPI calendar create --summary "Review" --start 2026-03-01T14:00:00Z --end 2026-03-01T15:00:00Z --attendees "alice@co.com,bob@co.com"

# Delete event
$GAPI calendar delete EVENT_ID
```

### Drive

```bash
# Search existing files
$GAPI drive search "quarterly report" --max 10
$GAPI drive search "mimeType='application/pdf'" --raw-query --max 5

# Get metadata for a single file
$GAPI drive get FILE_ID

# Upload a local file (auto-detects MIME type)
$GAPI drive upload /path/to/report.pdf
$GAPI drive upload /path/to/image.png --name "Logo.png" --parent FOLDER_ID

# Download (binary files download as-is; Google-native files export to a
# sensible default — Docs→pdf, Sheets→csv, Slides→pdf, Drawings→png)
$GAPI drive download FILE_ID
$GAPI drive download DOC_ID --output ~/doc.pdf
$GAPI drive download DOC_ID --export-mime text/plain --output ~/doc.txt

# Create a folder
$GAPI drive create-folder "Reports"
$GAPI drive create-folder "Q4" --parent FOLDER_ID

# Share
$GAPI drive share FILE_ID --email alice@example.com --role reader
$GAPI drive share FILE_ID --email alice@example.com --role writer --notify
$GAPI drive share FILE_ID --type anyone --role reader        # anyone with link
$GAPI drive share FILE_ID --type domain --domain example.com --role reader

# Delete — defaults to trash (reversible). Use --permanent to skip the trash.
$GAPI drive delete FILE_ID
$GAPI drive delete FILE_ID --permanent
```

### Contacts

```bash
$GAPI contacts list --max 20
```

### Sheets

```bash
# Create a new spreadsheet
$GAPI sheets create --title "Q4 Budget"
$GAPI sheets create --title "Inventory" --sheet-name "Stock"

# Read
$GAPI sheets get SHEET_ID "Sheet1!A1:D10"

# Write
$GAPI sheets update SHEET_ID "Sheet1!A1:B2" --values '[["Name","Score"],["Alice","95"]]'

# Append rows
$GAPI sheets append SHEET_ID "Sheet1!A:C" --values '[["new","row","data"]]'
```

### Docs

```bash
# Read
$GAPI docs get DOC_ID

# Create a new Doc (optionally seeded with body text)
$GAPI docs create --title "Meeting Notes"
$GAPI docs create --title "Draft" --body "First paragraph..."

# Append text to the end of an existing Doc
$GAPI docs append DOC_ID --text "Additional content to append"
```

## Output Format

All commands return JSON. Parse with `jq` or read directly. Key fields:

- **Gmail search**: `[{id, threadId, from, to, subject, date, snippet, labels}]`
- **Gmail get**: `{id, threadId, from, to, subject, date, labels, body}`
- **Gmail send/reply**: `{status: "sent", id, threadId}`
- **Calendar list**: `[{id, summary, start, end, location, description, htmlLink}]`
- **Calendar create**: `{status: "created", id, summary, htmlLink}`
- **Drive search**: `[{id, name, mimeType, modifiedTime, webViewLink}]`
- **Drive get**: `{id, name, mimeType, modifiedTime, size, webViewLink, parents, owners}`
- **Drive upload**: `{status: "uploaded", id, name, mimeType, webViewLink}`
- **Drive download**: `{status: "downloaded", id, name, path, mimeType}`
- **Drive create-folder**: `{status: "created", id, name, webViewLink}`
- **Drive share**: `{status: "shared", permissionId, fileId, role, type}`
- **Drive delete**: `{status: "trashed" | "deleted", fileId, permanent}`
- **Contacts list**: `[{name, emails: [...], phones: [...]}]`
- **Sheets get**: `[[cell, cell, ...], ...]`
- **Sheets create**: `{status: "created", spreadsheetId, title, spreadsheetUrl}`
- **Docs create**: `{status: "created", documentId, title, url}`
- **Docs append**: `{status: "appended", documentId, inserted_at, characters}`

## Rules

1. **Never send email, create/delete calendar events, delete Drive files, share files, or modify Docs/Sheets without confirming with the user first.** Show what will be done (recipients, file IDs, content, share role) and ask for approval. For `drive delete`, prefer the default trash (reversible) over `--permanent`.
2. **Check auth before first use** — run `setup.py --check`. If it fails, guide the user through setup.
3. **Use the Gmail search syntax reference** for complex queries — load it with `skill_view("google-workspace", file_path="references/gmail-search-syntax.md")`.
4. **Calendar times must include timezone** — always use ISO 8601 with offset (e.g., `2026-03-01T10:00:00-06:00`) or UTC (`Z`).
5. **Respect rate limits** — avoid rapid-fire sequential API calls. Batch reads when possible.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `NOT_AUTHENTICATED` | Run setup Steps 2-5 above |
| `REFRESH_FAILED` | Token revoked or expired — redo Steps 3-5 |
| `HttpError 403: Insufficient Permission` | Missing API scope — `$GSETUP --revoke` then redo Steps 3-5 |
| `AUTHENTICATED (partial)` or "Token missing scopes" | New write capabilities (Drive write/delete, Docs create/edit) require re-authorization. `$GSETUP --revoke` then redo Steps 3-5 to grant the upgraded scopes. |
| `HttpError 403: Access Not Configured` | API not enabled — user needs to enable it in Google Cloud Console |
| `ModuleNotFoundError` | Run `$GSETUP --install-deps` |
| Advanced Protection blocks auth | Workspace admin must allowlist the OAuth client ID |

## Revoking Access

```bash
$GSETUP --revoke
```

```

---

## maps

- **Name:** `maps`
- **Version:** 1.2.0
- **Author:** Mibayy
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/productivity\maps/SKILL.md`

**Description:**

> Geocode, POIs, routes, timezones via OpenStreetMap/OSRM.

**Full SKILL.md body:**

```markdown
---
name: maps
description: "Geocode, POIs, routes, timezones via OpenStreetMap/OSRM."
version: 1.2.0
author: Mibayy
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [maps, geocoding, places, routing, distance, directions, nearby, location, openstreetmap, nominatim, overpass, osrm]
    category: productivity
    requires_toolsets: [terminal]
    supersedes: [find-nearby]
---

# Maps Skill

Location intelligence using free, open data sources. 8 commands, 44 POI
categories, zero dependencies (Python stdlib only), no API key required.

Data sources: OpenStreetMap/Nominatim, Overpass API, OSRM, TimeAPI.io.

This skill supersedes the old `find-nearby` skill — all of find-nearby's
functionality is covered by the `nearby` command below, with the same
`--near "<place>"` shortcut and multi-category support.

## When to Use

- User sends a Telegram location pin (latitude/longitude in the message) → `nearby`
- User wants coordinates for a place name → `search`
- User has coordinates and wants the address → `reverse`
- User asks for nearby restaurants, hospitals, pharmacies, hotels, etc. → `nearby`
- User wants driving/walking/cycling distance or travel time → `distance`
- User wants turn-by-turn directions between two places → `directions`
- User wants timezone information for a location → `timezone`
- User wants to search for POIs within a geographic area → `area` + `bbox`

## Prerequisites

Python 3.8+ (stdlib only — no pip installs needed).

Script path: `~/.hermes/skills/maps/scripts/maps_client.py`

## Commands

```bash
MAPS=~/.hermes/skills/maps/scripts/maps_client.py
```

### search — Geocode a place name

```bash
python3 $MAPS search "Eiffel Tower"
python3 $MAPS search "1600 Pennsylvania Ave, Washington DC"
```

Returns: lat, lon, display name, type, bounding box, importance score.

### reverse — Coordinates to address

```bash
python3 $MAPS reverse 48.8584 2.2945
```

Returns: full address breakdown (street, city, state, country, postcode).

### nearby — Find places by category

```bash
# By coordinates (from a Telegram location pin, for example)
python3 $MAPS nearby 48.8584 2.2945 restaurant --limit 10
python3 $MAPS nearby 40.7128 -74.0060 hospital --radius 2000

# By address / city / zip / landmark — --near auto-geocodes
python3 $MAPS nearby --near "Times Square, New York" --category cafe
python3 $MAPS nearby --near "90210" --category pharmacy

# Multiple categories merged into one query
python3 $MAPS nearby --near "downtown austin" --category restaurant --category bar --limit 10
```

46 categories: restaurant, cafe, bar, hospital, pharmacy, hotel, guest_house,
camp_site, supermarket, atm, gas_station, parking, museum, park, school,
university, bank, police, fire_station, library, airport, train_station,
bus_stop, church, mosque, synagogue, dentist, doctor, cinema, theatre, gym,
swimming_pool, post_office, convenience_store, bakery, bookshop, laundry,
car_wash, car_rental, bicycle_rental, taxi, veterinary, zoo, playground,
stadium, nightclub.

Each result includes: `name`, `address`, `lat`/`lon`, `distance_m`,
`maps_url` (clickable Google Maps link), `directions_url` (Google Maps
directions from the search point), and promoted tags when available —
`cuisine`, `hours` (opening_hours), `phone`, `website`.

### distance — Travel distance and time

```bash
python3 $MAPS distance "Paris" --to "Lyon"
python3 $MAPS distance "New York" --to "Boston" --mode driving
python3 $MAPS distance "Big Ben" --to "Tower Bridge" --mode walking
```

Modes: driving (default), walking, cycling. Returns road distance, duration,
and straight-line distance for comparison.

### directions — Turn-by-turn navigation

```bash
python3 $MAPS directions "Eiffel Tower" --to "Louvre Museum" --mode walking
python3 $MAPS directions "JFK Airport" --to "Times Square" --mode driving
```

Returns numbered steps with instruction, distance, duration, road name, and
maneuver type (turn, depart, arrive, etc.).

### timezone — Timezone for coordinates

```bash
python3 $MAPS timezone 48.8584 2.2945
python3 $MAPS timezone 35.6762 139.6503
```

Returns timezone name, UTC offset, and current local time.

### area — Bounding box and area for a place

```bash
python3 $MAPS area "Manhattan, New York"
python3 $MAPS area "London"
```

Returns bounding box coordinates, width/height in km, and approximate area.
Useful as input for the bbox command.

### bbox — Search within a bounding box

```bash
python3 $MAPS bbox 40.75 -74.00 40.77 -73.98 restaurant --limit 20
```

Finds POIs within a geographic rectangle. Use `area` first to get the
bounding box coordinates for a named place.

## Working With Telegram Location Pins

When a user sends a location pin, the message contains `latitude:` and
`longitude:` fields. Extract those and pass them straight to `nearby`:

```bash
# User sent a pin at 36.17, -115.14 and asked "find cafes nearby"
python3 $MAPS nearby 36.17 -115.14 cafe --radius 1500
```

Present results as a numbered list with names, distances, and the
`maps_url` field so the user gets a tap-to-open link in chat. For "open
now?" questions, check the `hours` field; if missing or unclear, verify
with `web_search` since OSM hours are community-maintained and not always
current.

## Workflow Examples

**"Find Italian restaurants near the Colosseum":**
1. `nearby --near "Colosseum Rome" --category restaurant --radius 500`
   — one command, auto-geocoded

**"What's near this location pin they sent?":**
1. Extract lat/lon from the Telegram message
2. `nearby LAT LON cafe --radius 1500`

**"How do I walk from hotel to conference center?":**
1. `directions "Hotel Name" --to "Conference Center" --mode walking`

**"What restaurants are in downtown Seattle?":**
1. `area "Downtown Seattle"` → get bounding box
2. `bbox S W N E restaurant --limit 30`

## Pitfalls

- Nominatim ToS: max 1 req/s (handled automatically by the script)
- `nearby` requires lat/lon OR `--near "<address>"` — one of the two is needed
- OSRM routing coverage is best for Europe and North America
- Overpass API can be slow during peak hours; the script automatically
  falls back between mirrors (overpass-api.de → overpass.kumi.systems)
- `distance` and `directions` use `--to` flag for the destination (not positional)
- If a zip code alone gives ambiguous results globally, include country/state

## Verification

```bash
python3 ~/.hermes/skills/maps/scripts/maps_client.py search "Statue of Liberty"
# Should return lat ~40.689, lon ~-74.044

python3 ~/.hermes/skills/maps/scripts/maps_client.py nearby --near "Times Square" --category restaurant --limit 3
# Should return a list of restaurants within ~500m of Times Square
```

```

---

## nano-pdf

- **Name:** `nano-pdf`
- **Version:** 1.0.0
- **Author:** community
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/productivity\nano-pdf/SKILL.md`

**Description:**

> Edit PDF text/typos/titles via nano-pdf CLI (NL prompts).

**Full SKILL.md body:**

```markdown
---
name: nano-pdf
description: "Edit PDF text/typos/titles via nano-pdf CLI (NL prompts)."
version: 1.0.0
author: community
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [PDF, Documents, Editing, NLP, Productivity]
    homepage: https://pypi.org/project/nano-pdf/
---

# nano-pdf

Edit PDFs using natural-language instructions. Point it at a page and describe what to change.

## Prerequisites

```bash
# Install with uv (recommended — already available in Hermes)
uv pip install nano-pdf

# Or with pip
pip install nano-pdf
```

## Usage

```bash
nano-pdf edit <file.pdf> <page_number> "<instruction>"
```

## Examples

```bash
# Change a title on page 1
nano-pdf edit deck.pdf 1 "Change the title to 'Q3 Results' and fix the typo in the subtitle"

# Update a date on a specific page
nano-pdf edit report.pdf 3 "Update the date from January to February 2026"

# Fix content
nano-pdf edit contract.pdf 2 "Change the client name from 'Acme Corp' to 'Acme Industries'"
```

## Notes

- Page numbers may be 0-based or 1-based depending on version — if the edit hits the wrong page, retry with ±1
- Always verify the output PDF after editing (use `read_file` to check file size, or open it)
- The tool uses an LLM under the hood — requires an API key (check `nano-pdf --help` for config)
- Works well for text changes; complex layout modifications may need a different approach

```

---

## notion

- **Name:** `notion`
- **Version:** 2.0.0
- **Author:** community
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/productivity\notion/SKILL.md`

**Description:**

> Notion API + ntn CLI: pages, databases, markdown, Workers.

**Full SKILL.md body:**

```markdown
---
name: notion
description: "Notion API + ntn CLI: pages, databases, markdown, Workers."
version: 2.0.0
author: community
license: MIT
platforms: [linux, macos, windows]
prerequisites:
  env_vars: [NOTION_API_KEY]
metadata:
  hermes:
    tags: [Notion, Productivity, Notes, Database, API, CLI, Workers]
    homepage: https://developers.notion.com
---

# Notion

Talk to Notion two ways. Same integration token works for both — pick by what's available.

◆ **`ntn` CLI** — Notion's official CLI. Shorter syntax, one-line file uploads, required for Workers. macOS + Linux only as of May 2026 (Windows support "coming soon"). **Default when installed.**
◆ **HTTP + curl** — works everywhere including Windows. **Default fallback** when `ntn` isn't installed.

## Setup

### 1. Get an integration token (required for both paths)

1. Create an integration at https://notion.so/my-integrations
2. Copy the API key (starts with `ntn_` or `secret_`)
3. Store in `${HERMES_HOME:-~/.hermes}/.env`:
   ```
   NOTION_API_KEY=ntn_your_key_here
   ```
4. **Share target pages/databases with the integration** in Notion: page menu `...` → `Connect to` → your integration name. Without this, the API returns 404 for that page even though it exists.

### 2. Install `ntn` (preferred path on macOS / Linux)

```bash
# Recommended
curl -fsSL https://ntn.dev | bash

# Or via npm (needs Node 22+, npm 10+)
npm install --global ntn

ntn --version    # verify
```

**Skip `ntn login` — use the integration token instead.** This works headlessly, no browser needed:
```bash
export NOTION_API_TOKEN=$NOTION_API_KEY      # ntn reads NOTION_API_TOKEN
export NOTION_KEYRING=0                       # don't try to use the OS keychain
```

Add those exports to your shell profile (or to `${HERMES_HOME:-~/.hermes}/.env`) so every session inherits them.

### 3. Choose path at runtime

```bash
if command -v ntn >/dev/null 2>&1; then
  # use ntn
else
  # fall back to curl
fi
```

Windows users: skip step 2 entirely until native `ntn` ships — Path B works fine. If you want CLI ergonomics now, install `ntn` inside WSL2.

## API Basics

`Notion-Version: 2025-09-03` is required on all HTTP requests. `ntn` handles this for you. In this version, what users call "databases" are called **data sources** in the API.

## Path A — `ntn` CLI (preferred, macOS / Linux)

### Raw API calls (shorthand for curl)
```bash
ntn api v1/users                                  # GET
ntn api v1/pages parent[page_id]=abc123 \         # POST with inline body
  properties[title][0][text][content]="Notes"
ntn api v1/pages/abc123 -X PATCH archived:=true   # PATCH; := is non-string (bool/num/null)
```

Syntax notes:
- `key=value` — string fields
- `key[nested]=value` — nested object fields
- `key:=value` — typed assignment (booleans, numbers, null, arrays)

### Search
```bash
ntn api v1/search query="page title"
```

### Read page metadata
```bash
ntn api v1/pages/{page_id}
```

### Read page as Markdown (agent-friendly)
```bash
ntn api v1/pages/{page_id}/markdown
```

### Read page content as blocks
```bash
ntn api v1/blocks/{page_id}/children
```

### Create page from Markdown
```bash
ntn api v1/pages \
  parent[page_id]=xxx \
  properties[title][0][text][content]="Notes from meeting" \
  markdown="# Agenda

- Q3 roadmap
- Hiring"
```

### Patch a page with Markdown
```bash
ntn api v1/pages/{page_id}/markdown -X PATCH \
  markdown="## Update

Shipped the prototype."
```

### Query a database (data source)
```bash
ntn api v1/data_sources/{data_source_id}/query -X POST \
  filter[property]=Status filter[select][equals]=Active
```

For complex queries with `sorts`, multiple filter clauses, or compound logic, pipe JSON in:
```bash
echo '{"filter": {"property": "Status", "select": {"equals": "Active"}}, "sorts": [{"property": "Date", "direction": "descending"}]}' | \
  ntn api v1/data_sources/{data_source_id}/query -X POST --json -
```

### File uploads (one-liner — biggest CLI win)
```bash
ntn files create < photo.png
ntn files create --external-url https://example.com/photo.png
ntn files list
```

Compare to the 3-step HTTP flow (create upload → PUT bytes → reference).

### Useful env vars
| Var | Effect |
|---|---|
| `NOTION_API_TOKEN` | Auth token (overrides keychain) — set this to your integration token |
| `NOTION_KEYRING=0` | File-based creds at `~/.config/notion/auth.json` instead of OS keychain |
| `NOTION_WORKSPACE_ID` | Skip the workspace picker prompt |

## Path B — HTTP + curl (cross-platform, default on Windows)

All requests share this pattern:

```bash
curl -s -X GET "https://api.notion.com/v1/..." \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json"
```

On Windows the `curl` shipped with Windows 10+ works as-is. PowerShell users can also use `Invoke-RestMethod`.

### Search
```bash
curl -s -X POST "https://api.notion.com/v1/search" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"query": "page title"}'
```

### Read page metadata
```bash
curl -s "https://api.notion.com/v1/pages/{page_id}" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03"
```

### Read page as Markdown (agent-friendly)

Easier to feed to a model than block JSON.

```bash
curl -s "https://api.notion.com/v1/pages/{page_id}/markdown" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03"
```

### Read page content as blocks (when you need structure)
```bash
curl -s "https://api.notion.com/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03"
```

### Create page from Markdown

`POST /v1/pages` accepts a `markdown` body param.

```bash
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"page_id": "xxx"},
    "properties": {"title": [{"text": {"content": "Notes from meeting"}}]},
    "markdown": "# Agenda\n\n- Q3 roadmap\n- Hiring\n\n## Decisions\n- Ship MVP Friday"
  }'
```

### Patch a page with Markdown
```bash
curl -s -X PATCH "https://api.notion.com/v1/pages/{page_id}/markdown" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"markdown": "## Update\n\nShipped the prototype."}'
```

### Create page in a database (typed properties)
```bash
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"database_id": "xxx"},
    "properties": {
      "Name": {"title": [{"text": {"content": "New Item"}}]},
      "Status": {"select": {"name": "Todo"}}
    }
  }'
```

### Query a database (data source)
```bash
curl -s -X POST "https://api.notion.com/v1/data_sources/{data_source_id}/query" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {"property": "Status", "select": {"equals": "Active"}},
    "sorts": [{"property": "Date", "direction": "descending"}]
  }'
```

### Create a database
```bash
curl -s -X POST "https://api.notion.com/v1/data_sources" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": {"page_id": "xxx"},
    "title": [{"text": {"content": "My Database"}}],
    "properties": {
      "Name": {"title": {}},
      "Status": {"select": {"options": [{"name": "Todo"}, {"name": "Done"}]}},
      "Date": {"date": {}}
    }
  }'
```

### Update page properties
```bash
curl -s -X PATCH "https://api.notion.com/v1/pages/{page_id}" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"properties": {"Status": {"select": {"name": "Done"}}}}'
```

### Append blocks to a page
```bash
curl -s -X PATCH "https://api.notion.com/v1/blocks/{page_id}/children" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{
    "children": [
      {"object": "block", "type": "paragraph", "paragraph": {"rich_text": [{"text": {"content": "Hello from Hermes!"}}]}}
    ]
  }'
```

### File uploads (3-step flow)
```bash
# 1. Create upload
curl -s -X POST "https://api.notion.com/v1/file_uploads" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"filename": "photo.png", "content_type": "image/png"}'

# 2. PUT bytes to the upload_url returned above
curl -s -X PUT "{upload_url}" --data-binary @photo.png

# 3. Reference {file_upload_id} in a page/block payload
```

## Property Types

Common property formats for database items:

- **Title:** `{"title": [{"text": {"content": "..."}}]}`
- **Rich text:** `{"rich_text": [{"text": {"content": "..."}}]}`
- **Select:** `{"select": {"name": "Option"}}`
- **Multi-select:** `{"multi_select": [{"name": "A"}, {"name": "B"}]}`
- **Date:** `{"date": {"start": "2026-01-15", "end": "2026-01-16"}}`
- **Checkbox:** `{"checkbox": true}`
- **Number:** `{"number": 42}`
- **URL:** `{"url": "https://..."}`
- **Email:** `{"email": "user@example.com"}`
- **Relation:** `{"relation": [{"id": "page_id"}]}`

## API Version 2025-09-03 — Databases vs Data Sources

- **Databases became data sources.** Use `/data_sources/` endpoints for queries and retrieval.
- **Two IDs per database:** `database_id` and `data_source_id`.
  - `database_id` when creating pages: `parent: {"database_id": "..."}`
  - `data_source_id` when querying: `POST /v1/data_sources/{id}/query`
- Search returns databases as `"object": "data_source"` with the `data_source_id` field.

## Notion Workers (advanced, requires `ntn`)

Workers are TypeScript programs Notion hosts for you. One worker can expose any combination of:
- **Syncs** — pull data from external APIs into a Notion database on a schedule (default 30 min).
- **Tools** — appear as callable tools inside Notion's Custom Agents.
- **Webhooks** — receive HTTP events from external services (GitHub, Stripe, etc.) and act in Notion.

**Plan / platform gating:**
- CLI works on all plans. **Deploying Workers requires Business or Enterprise.**
- `ntn` is macOS/Linux only as of May 2026. Windows users need WSL2 or to wait for native support.
- Free through August 11, 2026; metered on Notion credits after.

### Minimal Worker

```bash
ntn workers new my-worker      # scaffold
cd my-worker
# Edit src/index.ts
ntn workers deploy --name my-worker
```

`src/index.ts`:
```typescript
import { Worker } from "@notionhq/workers";

const worker = new Worker();
export default worker;

worker.tool("greet", {
  title: "Greet a User",
  description: "Returns a friendly greeting",
  inputSchema: { type: "object", properties: { name: { type: "string" } }, required: ["name"] },
  execute: async ({ name }) => `Hello, ${name}!`,
});
```

### Webhook capability

```typescript
worker.webhook("onGithubPush", {
  title: "GitHub Push Handler",
  execute: async (events, { notion }) => {
    for (const event of events) {
      // event.body, event.rawBody (for signature verification), event.headers
      console.log("got delivery", event.deliveryId);
    }
  },
});
```

After deploy: `ntn workers webhooks list` shows the URL Notion generates. Treat that URL as a secret — anyone with it can POST events unless you add signature verification.

### Worker lifecycle commands

```bash
ntn workers deploy
ntn workers list
ntn workers exec <capability-key> -d '{"name": "world"}'
ntn workers sync trigger <key>            # run a sync now
ntn workers sync pause <key>
ntn workers env set GITHUB_WEBHOOK_SECRET=...
ntn workers runs list                     # recent invocations
ntn workers runs logs <run-id>
ntn workers webhooks list
```

When asked to build a Worker, scaffold with `ntn workers new`, write the code in `src/index.ts`, set any secrets with `ntn workers env set`, and deploy. Notion's docs at https://developers.notion.com/workers cover the full API surface.

## Notion-Flavored Markdown (used by `/markdown` endpoints)

Standard CommonMark plus XML-like tags for Notion-specific blocks. Use **tabs** for indentation.

**Blocks beyond CommonMark:**
```
<callout icon="🎯" color="blue_bg">
	Ship the MVP by **Friday**.
</callout>

<details color="gray">
<summary>Toggle title</summary>
	Children indented one tab
</details>

<columns>
	<column>Left side</column>
	<column>Right side</column>
</columns>

<table_of_contents color="gray"/>
```

**Inline:**
- Mentions: `<mention-user url="..."/>`, `<mention-page url="...">Title</mention-page>`, `<mention-date start="2026-05-15"/>`
- Underline: `<span underline="true">text</span>`
- Color: `<span color="blue">text</span>` or block-level `{color="blue"}` on the first line
- Math: inline `$x^2$`, block `$$ ... $$`
- Citations: `[^https://example.com]`

**Colors:** `gray brown orange yellow green blue purple pink red`, plus `*_bg` variants for backgrounds.

Headings 5/6 collapse to H4. Multiple `>` lines render as separate quote blocks — use `<br>` inside a single `>` for multi-line quotes.

## Choosing the Right Path

| Task | mac / Linux | Windows |
|---|---|---|
| Read/write pages, search, query databases | `ntn api ...` | curl |
| Read a page for an agent to summarize | `ntn api v1/pages/{id}/markdown` | curl `/markdown` endpoint |
| Upload a file | `ntn files create < file` | 3-step HTTP flow |
| One-off API exploration | `ntn api ...` | curl |
| Build a sync / webhook / agent tool hosted by Notion | `ntn workers ...` | WSL2 + `ntn workers ...` |

## Notes

- Page/database IDs are UUIDs (with or without dashes — both accepted).
- Rate limit: ~3 requests/second average. The CLI doesn't bypass this.
- The API cannot set database **view** filters — that's UI-only.
- Use `"is_inline": true` when creating data sources to embed them in a page.
- Always pass `-s` to curl to suppress progress bars (cleaner agent output).
- Pipe JSON through `jq` when reading: `... | jq '.results[0].properties'`.
- Notion also ships an MCP server now (`Notion MCP`, ~91% more token-efficient on DB ops than the previous version) — wire it via Hermes' MCP support if you want streaming Notion access from inside a session, but the paths above are enough for most one-shot tasks.

```

---

## ocr-and-documents

- **Name:** `ocr-and-documents`
- **Version:** 2.3.0
- **Author:** Hermes Agent
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/productivity\ocr-and-documents/SKILL.md`

**Description:**

> Extract text from PDFs/scans (pymupdf, marker-pdf).

**Full SKILL.md body:**

```markdown
---
name: ocr-and-documents
description: "Extract text from PDFs/scans (pymupdf, marker-pdf)."
version: 2.3.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [PDF, Documents, Research, Arxiv, Text-Extraction, OCR]
    related_skills: [powerpoint]
---

# PDF & Document Extraction

For DOCX: use `python-docx` (parses actual document structure, far better than OCR).
For PPTX: see the `powerpoint` skill (uses `python-pptx` with full slide/notes support).
This skill covers **PDFs and scanned documents**.

## Step 1: Remote URL Available?

If the document has a URL, **always try `web_extract` first**:

```
web_extract(urls=["https://arxiv.org/pdf/2402.03300"])
web_extract(urls=["https://example.com/report.pdf"])
```

This handles PDF-to-markdown conversion via Firecrawl with no local dependencies.

Only use local extraction when: the file is local, web_extract fails, or you need batch processing.

## Step 2: Choose Local Extractor

| Feature | pymupdf (~25MB) | marker-pdf (~3-5GB) |
|---------|-----------------|---------------------|
| **Text-based PDF** | ✅ | ✅ |
| **Scanned PDF (OCR)** | ❌ | ✅ (90+ languages) |
| **Tables** | ✅ (basic) | ✅ (high accuracy) |
| **Equations / LaTeX** | ❌ | ✅ |
| **Code blocks** | ❌ | ✅ |
| **Forms** | ❌ | ✅ |
| **Headers/footers removal** | ❌ | ✅ |
| **Reading order detection** | ❌ | ✅ |
| **Images extraction** | ✅ (embedded) | ✅ (with context) |
| **Images → text (OCR)** | ❌ | ✅ |
| **EPUB** | ✅ | ✅ |
| **Markdown output** | ✅ (via pymupdf4llm) | ✅ (native, higher quality) |
| **Install size** | ~25MB | ~3-5GB (PyTorch + models) |
| **Speed** | Instant | ~1-14s/page (CPU), ~0.2s/page (GPU) |

**Decision**: Use pymupdf unless you need OCR, equations, forms, or complex layout analysis.

If the user needs marker capabilities but the system lacks ~5GB free disk:
> "This document needs OCR/advanced extraction (marker-pdf), which requires ~5GB for PyTorch and models. Your system has [X]GB free. Options: free up space, provide a URL so I can use web_extract, or I can try pymupdf which works for text-based PDFs but not scanned documents or equations."

---

## pymupdf (lightweight)

```bash
pip install pymupdf pymupdf4llm
```

**Via helper script**:
```bash
python scripts/extract_pymupdf.py document.pdf              # Plain text
python scripts/extract_pymupdf.py document.pdf --markdown    # Markdown
python scripts/extract_pymupdf.py document.pdf --tables      # Tables
python scripts/extract_pymupdf.py document.pdf --images out/ # Extract images
python scripts/extract_pymupdf.py document.pdf --metadata    # Title, author, pages
python scripts/extract_pymupdf.py document.pdf --pages 0-4   # Specific pages
```

**Inline**:
```bash
python3 -c "
import pymupdf
doc = pymupdf.open('document.pdf')
for page in doc:
    print(page.get_text())
"
```

---

## marker-pdf (high-quality OCR)

```bash
# Check disk space first
python scripts/extract_marker.py --check

pip install marker-pdf
```

**Via helper script**:
```bash
python scripts/extract_marker.py document.pdf                # Markdown
python scripts/extract_marker.py document.pdf --json         # JSON with metadata
python scripts/extract_marker.py document.pdf --output_dir out/  # Save images
python scripts/extract_marker.py scanned.pdf                 # Scanned PDF (OCR)
python scripts/extract_marker.py document.pdf --use_llm      # LLM-boosted accuracy
```

**CLI** (installed with marker-pdf):
```bash
marker_single document.pdf --output_dir ./output
marker /path/to/folder --workers 4    # Batch
```

---

## Arxiv Papers

```
# Abstract only (fast)
web_extract(urls=["https://arxiv.org/abs/2402.03300"])

# Full paper
web_extract(urls=["https://arxiv.org/pdf/2402.03300"])

# Search
web_search(query="arxiv GRPO reinforcement learning 2026")
```

## Split, Merge & Search

pymupdf handles these natively — use `execute_code` or inline Python:

```python
# Split: extract pages 1-5 to a new PDF
import pymupdf
doc = pymupdf.open("report.pdf")
new = pymupdf.open()
for i in range(5):
    new.insert_pdf(doc, from_page=i, to_page=i)
new.save("pages_1-5.pdf")
```

```python
# Merge multiple PDFs
import pymupdf
result = pymupdf.open()
for path in ["a.pdf", "b.pdf", "c.pdf"]:
    result.insert_pdf(pymupdf.open(path))
result.save("merged.pdf")
```

```python
# Search for text across all pages
import pymupdf
doc = pymupdf.open("report.pdf")
for i, page in enumerate(doc):
    results = page.search_for("revenue")
    if results:
        print(f"Page {i+1}: {len(results)} match(es)")
        print(page.get_text("text"))
```

No extra dependencies needed — pymupdf covers split, merge, search, and text extraction in one package.

---

## Notes

- `web_extract` is always first choice for URLs
- pymupdf is the safe default — instant, no models, works everywhere
- marker-pdf is for OCR, scanned docs, equations, complex layouts — install only when needed
- Both helper scripts accept `--help` for full usage
- marker-pdf downloads ~2.5GB of models to `~/.cache/huggingface/` on first use
- For Word docs: `pip install python-docx` (better than OCR — parses actual structure)
- For PowerPoint: see the `powerpoint` skill (uses python-pptx)

```

---

## petdex

- **Name:** `petdex`
- **Version:** 1.0.0
- **Author:** Hermes Agent
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/productivity\petdex/SKILL.md`

**Description:**

> Install and select animated petdex mascots for Hermes.

**Full SKILL.md body:**

```markdown
---
name: petdex
description: Install and select animated petdex mascots for Hermes.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [petdex, mascot, display, cli, tui, desktop]
    category: productivity
    homepage: https://petdex.dev
---

# Petdex Skill

Browse, install, and select animated "pet" mascots from the public
[petdex](https://github.com/crafter-station/petdex) gallery. An installed pet
reacts to agent activity (idle, running a tool, reviewing, error, done) across
the Hermes CLI, TUI, and desktop app. This skill drives the `hermes pets` CLI
and the `display.pet` config — it does not generate sprites.

## When to Use

- The user wants a desktop/terminal mascot or asks about "pets" / petdex.
- The user wants to change, preview, or disable the active pet.
- Diagnosing why a pet isn't showing (terminal graphics support, config).

## Prerequisites

- Network access to `petdex.dev` for the gallery/manifest (read-only, no auth).
- Pillow (a core Hermes dependency) for sprite decoding — already installed.
- For full-fidelity terminal rendering: a graphics-capable terminal (kitty,
  Ghostty, WezTerm, iTerm2, or sixel). Otherwise a truecolor Unicode
  half-block fallback is used automatically.

## How to Run

Use the `terminal` tool to run `hermes pets <subcommand>`.

## Quick Reference

| Goal | Command |
| --- | --- |
| Browse the gallery | `hermes pets list` (add a substring to filter: `hermes pets list cat`) |
| List installed pets | `hermes pets list --installed` |
| Install a pet | `hermes pets install <slug>` (add `--select` to make it active) |
| Set the active pet | `hermes pets select <slug>` (omit slug for a picker) |
| Resize the pet everywhere | `hermes pets scale <factor>` (e.g. `0.5`, clamped 0.1–3.0) |
| Preview/animate in terminal | `hermes pets show [slug] [--cycle] [--state run]` |
| Disable the pet | `hermes pets off` |
| Remove a pet | `hermes pets remove <slug>` |
| Diagnose setup | `hermes pets doctor` |

## Procedure

1. Find a pet: `hermes pets list <query>` and note its `slug`.
2. Install + activate: `hermes pets install <slug> --select`.
3. Preview it: `hermes pets show` (Ctrl+C to stop).
4. Confirm setup: `hermes pets doctor` — shows the resolved pet, configured
   render mode, detected terminal graphics protocol, and effective mode.

Pets install into `<HERMES_HOME>/pets/<slug>/` (profile-aware). Selecting a pet
writes `display.pet.slug` + `display.pet.enabled` to `config.yaml`.

## Configuration

Under `display.pet` in `config.yaml`:

- `enabled` (bool) — master on/off.
- `slug` (str) — active pet; empty = first installed.
- `render_mode` — `auto` (detect) | `kitty` | `iterm` | `sixel` | `unicode` | `off`.
- `scale` (float) — on-screen size of the native 192×208 frames (default 0.33,
  clamped 0.1–3.0). One knob resizes every surface; set it with
  `hermes pets scale <factor>`, the `/pet scale` slash command, or the desktop
  Appearance slider.
- `unicode_cols` (int) — width in columns for the Unicode fallback.

## Pitfalls

- A pet only shows once one is installed AND selected (`enabled: true`).
- Inside a pipe/redirect (no TTY) terminal rendering is disabled by design.
- The petdex npm CLI installs to `~/.codex/pets`; Hermes uses its own
  profile-scoped `<HERMES_HOME>/pets/` instead — install through `hermes pets`.

## Verification

- `hermes pets doctor` reports `✓ ready` when a pet is installed, selected,
  enabled, and Pillow is importable.

```

---

## powerpoint

- **Name:** `powerpoint`
- **License:** Proprietary. LICENSE.txt has complete terms
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/productivity\powerpoint/SKILL.md`

**Description:**

> Create, read, edit .pptx decks, slides, notes, templates.

**Full SKILL.md body:**

```markdown
---
name: powerpoint
description: "Create, read, edit .pptx decks, slides, notes, templates."
license: Proprietary. LICENSE.txt has complete terms
platforms: [linux, macos, windows]
---

# Powerpoint Skill

## When to use

Use this skill any time a .pptx file is involved in any way — as input, output, or both. This includes: creating slide decks, pitch decks, or presentations; reading, parsing, or extracting text from any .pptx file (even if the extracted content will be used elsewhere, like in an email or summary); editing, modifying, or updating existing presentations; combining or splitting slide files; working with templates, layouts, speaker notes, or comments. Trigger whenever the user mentions "deck," "slides," "presentation," or references a .pptx filename, regardless of what they plan to do with the content afterward. If a .pptx file needs to be opened, created, or touched, use this skill.

## Quick Reference

| Task | Guide |
|------|-------|
| Read/analyze content | `python -m markitdown presentation.pptx` |
| Edit or create from template | Read [editing.md](editing.md) |
| Create from scratch | Read [pptxgenjs.md](pptxgenjs.md) |

---

## Reading Content

```bash
# Text extraction
python -m markitdown presentation.pptx

# Visual overview
python scripts/thumbnail.py presentation.pptx

# Raw XML
python scripts/office/unpack.py presentation.pptx unpacked/
```

---

## Editing Workflow

**Read [editing.md](editing.md) for full details.**

1. Analyze template with `thumbnail.py`
2. Unpack → manipulate slides → edit content → clean → pack

---

## Creating from Scratch

**Read [pptxgenjs.md](pptxgenjs.md) for full details.**

Use when no template or reference presentation is available.

---

## Design Ideas

**Don't create boring slides.** Plain bullets on a white background won't impress anyone. Consider ideas from this list for each slide.

### Before Starting

- **Pick a bold, content-informed color palette**: The palette should feel designed for THIS topic. If swapping your colors into a completely different presentation would still "work," you haven't made specific enough choices.
- **Dominance over equality**: One color should dominate (60-70% visual weight), with 1-2 supporting tones and one sharp accent. Never give all colors equal weight.
- **Dark/light contrast**: Dark backgrounds for title + conclusion slides, light for content ("sandwich" structure). Or commit to dark throughout for a premium feel.
- **Commit to a visual motif**: Pick ONE distinctive element and repeat it — rounded image frames, icons in colored circles, thick single-side borders. Carry it across every slide.

### Color Palettes

Choose colors that match your topic — don't default to generic blue. Use these palettes as inspiration:

| Theme | Primary | Secondary | Accent |
|-------|---------|-----------|--------|
| **Midnight Executive** | `1E2761` (navy) | `CADCFC` (ice blue) | `FFFFFF` (white) |
| **Forest & Moss** | `2C5F2D` (forest) | `97BC62` (moss) | `F5F5F5` (cream) |
| **Coral Energy** | `F96167` (coral) | `F9E795` (gold) | `2F3C7E` (navy) |
| **Warm Terracotta** | `B85042` (terracotta) | `E7E8D1` (sand) | `A7BEAE` (sage) |
| **Ocean Gradient** | `065A82` (deep blue) | `1C7293` (teal) | `21295C` (midnight) |
| **Charcoal Minimal** | `36454F` (charcoal) | `F2F2F2` (off-white) | `212121` (black) |
| **Teal Trust** | `028090` (teal) | `00A896` (seafoam) | `02C39A` (mint) |
| **Berry & Cream** | `6D2E46` (berry) | `A26769` (dusty rose) | `ECE2D0` (cream) |
| **Sage Calm** | `84B59F` (sage) | `69A297` (eucalyptus) | `50808E` (slate) |
| **Cherry Bold** | `990011` (cherry) | `FCF6F5` (off-white) | `2F3C7E` (navy) |

### For Each Slide

**Every slide needs a visual element** — image, chart, icon, or shape. Text-only slides are forgettable.

**Layout options:**
- Two-column (text left, illustration on right)
- Icon + text rows (icon in colored circle, bold header, description below)
- 2x2 or 2x3 grid (image on one side, grid of content blocks on other)
- Half-bleed image (full left or right side) with content overlay

**Data display:**
- Large stat callouts (big numbers 60-72pt with small labels below)
- Comparison columns (before/after, pros/cons, side-by-side options)
- Timeline or process flow (numbered steps, arrows)

**Visual polish:**
- Icons in small colored circles next to section headers
- Italic accent text for key stats or taglines

### Typography

**Choose an interesting font pairing** — don't default to Arial. Pick a header font with personality and pair it with a clean body font.

| Header Font | Body Font |
|-------------|-----------|
| Georgia | Calibri |
| Arial Black | Arial |
| Calibri | Calibri Light |
| Cambria | Calibri |
| Trebuchet MS | Calibri |
| Impact | Arial |
| Palatino | Garamond |
| Consolas | Calibri |

| Element | Size |
|---------|------|
| Slide title | 36-44pt bold |
| Section header | 20-24pt bold |
| Body text | 14-16pt |
| Captions | 10-12pt muted |

### Spacing

- 0.5" minimum margins
- 0.3-0.5" between content blocks
- Leave breathing room—don't fill every inch

### Avoid (Common Mistakes)

- **Don't repeat the same layout** — vary columns, cards, and callouts across slides
- **Don't center body text** — left-align paragraphs and lists; center only titles
- **Don't skimp on size contrast** — titles need 36pt+ to stand out from 14-16pt body
- **Don't default to blue** — pick colors that reflect the specific topic
- **Don't mix spacing randomly** — choose 0.3" or 0.5" gaps and use consistently
- **Don't style one slide and leave the rest plain** — commit fully or keep it simple throughout
- **Don't create text-only slides** — add images, icons, charts, or visual elements; avoid plain title + bullets
- **Don't forget text box padding** — when aligning lines or shapes with text edges, set `margin: 0` on the text box or offset the shape to account for padding
- **Don't use low-contrast elements** — icons AND text need strong contrast against the background; avoid light text on light backgrounds or dark text on dark backgrounds
- **NEVER use accent lines under titles** — these are a hallmark of AI-generated slides; use whitespace or background color instead

---

## QA (Required)

**Assume there are problems. Your job is to find them.**

Your first render is almost never correct. Approach QA as a bug hunt, not a confirmation step. If you found zero issues on first inspection, you weren't looking hard enough.

### Content QA

```bash
python -m markitdown output.pptx
```

Check for missing content, typos, wrong order.

**When using templates, check for leftover placeholder text:**

```bash
python -m markitdown output.pptx | grep -iE "xxxx|lorem|ipsum|this.*(page|slide).*layout"
```

If grep returns results, fix them before declaring success.

### Visual QA

**⚠️ USE SUBAGENTS** — even for 2-3 slides. You've been staring at the code and will see what you expect, not what's there. Subagents have fresh eyes.

Convert slides to images (see [Converting to Images](#converting-to-images)), then use this prompt:

```
Visually inspect these slides. Assume there are issues — find them.

Look for:
- Overlapping elements (text through shapes, lines through words, stacked elements)
- Text overflow or cut off at edges/box boundaries
- Decorative lines positioned for single-line text but title wrapped to two lines
- Source citations or footers colliding with content above
- Elements too close (< 0.3" gaps) or cards/sections nearly touching
- Uneven gaps (large empty area in one place, cramped in another)
- Insufficient margin from slide edges (< 0.5")
- Columns or similar elements not aligned consistently
- Low-contrast text (e.g., light gray text on cream-colored background)
- Low-contrast icons (e.g., dark icons on dark backgrounds without a contrasting circle)
- Text boxes too narrow causing excessive wrapping
- Leftover placeholder content

For each slide, list issues or areas of concern, even if minor.

Read and analyze these images:
1. /path/to/slide-01.jpg (Expected: [brief description])
2. /path/to/slide-02.jpg (Expected: [brief description])

Report ALL issues found, including minor ones.
```

### Verification Loop

1. Generate slides → Convert to images → Inspect
2. **List issues found** (if none found, look again more critically)
3. Fix issues
4. **Re-verify affected slides** — one fix often creates another problem
5. Repeat until a full pass reveals no new issues

**Do not declare success until you've completed at least one fix-and-verify cycle.**

---

## Converting to Images

Convert presentations to individual slide images for visual inspection:

```bash
python scripts/office/soffice.py --headless --convert-to pdf output.pptx
pdftoppm -jpeg -r 150 output.pdf slide
```

This creates `slide-01.jpg`, `slide-02.jpg`, etc.

To re-render specific slides after fixes:

```bash
pdftoppm -jpeg -r 150 -f N -l N output.pdf slide-fixed
```

---

## Dependencies

- `pip install "markitdown[pptx]"` - text extraction
- `pip install Pillow` - thumbnail grids
- `npm install -g pptxgenjs` - creating from scratch
- LibreOffice (`soffice`) - PDF conversion (auto-configured for sandboxed environments via `scripts/office/soffice.py`)
- Poppler (`pdftoppm`) - PDF to images

```

---

## teams-meeting-pipeline

- **Name:** `teams-meeting-pipeline`
- **Version:** 1.1.0
- **Author:** Hermes Agent + Teknium
- **License:** MIT
- **On-disk path:** `C:/Users/soren/AppData/Local/hermes/skills/productivity\teams-meeting-pipeline/SKILL.md`

**Description:**

> Operate the Teams meeting summary pipeline via Hermes CLI — summarize meetings, inspect pipeline status, replay jobs, manage Microsoft Graph subscriptions.

**Full SKILL.md body:**

```markdown
---
name: teams-meeting-pipeline
description: "Operate the Teams meeting summary pipeline via Hermes CLI — summarize meetings, inspect pipeline status, replay jobs, manage Microsoft Graph subscriptions."
version: 1.1.0
author: Hermes Agent + Teknium
license: MIT
prerequisites:
  env_vars: [MSGRAPH_TENANT_ID, MSGRAPH_CLIENT_ID, MSGRAPH_CLIENT_SECRET]
  commands: [hermes]
metadata:
  hermes:
    tags: [Teams, Microsoft Graph, Meetings, Productivity, Operations]
    related_docs:
      - /docs/guides/microsoft-graph-app-registration
      - /docs/user-guide/messaging/teams-meetings
      - /docs/guides/operate-teams-meeting-pipeline
---

# Teams Meeting Pipeline

Use this skill whenever the user asks about Microsoft Teams meeting summaries, transcripts, recordings, action items, Graph subscriptions, or any operational question about the Teams meeting pipeline. Works in any language — the triggers below are examples, not an exhaustive list.

Everything operator-facing is a `hermes teams-pipeline` subcommand run via the terminal tool. There are no new model tools for this pipeline — the CLI is the surface.

## When to use this skill

The user is asking to:
- summarize a Teams meeting / extract action items / pull meeting notes
- check pipeline status, inspect a stored meeting job, or see recent meetings
- replay / re-run a stored job that failed or needs a fresh summary
- validate Microsoft Graph setup after changing env or config
- troubleshoot "meeting summary never arrived" or "no new meetings are ingesting"
- manage Graph webhook subscriptions (create, renew, delete, inspect)
- set up automated subscription renewal (see pitfall below)

Multilingual trigger examples (not exhaustive):
- English: "summarize the Teams meeting", "pipeline status", "replay job X"
- Turkish: "Teams meeting özetle", "action item çıkar", "toplantı notu", "pipeline durumu", "replay job"

## Prerequisites

Before using the pipeline, verify these are set in `${HERMES_HOME:-~/.hermes}/.env`:

```bash
MSGRAPH_TENANT_ID=...
MSGRAPH_CLIENT_ID=...
MSGRAPH_CLIENT_SECRET=...
```

If any are missing, direct the user to the Azure app registration guide at `/docs/guides/microsoft-graph-app-registration` — they need an Azure AD app registration with admin-consented Graph application permissions before the pipeline will work.

## Command reference

### Status and inspection (start here)

```bash
hermes teams-pipeline validate              # config snapshot — run first after any change
hermes teams-pipeline token-health          # Graph token status
hermes teams-pipeline token-health --force-refresh   # force a fresh token acquisition
hermes teams-pipeline list                  # recent meeting jobs
hermes teams-pipeline list --status failed  # only failed jobs
hermes teams-pipeline show <job-id>         # full detail of one job
hermes teams-pipeline subscriptions         # current Graph webhook subscriptions
```

### Re-running / debugging

```bash
hermes teams-pipeline run <job-id>          # replay a stored job (re-summarize, re-deliver)
hermes teams-pipeline fetch --meeting-id <id>   # dry-run: resolve meeting + transcript without persisting
hermes teams-pipeline fetch --join-web-url "<url>"   # dry-run by join URL
```

### Subscription management

```bash
hermes teams-pipeline subscribe \
  --resource communications/onlineMeetings/getAllTranscripts \
  --notification-url https://<your-public-host>/msgraph/webhook \
  --client-state "$MSGRAPH_WEBHOOK_CLIENT_STATE"

hermes teams-pipeline renew-subscription <sub-id> --expiration <iso-8601>
hermes teams-pipeline delete-subscription <sub-id>
hermes teams-pipeline maintain-subscriptions            # renew near-expiry ones
hermes teams-pipeline maintain-subscriptions --dry-run  # show what would be renewed
```

## Decision tree for common asks

- User asks "why didn't I get a summary for today's meeting?" → start with `list --status failed`, then `show <job-id>` on the relevant row. If the job doesn't exist at all, check `subscriptions` — the webhook may have expired (see pitfall below).
- User asks "is setup working?" → `validate`, then `token-health`, then `subscriptions`. If all three pass, request a test meeting and check `list` for a fresh row.
- User asks "re-run summary for meeting X" → `list` to find the job ID, `run <job-id>` to replay. If it fails again, `show <job-id>` to inspect the error and `fetch --meeting-id` to dry-run the artifact resolution.
- User asks "add meeting X to the pipeline" → usually you don't — the pipeline is subscription-driven, not per-meeting. If they want a specific past meeting summarized, use `fetch` to pull transcript + `run` after a job is created.

## Critical pitfall: Graph subscriptions expire in 72 hours

Microsoft Graph caps webhook subscriptions at 72 hours and **will not auto-renew them**. If `maintain-subscriptions` is not scheduled, meeting notifications silently stop arriving 3 days after any manual subscription creation.

When the user reports "the pipeline worked yesterday but nothing is arriving today":
1. Run `hermes teams-pipeline subscriptions` — if it's empty or all entries show `expirationDateTime` in the past, that's the cause.
2. Recreate with `subscribe` as shown above.
3. **Set up automated renewal immediately** via `hermes cron add`, a systemd timer, or plain crontab. The operator runbook at `/docs/guides/operate-teams-meeting-pipeline#automating-subscription-renewal-required-for-production` has all three options. 12-hour interval is safe (6x headroom against the 72h limit).

## Other pitfalls

- **Transcript not available yet.** Teams takes some time after a meeting ends to generate the transcript artifact. `fetch --meeting-id` on a just-ended meeting may return empty. Wait 2-5 minutes and retry, or let the Graph webhook drive ingestion naturally.
- **Delivery mode mismatch.** If summaries are produced (`list` shows success) but nothing lands in Teams, check `platforms.teams.extra.delivery_mode` and the matching target config (`incoming_webhook_url` OR `chat_id` OR `team_id`+`channel_id`). The writer reads these from config.yaml or `TEAMS_*` env vars.
- **Graph app permissions.** A token acquires cleanly (`token-health` passes) but Graph API calls return 401/403 when permissions were added but admin consent wasn't re-granted. Have the user revisit the app registration in the Azure portal and click "Grant admin consent" again.

## Related docs

Point the user to these when they need more depth than this skill covers:
- Azure app registration walkthrough: `/docs/guides/microsoft-graph-app-registration`
- Full pipeline setup: `/docs/user-guide/messaging/teams-meetings`
- Operator runbook (renewal automation, troubleshooting, go-live checklist): `/docs/guides/operate-teams-meeting-pipeline`
- Webhook listener setup: `/docs/user-guide/messaging/msgraph-webhook`

```

---
