# Contributing to TrustBridge Dashboard

Thank you for your interest in contributing! This project is open source and community-driven.

← Back to [README](../README.md) · See also [Architecture](./ARCHITECTURE.md) · [Setup guide](./SETUP.md)

---

## Code of conduct

Be respectful, inclusive, and constructive. Harassment or discrimination is not tolerated.

---

## Ways to contribute

- 🐛 **Bug reports** — Open an issue with reproduction steps
- ✨ **Features** — Discuss in an issue before large changes
- 📖 **Documentation** — Fix typos, improve guides (see `docs/`)
- 🎨 **UI/UX** — Stellar brand alignment, accessibility improvements
- 🧪 **Tests** — Add coverage for Horizon validation, auth edge cases

---

## Development setup

1. Follow [SETUP.md](./SETUP.md) to run the project locally
2. Create a branch from `main`:

   ```bash
   git checkout -b feat/short-description
   ```

3. Make focused changes — one concern per PR
4. Ensure the project builds, tests pass, and types are valid:

   ```bash
   npm run lint       # ESLint
   npm run typecheck  # TypeScript type checking
   npm run test       # Vitest
   npm run build      # Next.js build
   ```

   All of these run in CI and must pass before merging.

---

## Pull request guidelines

### Before opening a PR

- [ ] Issue linked (if applicable)
- [ ] `.env.example` updated if new env vars added
- [ ] Docs updated in `docs/` and linked from README
- [ ] `npm run lint` passes (ESLint)
- [ ] `npm run typecheck` passes (TypeScript strict mode)
- [ ] `npm run test` passes (all tests)
- [ ] `npm run build` passes (Next.js build succeeds)
- [ ] No secrets committed

### PR title format

Use conventional prefixes:

| Prefix | Use case |
|--------|----------|
| `feat:` | New feature |
| `fix:` | Bug fix |
| `docs:` | Documentation only |
| `refactor:` | Code change without behavior change |
| `chore:` | Tooling, deps, CI |

Example: `feat: add testnet Horizon toggle`

### PR description

Include:

1. **What** changed
2. **Why** it was needed
3. **How to test** locally

---

## Project conventions

### TypeScript

- Strict mode enabled — avoid `any`
- Shared types in `src/types/`
- Server-only logic in `src/lib/`, not in client components

### Components

- UI primitives: `src/components/ui/`
- Feature components: `src/components/`
- Use `"use client"` only when needed (hooks, browser APIs)

### Styling

- Tailwind utility classes
- Stellar brand: `stellar-purple` (#3E1BDB), `stellar-cyan` (#00B4D8)
- Support dark mode via `next-themes`

### API routes

- Use `export const runtime = "nodejs"` for stellar-sdk routes
- Use `export const dynamic = "force-dynamic"` for DB-backed routes
- Return structured JSON errors with appropriate HTTP status codes

### Database

- Schema changes require Prisma migration or documented `db:push` step
- Update [ARCHITECTURE.md](./ARCHITECTURE.md) if data model changes

---

## Reporting bugs

Include:

1. Steps to reproduce
2. Expected vs actual behavior
3. Browser/OS version
4. Relevant env config (redact secrets)
5. Console or server logs

---

## Feature requests

Open an issue describing:

- Problem statement (e.g. "Maintainers can't filter by org team")
- Proposed solution
- Alternatives considered

---

## Questions?

Open a [GitHub Discussion](https://github.com/your-org/trustbridge-dashboard/discussions) or issue with the `question` label.

---

## Data retention and deletion policy

Contributors can export and delete their own registration data via the
self-service API endpoints:

| Endpoint | Method | Description |
|---|---|---|
| `/api/register/export` | `GET` | Download your registration, address history and public profile fields as JSON. |
| `/api/register/delete` | `POST` | Soft-delete your active registration (send `{ "confirm": true }`). |

### What is deleted on request

- **Registration record** — soft-deleted (`deletedAt` set). The Stellar address
  is immediately freed by the partial unique index and can be re-registered by
  you or another contributor.
- **OAuth tokens** — `access_token` and `refresh_token` in the `Account` table
  are cleared. The in-memory JWT session expires naturally (see
  [docs/SESSIONS.md](./SESSIONS.md)); nothing can invalidate it earlier.

### What is NOT deleted (and why)

- **Audit log entries** (`AuditLog` table) — these exist to investigate fraud,
  abuse, and payment disputes and are security-purpose records. They must be
  retained regardless of individual deletion requests. Entries reference the
  actor by user ID and GitHub login at the time of the action; the user record
  itself is also retained for foreign-key integrity with these logs.
- **Address history** (`AddressHistoryRecord` table) — retained for dispute
  resolution. The data is limited to Stellar G-addresses, which are already
  public on the Stellar ledger.

### What is NOT exported (and why)

- **Encrypted access token** — returning ciphertext would be meaningless and
  returning plaintext would be a credential leak. Neither is included.
- **Security audit logs** — logs contain other actors' actions that reference
  this user as a target; exporting them in full would leak information about
  those actors.
- **Session rows** — never written in JWT mode (see [docs/SESSIONS.md](./SESSIONS.md)).

All mutating endpoints enforce CSRF same-origin protection via `assertSameOrigin`.

---

## Related docs

- [Architecture](./ARCHITECTURE.md)
- [Project structure](./PROJECT_STRUCTURE.md)
- [Environment variables](./ENVIRONMENT.md)
- [Sessions](./SESSIONS.md)
