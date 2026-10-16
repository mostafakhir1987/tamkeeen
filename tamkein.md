# Tamkeen: Resolving the [TBD] Items

Recommendations for PRD-001 v1.0, prepared 2026-10-16. These are suggestions; the team's skills and constraints can override any of them.

## Accessibility layer delivery

This is the highest-risk decision (R-001), and the tech stack depends on it.

| Option | Works on gov sites? | Main problem |
|---|---|---|
| Browser extension | Yes, runs on the site's real origin | User must install it; limited on mobile browsers |
| Rewriting proxy | Sometimes | Breaks logins, SSO, and CSRF, and routes government credentials through your servers (a serious privacy and trust risk, tied to R-005) |
| iframe wrapper | Rarely | Most government sites send frame-blocking headers, and cross-origin rules stop you from changing the page anyway |

**Recommendation: a Manifest V3 browser extension, with the Tamkeen web app as the launcher.** The web app handles login, disability selection, and website choice. The redirect then opens the target site in a new tab, and the extension reads the access session config (via a short-lived token) and applies the matching feature set. This avoids the iframe and proxy problems, keeps the user's government login on the real site, and fits the non-goal of working alongside screen readers.

### Trade-offs to write into the PRD

- Users must install the extension, so add this to onboarding (P1-F004) and to the first-use flow.
- Chrome on Android does not support extensions, so mobile support is limited to Firefox Android and Safari. State this as a constraint.
- The wording in P0-F003 ("System loads the target website with the layer") should change to "opens the website with the layer applied by the Tamkeen extension."

### Phase 1 spike (before Sprint 1)

Test a minimal extension against the 3 target sites. Pass criteria:

- Text scales to 200% without breakage.
- A keyboard focus indicator can be injected.
- The layer survives page navigation.

This validates R-001 before committing to anything else.

## Tech stack (suggested)

- **Web app:** React + TypeScript, with an accessible component library (React Aria or Radix) so focus management is not hand-built.
- **Extension:** TypeScript, built with WXT or Plasmo, sharing types with the backend.
- **Backend:** Node.js (NestJS or Express) + TypeScript, so one language covers the web app, extension, and API. Django is a fine alternative if the developer knows Python.
- **Database:** PostgreSQL. It fits the relational model and the unique constraints on email and (user_id, website_id).
- **Auth:** Argon2id (or bcrypt) hashing, short-lived access tokens, and a refresh token in an httpOnly cookie.
- **Testing:** axe-core in CI, Playwright for end-to-end, and manual checks with NVDA, VoiceOver, and JAWS.
- **Monitoring:** Sentry for errors and any uptime service for the 99.5% target.

## Other [TBD] items

- **Interface languages:** Arabic and English with full RTL support. Decide this now, since it affects layout, testing, and how the layer handles Arabic text.
- **Session timeout:** 30 to 60 minutes of inactivity, with a warning and an option to extend. Users with physical disabilities need longer timeouts anyway, and WCAG expects this.
- **Email verification:** use a transactional email provider (Postmark, SendGrid, or similar). Add email verification to P0-F001 if wanted, since it is not in the acceptance criteria today.
- **Environments and CI/CD:** dev, staging, and production, with GitHub Actions if the repository is on GitHub.
- **Hosting:** depends on whether government-facing users or partners require data to stay in Egypt or the region. Confirm this early.

## Items only the owner can fill in

Team names, contacts, repository, and design links.

## Next steps

- Rewrite the affected PRD sections (delivery mechanism, tech architecture, constraints, and the P0-F003 wording) with these decisions applied.
- Confirm the 3 target websites for the Phase 1 spike.
