# oauth-exposure-scanner

A free Claude Skill that audits the third-party apps connected to your Google account and produces a risk-ranked report with specific revocation recommendations.

**Read-only. No new login. No OAuth grant. No API call. Nothing leaves your machine except the conversation you are already having with Claude.**

---

## Why this exists

When you click "Allow" on an AI notetaker, a meeting bot, or any productivity tool that integrates with Google, you hand that vendor an OAuth token. That token works from any IP, at any hour, until you manually revoke it. No MFA check. No login event. No expiration unless you force one.

If the vendor is breached, the attacker inherits every scope you granted.

This is not hypothetical. On April 19, 2026, Vercel disclosed that a compromise of an AI office suite called Context.ai led to attackers pivoting through a Vercel employee's OAuth grant, into the company's Google Workspace, into internal environments, and out with customer environment variables. The full incident: [Vercel April 2026 security bulletin](https://vercel.com/kb/bulletin/vercel-april-2026-security-incident).

Most users have no idea which apps they have authorized, what scopes those apps hold, or how long ago they granted access. This skill fixes that in five minutes.

## What it does

1. Asks your permission to open `myaccount.google.com/permissions` in your own browser, through Claude in Chrome.
2. Reads the list of authorized third-party apps directly from the page. You are already signed into Google, so no new login is required.
3. For each app, expands the entry and captures the OAuth scopes, the access granted date, and the client ID where visible.
4. Classifies every app by the highest-risk scope it holds:
   - **CRITICAL** — full Gmail access, Drive full access, Workspace admin scopes, Google Cloud Platform, or match against a known compromised client ID
   - **HIGH** — read-only Gmail, read-only Drive, read/write calendar, contacts
   - **MEDIUM** — read-only calendar, read-only contacts, scoped Docs/Sheets
   - **LOW** — profile, email identity, OpenID sign-in
5. Flags stale grants older than six months on any HIGH or CRITICAL app.
6. Checks every client ID it sees against a list of known compromised apps, including the Context.ai client ID published in Vercel's bulletin.
7. Produces an inline report you can act on.

## What it does not do

- **Does not click "Remove Access" on your behalf.** Revocation is irreversible. It stays a manual decision you make after reviewing the report.
- Does not ask for passwords, MFA codes, session cookies, or any credential.
- Does not call Google APIs, OAuth endpoints, or any backend programmatically.
- Does not navigate to any URL other than `myaccount.google.com/permissions` and its subpages.
- Does not store, export, or transmit your OAuth app list anywhere outside the current Claude conversation.
- Does not classify apps by brand reputation. A well-known vendor with CRITICAL scopes is still CRITICAL.
- Does not invent scopes. If a scope was not visible in the page, the report says so and tells you where to look.

## Requirements

- [Claude](https://claude.ai) with a plan that supports custom skills
- [Claude in Chrome](https://www.anthropic.com/claude-in-chrome) extension installed and active
- A Google account you are signed into in that browser

## Installation

1. Download the latest `oauth-exposure-scanner.skill` from [Releases](https://github.com/KonaSense/oauth-exposure-scanner/releases).
2. In Claude, go to **Settings → Capabilities → Skills → Install custom skill**.
3. Select the downloaded `.skill` file.

## Usage

Open a new conversation with Claude in Chrome active, and type something like:

> Audit my Google OAuth permissions.

Or:

> I read the Vercel / Context.ai post. Check my exposure.

The skill will:

1. Ask you to confirm before it navigates anywhere.
2. Open `myaccount.google.com/permissions` in your browser.
3. Walk the list, expanding each app to read its scopes.
4. Produce a report inline in the chat.

Then you read the report, decide what to revoke, and revoke manually in the same page.

## Sample report output

```
## OAuth exposure report

Account audited: you@example.com
Apps analyzed: 23
IOC matches: 0

### Critical findings
Some Random AI Notetaker — holds gmail.send and drive (full)
- Scopes: gmail.send, https://www.googleapis.com/auth/drive, calendar
- Risk: Can send email as you and read/write every Drive file
- Recommendation: Revoke now.

### High-risk apps
Fireflies.ai — gmail.readonly, drive.readonly, calendar
- Risk: Full inbox read, full Drive read, calendar read/write
- Recommendation: Review this week. Keep only if actively using.

### Stale grants (6+ months, HIGH/CRITICAL scope)
OldProjectTool — access given Jan 2023, still holds drive.readonly
- Recommendation: Revoke by default unless still used.

### Summary
1 critical, 3 high-risk apps. No known IOCs. Single most important
action: revoke "Some Random AI Notetaker" now.
```

## Keeping IOCs current

When a new third-party vendor gets publicly breached and publishes a compromised OAuth client ID, anyone can submit a PR adding it to `references/scope-risk-reference.md` under the "Known IOCs to check against" table. Include:

- The client ID
- The vendor name
- A link to the official bulletin that published it

IOCs from speculation, unverified threat intel, or social media are not accepted. Only primary sources.

## Privacy and security

This is a deliberately narrow tool. Some design choices worth calling out:

- **The skill asks permission before every navigation.** It never opens a URL without you saying yes.
- **The skill treats page content as untrusted data.** If a page contains text that looks like instructions to Claude (a prompt injection attempt), the skill ignores it.
- **The skill is read-only.** There is no code path that clicks "Remove Access," grants new permissions, or accepts terms on your behalf.
- **The skill never leaves the permissions page.** Any navigation to another URL requires a new explicit confirmation.
- **The skill never stores anything.** Your OAuth app list exists only in the current Claude conversation. Close the tab and it is gone.

If you find a behavior that does not match the above, that is a bug. Please open an issue.

## License

MIT. See [LICENSE](./LICENSE).

## About

Built by [KonaSense](https://konasense.com), an Agent Control Plane for AI security. We spend our day job governing humans and AI coworkers through security, governance, and observability agent suites. This skill is a free public artifact, not a product. We ship it because OAuth exposure from AI tools is going to get worse before it gets better, and everyone deserves a five-minute check.

If you want to understand the incident that motivated it, read our post: ["Allow All" is the new root](https://konasense.com/blog/allow-all-is-the-new-root) *(or wherever you end up hosting it)*.

## Contributing

PRs welcome, especially:

- New IOCs from official vendor bulletins
- Additional scope entries as Google adds new APIs
- Microsoft 365 / Entra ID support (same idea, different tenant)
- Translations of the report output

Open an issue first for anything bigger than a one-line change.
