# Prompt 03 — Extract Data Model

**Input:** `--repo <path>` (specifically `supabase/migrations/`, `supabase/config.toml`, `src/integrations/supabase/types.ts`)
**Output:** `<out>/.scratch/data-model.json`

## Purpose

Derive the stack-free data model. The output must let a reimplementer rebuild the same schema on Postgres, MySQL, DynamoDB, or SQLite without caring that Lovable used Supabase.

## Output shape

```json
{
  "source": "supabase-migrations|prisma|drizzle|other",
  "tables": [
    {
      "name": "sessions",
      "purpose": "one sentence — what this table represents in the domain",
      "columns": [
        { "name": "id", "type": "uuid", "nullable": false, "pk": true, "default": "gen_random_uuid()" },
        { "name": "created_at", "type": "timestamp", "nullable": false, "default": "now()" },
        { "name": "user_id", "type": "uuid", "nullable": false, "fk": "auth.users(id)" }
      ],
      "indexes": [{ "columns": ["user_id"], "unique": false }],
      "rls": {
        "enabled": true,
        "policies": [
          { "name": "users can read own sessions", "for": "select", "using": "auth.uid() = user_id" }
        ]
      },
      "evidence": ["supabase/migrations/20240115_init.sql:12"]
    }
  ],
  "relationships": [
    {
      "from": "session_assignments.session_id",
      "to": "sessions.id",
      "kind": "many-to-one",
      "on_delete": "cascade"
    }
  ],
  "enums": [
    { "name": "role_kind", "values": ["admin", "consultant"], "evidence": ["..."] }
  ],
  "storage_buckets": [
    { "name": "documents", "public": false, "purpose": "session document uploads" }
  ],
  "auth": {
    "provider": "supabase-auth",
    "methods": ["email_password"],
    "evidence": ["supabase/config.toml:..."]
  },
  "portability_notes": [
    "auth.uid() is Supabase-specific — reimpl needs equivalent 'current authenticated user id' function",
    "RLS policies assume Postgres; on non-PG stacks translate to application-layer middleware",
    "gen_random_uuid() → use DB-native UUID or app-generated UUIDv7"
  ]
}
```

## How

- Read migrations in order; build cumulative schema state (CREATE/ALTER/DROP).
- Use `src/integrations/supabase/types.ts` to cross-check final column types.
- Derive `purpose` from chat-insights when available (look up table name in decisions/business_rules); otherwise note `[TBD]`.
- Capture RLS policies verbatim — they encode authorization rules.
- Always populate `portability_notes` for any Supabase/Postgres-specific construct.
- Don't include seed data or test fixtures.
