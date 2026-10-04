# Security Policy

## Reporting a Vulnerability

If you find a security issue in this repository, please report it privately rather than opening
a public issue.

**Email:** [ujuonnongjames@gmail.com](mailto:ujuonnongjames@gmail.com)

Please include a description of the issue, the affected file or workflow node, and steps to
reproduce it if available. You can expect an acknowledgement within 7 days.

---

## Secrets Policy

This repository is **safe to publish**. It contains workflow templates only — never live
credentials.

### Never commit

| Secret type | Examples |
|---|---|
| Environment files | `.env`, `.env.local`, `.env.production` |
| n8n credential stores | `*.credentials.json`, anything inside `.n8n/` |
| API keys | OpenAI `sk-…`, Pinecone `pcsk_…`, Cloudinary API key/secret, Telegram bot tokens |
| OAuth client data | Google client secrets, private keys, `*.pem`, `*.key`, `*.p12` |
| Service account files | `service-account*.json` |
| Customer data | Live exports of the `Leads/clients` or `Viewing` sheets |
| Personal identifiers | Real phone numbers, personal e-mail addresses |
| Infrastructure identifiers | n8n `instanceId`, webhook path tokens, private Sheet / Calendar / Drive IDs |

### Public URLs are not secrets

Cloudinary **delivery URLs** for your property photos are meant to be public — customers click them
in the chat. Committing one is fine. What must never be committed or pasted into a spreadsheet is
the Cloudinary **API key**, **API secret**, or any **signed/private** upload URL, since those grant
write access to your media library.

### How the published templates stay safe

Every non-secret value that would otherwise identify live infrastructure has been replaced with
an environment variable reference, and every credential binding has been removed.

| Removed from the templates | Replaced with |
|---|---|
| Credential IDs and credential names | Empty slots — you attach your own in n8n |
| Google Sheets document ID | `$env.GOOGLE_SHEET_ID` |
| Google Calendar ID | `$env.GOOGLE_CALENDAR_ID` |
| Google Drive folder ID | `$env.GOOGLE_DRIVE_FOLDER_ID` |
| Agent e-mail address | `$env.AGENT_EMAIL` |
| Agent name and phone number in the prompt | Read from the database at runtime |
| n8n `instanceId` | Removed |
| Webhook path tokens (`webhookId`) | Removed |
| Pinned execution data (`pinData`) | Cleared |
| Cached Google `cachedResultUrl` / `cachedResultName` | Removed or replaced with safe labels |

Tab names (`Properties`, `Leads/clients`, `Viewing`), the Pinecone index name
(`kaduprimeestate`) and the namespace (`Prime Estate`) are **intentionally kept** — they are
schema identifiers, not secrets, and keeping them makes the templates usable out of the box.

---

## Safe Workflow-Sharing Practice

n8n exports contain credential **references**, and a reference is enough to reveal your account
structure. Before committing any export:

1. In n8n, open the workflow and remove credentials from every node, **or** export, then strip
   them from the JSON.
2. Delete the `meta` block, which contains your `instanceId`.
3. Delete pinned data. Right-click a node → **Pin data → Unpin**. Pinned output can contain real
   customer records.
4. Search the file for `@`, your domain, and any 40+ character ID before committing.
5. Run `git status` and `git diff --staged` and read the output.

### Automating the check

A quick pre-commit guard:

```bash
# Fails if a secret is about to be committed
git diff --staged --name-only | grep -Ei '\.env$|credentials\.json|\.n8n/' && echo "SECRET FILE STAGED - ABORT"
```

---

## Reporting Exposure

If you ever commit a real secret, treat it as compromised — deleting the file in a later commit
is **not** enough, because Git keeps history.

1. **Revoke and rotate the credential immediately** (OpenAI, Pinecone, Telegram, Google OAuth).
2. `git rm --cached <file>` to unstage it.
3. Rewrite history with `git filter-repo` or BFG, then force-push.
4. Contact GitHub Support to garbage-collect cached clones if it was ever pushed publicly.
5. Review access logs on every affected service.

Rotating the credential is always the essential step. History rewriting is cleanup.

---

## Responsible Use

This project automates conversations with real customers and writes their personal data to
Google Sheets. If you deploy it:

- Obtain consent before recording or transcribing voice messages.
- Publish a privacy policy covering what data is collected and how long it is retained.
- Comply with **NDPC** (Nigeria Data Protection Commission) guidance on personal data in Nigeria.
- Encrypt Google Sheets and Drive; restrict sharing to the service account and named agents.
- Keep the Telegram bot token and API keys in a secret manager, never in the workflow JSON.