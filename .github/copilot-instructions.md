# AINU Credential Helper: Agent Instructions

## What this webpage is for

`index.html` is a single-file, no-build, no-dependency web page titled **"AINU Account Ticket Helper"**.

Users fill in a form to describe the accounts they need (environment, PortalPartner, account types, permissions, RSA key choices, clone sources, IP/domain whitelist). The page validates the input and generates:

- a **Jira ticket title**, shown in `#outTitle`
- a **Jira ticket description** in Markdown, shown in `#outDesc`

The user copies both into a new Jira ticket that follows the template in **AINU-5555**.

Key points in the code:
- All HTML, CSS, and JS live in `index.html`. Keep it that way unless asked otherwise.
- `PERMS` and `KINDS` define the permissions and account types. `validate()` holds the rules. `buildTitle()` and `buildDesc()` produce the output.
- Output must match the AINU-5555 template headings: `### 0. PortalPartner`, `### 1. Demo account`, `### 2. API account`, `### 3. Mobile account`, `### 4. EKYC sdk and web liveness account`, `### 5. IP and Domain whitelist`.

## Required behavior: generate title and description for AINU-5555

This page **must generate a title and description for AINU-5555**. When working on this project:

1. Read the AINU-5555 template and its requirements from Jira (see below).
2. Make sure the generated title and description match that template's structure and wording.
3. Do not break the generated output for any account type or combination.

## How to read AINU-5555 from Jira

- Template ticket: `https://jira-kx.atlassian.net/browse/AINU-5555` (Jira Cloud, summary "Customer accounts creation template").
- The Jira API credentials are already set as environment variables on this machine: `JIRA_EMAIL` and `JIRA_API_TOKEN`. Use them as they are.
- Do not print, log, hard-code, or commit the credentials or the API token. Do not echo their values; checking that they are set (e.g. `env | grep JIRA | sed -E 's/=.*/=<set>/'`) is enough.
- Fetch the issue with Basic auth (`-u "$JIRA_EMAIL:$JIRA_API_TOKEN"`):
  - `curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" -H "Accept: application/json" "https://jira-kx.atlassian.net/rest/api/3/issue/AINU-5555?fields=summary,description" -o /tmp/ainu-5555.json`
- The `description` is Atlassian Document Format (JSON). Convert it to text before comparing it with the code. Walk the nodes: `text` nodes, `heading` (level), `listItem` (`- `), `paragraph`, `hardBreak`.
- Store the response only in `/tmp`. Do not commit it to the repo.
- The template can change. Re-fetch it before each change to `buildTitle()` or `buildDesc()`.

## Current template rules (AINU-5555, as of the last fetch)

- Title: `[PROD/UAT] Create accounts for <customer>` or `[PROD/UAT] Add licenses to existing account <accountId> for <customer>`.
- `### 1. Demo account`: `accountId: demo-<customer>`, `secret: <secret>`, permissions.
- `### 2. API account`: `accountId: <customer>001`, permissions, and **only** `Create new RSA Key for liveness signing: Yes/No`. No DOPA or Webhook RSA key lines for API accounts.
- `### 3. Mobile account`: `accountId:` followed by appId/bundleId list, permissions (smart-capture, liveness), and `Clone RSA Key from: <API account>` (liveness RSA key).
- `### 4. EKYC sdk and web liveness account`: `accountId: ekycsdk-<customer>`, `Use webhook: Yes/No`, `If use webhook, create new RSA Key: Yes/No` (created, not cloned). Permissions: face compare, ocr thaicard (front/back/fraud), web liveness with `Clone RSA Key from: <API account>` (liveness key), DOPA with `Create new RSA Key: Yes/No` (created, not cloned).
- `### 5. IP and Domain whitelist`: `IP Whitelist` and `Domain Whitelist` lists.
- The **only** RSA key that is cloned from an API account (chosen from the page's API accounts) is the liveness key (web liveness in EKYC, and mobile). API accounts never create DOPA or Webhook keys. DOPA and Webhook keys in the `ekycsdk-*` account are either created, or cloned from a manually typed accountId + keyId.

## Working rules

- Keep `index.html` self-contained: no build step, no external dependencies.
- After any change, check that:
  - the title format is `[<ENV>] Create accounts for <PortalPartner>` or `[<ENV>] Add licenses to existing account <ID> for <PortalPartner>`
  - the description keeps the section order and headings above
  - validation still blocks invalid input (empty environment, PortalPartner placeholders like `<` or `>`, invalid account IDs, missing permissions, duplicate IDs)
- Test by opening `index.html` in a browser and filling in each account type.
- Keep changes small and in the existing code style (ES5-style JS, `var`, plain DOM APIs).
