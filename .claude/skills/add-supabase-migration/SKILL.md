---
name: add-supabase-migration
description: Use when an Endpaper task changes the database schema, RLS policies, functions or seed data. Covers creating a Supabase CLI migration, writing RLS deny-by-default, regenerating TypeScript types, and adding cross-user isolation tests. Phase 3+ only.
---

# Add a Supabase migration and regenerate types

**Who may do this:** the Head Manager or the backend-lead only. Never brief this to a Haiku coder (brief §5.3). Every migration gets a Reviewer audit.

**Never** change the schema in the Supabase dashboard. The migration files are the source of truth.

## Steps
1. Make sure local Supabase is running: `pnpm supabase start` (see `CLAUDE.md` for the exact commands once the app is scaffolded).
2. Create the file: `pnpm supabase migration new <short_snake_case_name>`. This writes `supabase/migrations/<timestamp>_<name>.sql`.
3. Write the SQL. For every new table holding user data:
   - `alter table ... enable row level security;` in the same migration as the `create table`.
   - No policy means no access (deny by default). Add explicit policies per operation, keyed on `auth.uid() = user_id`.
   - Add `not null`, foreign keys with deliberate `on delete` behaviour, check constraints and indexes for every foreign key and common filter.
   - Timestamps are `timestamptz`. An entry's "day" is a `date` computed from the user's timezone at recording time (see `docs/plan/D06-data-model.md`).
4. Apply locally from scratch: `pnpm supabase db reset` (runs all migrations plus `supabase/seed.sql`). It must succeed with no manual steps.
5. Regenerate types: `pnpm supabase gen types typescript --local > src/lib/db/types.gen.ts` (path per `docs/plan/D14-repository-layout.md`). Commit the generated file in the same PR.
6. Tests:
   - Add or update an RLS isolation test proving user A cannot select, insert, update or delete user B's rows in the new table.
   - Run `pnpm typecheck && pnpm test`.
7. Update `docs/plan/D06-data-model.md` if the model changed, and note the migration in the PR's Manager Report.

## Checklist before opening the PR
- [ ] Migration applies cleanly with `db reset`
- [ ] RLS enabled on every new user-data table; policies per operation
- [ ] Types regenerated and committed
- [ ] Isolation test added and passing
- [ ] No journal content in seed data beyond obviously fake samples
