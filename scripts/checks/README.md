# Static checks

Dependency-free Node scripts. They run from the project root and need no install.

- `imports.js`: every local import resolves, and every package import is declared in package.json.
- `named-imports.js`: every named or default import from a local module is actually exported there.
- `next-boundaries.js`: "use server" files export only async functions, and "use client" files do not import server-only modules (env, session, auth, database, Node built-ins).

Run all three with `pnpm check:static`.
