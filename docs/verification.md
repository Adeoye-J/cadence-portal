# Verification record

This file records what was actually run against the code, and what was not. It is the only
place that claims verification. Anything not listed here has not been verified.

## Environment for this record

The build sandbox had no network access and no npm registry, so `pnpm install` could not run. The
checks below used the offline TypeScript 6.0, Zod 3.25 (whose `zod/v4` entry point is the Zod 4
API used by the code), esbuild 0.28 and Node 22. Drizzle, Next.js, React types, Better Auth, Vitest
and the Lucide package were not available.

## Results

| Check | Scope | Result |
|---|---|---|
| Type check (strict, `noUncheckedIndexedAccess`) | Pure modules, schemas, DB schema, all unit tests | Passes. Drizzle's `pg-core` and Vitest were replaced by typed stand-ins (see below). |
| Unit tests | 17 files, 99 tests (format, milestones, sign-off, payments, invitations, milestone flow, change-request replies, outbox and retries, notifications, events, file rules and uploads, file removal and the abandoned-upload sweep, cron authorisation, SigV4 signing, studio access, proposal builder, email templates) | All 95 pass. Run through a Vitest stand-in; see below. |
| Import resolution | 123 files: local paths, `@/` aliases, declared packages | 0 problems |
| Named and default imports | 440 local imports | 0 problems |
| Next.js boundary rules | 121 files: "use server" exports, "use client" server-only imports | 0 problems |

## What the stand-ins mean

- **drizzle-orm/pg-core** was replaced by a typed stub. Enum value tuples keep their literal types,
  which is what the code relies on. No query API was exercised, and the schema has never run against Postgres.
- **vitest** was replaced by a small runner that supports `describe`, `it`, and the matchers the tests use
  (`toBe`, `toEqual`, `toMatchObject`, `toContain`, `toHaveLength`, `toBeNull`, `toBeTruthy`, `toMatch`,
  `toThrow`, and `rejects.toThrow`). It runs tests in order, as Vitest does within a file.
  Real Vitest will report the same results or a clearer failure, but this has not been run under it.

## Not verified

- **React components and pages (`.tsx`).** Only their syntax and their imports were checked. No type check
  ran, because `@types/react` and `next` were unavailable.
- **Next.js runtime behaviour.** `redirect`, `notFound`, `headers()`, the `params` promise, `useActionState`, and
  the form-action flow have not run.
- **Better Auth.** Its API and table names are from memory (docs/assumptions.md, item 31).
- **Paystack.** Its endpoints, the webhook signature header, and the response shapes are from memory (item 37 onwards).
- **SigV4 signing.** Checked against AWS's documented presigned-URL example, which matches exactly. Not checked against Cloudflare R2 or MinIO (assumptions, item 87).
- **Object storage.** The S3-compatible adapter, the upload and download flow in the browser, and the CSP change have not been run against a live bucket.
- **Database.** No migration has been generated or applied. No query has run. This includes the production outbox (`server/outbox-drizzle.ts`), which is unverified against drizzle-orm.
- **Browser behaviour.** No end-to-end, visual, or accessibility test has run.
- **Email.** The Resend call and the templates have not been sent.

## Tooling added but never run

| Item | File | State |
|---|---|---|
| CI workflow (install, static checks, typecheck, lint, format check, tests, audit, build) | .github/workflows/ci.yml | Parses as YAML. Has never run on GitHub. |
| Dependabot (npm and GitHub Actions, weekly) | .github/dependabot.yml | Parses as YAML. Not active until pushed. |
| Prettier configuration | .prettierrc.json, .prettierignore | Valid JSON. Never applied to the code. |
| Pre-commit hook (lint-staged) | .husky/pre-commit, package.json | Not installed. Needs a git repository. |
| Lockfile | pnpm-lock.yaml | Does not exist yet. CI uses --frozen-lockfile and will fail until it is committed. |

Follow docs/first-run.md. It sets out the order and the failures to expect.

## How to run the real checks

```bash
pnpm install
pnpm check:static      # the three checks above, no dependencies
pnpm typecheck         # the full project, including pages and components
pnpm test              # Vitest, the unit tests
pnpm db:generate       # then review db/migrations before applying
```
