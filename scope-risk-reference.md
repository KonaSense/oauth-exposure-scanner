# Google OAuth scope risk reference

Reference data for classifying the risk of OAuth scopes granted to third-party apps against a Google account. Source: Google OAuth 2.0 API Scopes documentation (developers.google.com/identity/protocols/oauth2/scopes).

Classification rule: a scope is rated CRITICAL if abuse enables impersonation, financial fraud, or organization-wide data theft. HIGH if it enables reading sensitive user data. MEDIUM if it enables reading moderately sensitive data or writing in limited contexts. LOW if it enables only metadata or public data access.

When a token carries a scope, assume it is valid from any IP, at any hour, until revoked. There is no MFA check on API calls using a live OAuth token.

## CRITICAL scopes

These scopes give an attacker capabilities equivalent to impersonation or admin takeover. Any third-party app holding these must be treated as a root-equivalent dependency.

| Scope | What it grants | Abuse scenario |
|---|---|---|
| `https://mail.google.com/` | Full access to Gmail, read/send/delete | Impersonate user in email threads, CFO wire fraud, delete evidence |
| `gmail.send` | Send email as the user | Write CFO/vendor messages from the real account in real thread history |
| `gmail.modify` | Read, compose, send, permanently delete email | Everything send does plus evidence destruction |
| `admin.directory.user` | Workspace admin: view/manage users | Full user map, add backdoor accounts, reset passwords org-wide |
| `admin.directory.group` | Workspace admin: view/manage groups | Add attacker email to sensitive groups |
| `admin.directory.domain` | Workspace admin: manage domains | Add malicious domains to tenant |
| `cloud-platform` | Full Google Cloud access | Take over GCP projects, read/write all cloud data |
| `https://www.googleapis.com/auth/drive` | Full Drive access (read/write/delete everything) | Exfiltrate every shared doc, overwrite files with malicious versions |

## HIGH scopes

These scopes enable significant data exfiltration even without write access.

| Scope | What it grants | Abuse scenario |
|---|---|---|
| `gmail.readonly` | Read all email | Password resets, SSO backup codes, DocuSign links, vendor wire instructions, M&A threads |
| `drive.readonly` | Read all Drive files | Every shared doc including "internal only" files, contracts, IP |
| `calendar` | Full calendar access, read/write | Org chart via meeting attendees, inject malicious Zoom invites into VP calendars |
| `calendar.events` | Create/modify events | Same attack surface as calendar for meeting injection |
| `contacts` | Read and write contacts | Target list for spear-phishing, pretexting data |
| `cloud_search` | Search across Workspace data | Keyword search for "password", "api key", "wire transfer" across every indexable asset |
| `admin.reports.audit.readonly` | Read audit logs | Understand detection surface, identify blind spots |

## MEDIUM scopes

| Scope | What it grants | Abuse scenario |
|---|---|---|
| `calendar.readonly` | Read calendar only | Org chart, deal pipeline from recurring meeting titles, travel schedule |
| `contacts.readonly` | Read contacts only | Target list generation |
| `tasks` | Tasks read/write | Task content may contain TODO-style notes with credentials |
| `chat.messages` | Read Google Chat | DM content exfiltration |
| `documents` | Specific Docs file access | Scoped, but files in scope are readable |
| `spreadsheets` | Specific Sheets access | Scoped, but sheets in scope are readable |

## LOW scopes

| Scope | What it grants | Abuse scenario |
|---|---|---|
| `userinfo.email` | User's email address | Identity only |
| `userinfo.profile` | Basic profile info | Name, picture, language preference |
| `openid` | OpenID Connect sign-in | Login identity confirmation |

## Scope families to recognize by prefix

When a scope is not explicitly listed above, classify by prefix:

- `admin.directory.*` → CRITICAL (any Workspace directory admin scope)
- `admin.*` → HIGH or CRITICAL (any Workspace admin scope is at least HIGH)
- `cloud-platform*` → CRITICAL
- `mail.google.com` or `gmail.*` (without readonly) → CRITICAL
- `gmail.readonly`, `gmail.metadata` → HIGH
- `drive` (not readonly) → CRITICAL
- `drive.readonly`, `drive.file` → HIGH
- `calendar` (not readonly) → HIGH
- `calendar.readonly` → MEDIUM
- `contacts` → HIGH
- `contacts.readonly` → MEDIUM
- `userinfo.*`, `openid`, `profile`, `email` → LOW

## Known IOCs to check against

Client IDs publicly reported as compromised. If any of these appear in the user's authorized apps list, treat as active incident.

| Client ID | Vendor | Source |
|---|---|---|
| `110671459871-30f1spbu0hptbs60cb4vsmv79i7bbvqj.apps.googleusercontent.com` | Context.ai | Vercel April 2026 security bulletin |

This list is small on purpose. Only include IOCs from official vendor bulletins or reputable incident reports, not speculation.

## Ambient risk patterns

Beyond individual scopes, certain combinations amplify risk:

- **Calendar + Gmail read** on a personal assistant or notetaker = full business intelligence vantage point
- **Drive + Gmail send** on any app = turns a file-access compromise into an impersonation vector
- **Any admin.* scope** granted to a non-Google-published app = treat as critical by default
- **App not used in 90+ days** with any HIGH or CRITICAL scope = revoke; dormant grants are pure attack surface

## References

- Google OAuth 2.0 Scopes: https://developers.google.com/identity/protocols/oauth2/scopes
- Google account third-party access: https://myaccount.google.com/permissions
- Google Workspace admin OAuth apps: Admin console > Security > API controls > App access control
- Vercel April 2026 security bulletin (Context.ai IOC): https://vercel.com/kb/bulletin/vercel-april-2026-security-incident
