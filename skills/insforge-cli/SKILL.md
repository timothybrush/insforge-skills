---
name: insforge-cli
description: Use this benchmark-focused skill for InsForge CLI database work: SQL migrations, raw SQL inspection, RLS policies, schema grants, indexes, triggers, functions, imports, and exports. For app code with @insforge/sdk, use the app SDK skill instead.
license: MIT
metadata:
  author: insforge
  version: "benchmark-db-only"
  organization: InsForge
  date: June 2026
---

# InsForge CLI Database Skill

This branch provides a DB-focused InsForge CLI skill snapshot for benchmark runs. It keeps the CLI guidance narrow so agents spend less context on unrelated backend modules.

## CLI Invocation

Always use:

```bash
npx @insforge/cli <command>
```

In benchmark workspaces, assume the project is already linked to the local sandbox backend. Do not run login, create, link, project discovery, organization listing, or cloud project commands.

## Database Workflow

- Prefer `npx @insforge/cli db migrations new <name>` plus a migration SQL file for schema, grants, indexes, triggers, functions, and RLS policy changes.
- Apply migrations with `npx @insforge/cli db migrations up --all`.
- Use `npx @insforge/cli db query <sql>` for inspection, targeted SQL checks, schema-cache reloads, and small corrective SQL only when a migration is not appropriate.
- Inspect current state with `db tables`, `db indexes`, `db policies`, `db triggers`, `db functions`, and `db migrations list`.
- Reload PostgREST schema cache after schema or policy changes when needed: `npx @insforge/cli db query "NOTIFY pgrst, 'reload schema'"`.
- For benchmark DB tasks, do all application database work in the `public` schema.
- Create, alter, drop, grant, revoke, index, trigger, function, view, and policy changes only for `public` application objects.
- Do not create custom schemas or write to InsForge-managed/system schemas such as `auth`, `storage`, `realtime`, `payments`, `graphql`, `extensions`, `pg_catalog`, `information_schema`, or `system`.
- It is allowed to reference built-in objects such as `auth.users(id)` and `auth.uid()` from public tables or public RLS policies; do not modify those built-in objects.
- Do not create users, seed business rows, or run application CRUD workflows unless the task explicitly asks for data migration or repair. Benchmark verifiers handle user creation and behavior checks.

## RLS Guidance

- Use `auth.uid()` or an equivalent authenticated identity expression for user ownership checks.
- Add both SQL privileges and RLS policies. Policies do not replace `GRANT`.
- Prefer helper functions for cross-table RLS checks when direct policy joins can recurse through other RLS policies.
- Helper functions called from RLS policies that query RLS-enabled tables should be `SECURITY DEFINER`.
- Put RLS helper functions in `public`.
- For production-quality `SECURITY DEFINER` helpers, set a fixed search path, for example `SET search_path = public`.
- Include `WITH CHECK` for INSERT and UPDATE policies so writes cannot create rows the user should not own.

## References

- `references/db-migrations.md` - migration file creation and apply workflow.
- `references/db-query.md` - raw SQL execution and inspection.
- `references/db-rls.md` - InsForge/Postgres RLS patterns, recursion avoidance, and policy guidance.
- `references/db-export.md` / `references/db-import.md` - schema or data import/export tasks.
