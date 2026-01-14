---
description: "Create implementation plan for a feature (TypeScript/Bun/Hono)"
---

# Plan Feature Implementation

## Feature: $ARGUMENTS

## Mission

Create a **comprehensive implementation plan** for a feature through:
- Codebase analysis
- External research
- Strategic planning

**Goal**: Enable one-pass implementation success.

**Philosophy**: The plan must contain ALL info needed - patterns, docs, validation commands.

---

## Planning Process

### Step 1: Feature Understanding

**Analyze:**
- Core problem being solved
- User value and business impact
- Feature type: New Capability / Enhancement / Refactor / Bug Fix
- Complexity: Low / Medium / High
- Affected systems (Backend / Frontend / Both)

**User Story:**
```
As a <user type>
I want to <action>
So that <benefit>
```

---

### Step 2: Read Project Rules

**CRITICAL - Read these files BEFORE planning:**

**Backend & Database:**
- `rules/backend.md` - Hono patterns, route organization, CORS, error handling
- `rules/database.md` - Drizzle ORM conventions, schema patterns, migrations
- `rules/security.md` - JWT, environment variables, API security

**TypeScript & Error Handling:**
- `rules/typescript.md` - Type conventions, arktype optional properties
- `rules/error-handling.md` - tryAsync/trySync patterns from wellcrafted

**Note key patterns:**
- Hono context pattern: `app.get('/', async (c) => {...})`
- Database: `generatedAlwaysAsIdentity()` for primary keys
- Tables: plural snake_case, columns: snake_case
- Error handling: Services return `Result<Data, Error>`

---

### Step 3: Codebase Pattern Research (Using Agents)

**Use Task tool to spawn codebase-pattern-finder agent:**

**Research 1: Service Organization**
```
Spawn agent: codebase-pattern-finder
Prompt: "Find service organization patterns in apps/whispering/src/lib/services/.
Show how services are structured, export patterns, factory functions, object method shorthand."
```

**Research 2: Error Handling**
```
Spawn agent: codebase-pattern-finder
Prompt: "Find error handling patterns using tryAsync and trySync from wellcrafted.
Show Result type usage, ServiceErr patterns."
```

**Research 3: File Organization**
```
Spawn agent: codebase-pattern-finder
Prompt: "Find file naming conventions and module organization in apps/whispering/src/lib/.
Show directory structure patterns, index.ts exports."
```

**Document findings for plan:**
- Service factory pattern (if used)
- Error handling approach
- File/folder naming conventions
- Import patterns

---

### Step 4: Project Context

**Read from PRD:**
- `PRD-FINAL-RU.md` - Find section for Phase being planned
- Extract goals, scope, deliverables

**Identify workspace:**
- `apps/api` (Backend with Hono + Drizzle)
- `apps/whispering` (Frontend with Svelte 5 + Tauri)
- Both

**Integration Points:**
- Files to create
- Files to modify
- Dependencies to install

---

### Step 5: External Research

**Core Documentation:**
- Hono: https://hono.dev/docs/api/routing (routing, validation)
- Drizzle ORM: https://orm.drizzle.team/docs/sql-schema-declaration (schema, queries)
- Supabase: https://supabase.com/docs/guides/auth (auth, database)
- Svelte 5: https://svelte.dev/docs/svelte/$state (runes, components)
- TanStack Query: https://tanstack.com/query/latest/docs/framework/svelte/overview (mutations, queries)

**Research specific sections:**
- Latest library versions and breaking changes
- Best practices for tech stack
- Common gotchas and known issues
- Security considerations
- Performance patterns

**Link specific documentation sections in plan** (not just main pages)

---

### Step 6: Strategic Thinking

**Consider:**
- Architecture fit
- Critical dependencies
- Edge cases and errors
- Testing strategy
- Security implications
- Performance impact

---

### Step 7: Create Implementation Plan

Combine all research (rules, agents, PRD, external docs) and generate comprehensive plan using template below:

---

## PLAN TEMPLATE

```markdown
# Feature: <feature-name>

## Overview

**Description:** <What this feature does>

**User Story:**
As a <user>
I want to <action>
So that <benefit>

**Type:** [New Capability / Enhancement / Refactor / Bug Fix]
**Complexity:** [Low / Medium / High]
**Affected:** [Backend / Frontend / Both]

---

## Context

### Project Conventions (READ THESE!)

**Backend Rules:** `rules/backend.md`
- Hono context pattern: Always use single `c` parameter
- Routes are thin (< 20 lines), services contain logic
- Use `zValidator` for all request validation
- One service per domain

**Database Rules:** `rules/database.md`
- Primary keys: Use `generatedAlwaysAsIdentity()`
- Tables: plural snake_case (`users`, `usage_logs`)
- Columns: snake_case (`created_at`, `user_id`)
- Always include `created_at` and `updated_at` timestamps

**TypeScript Rules:** `rules/typescript.md`
- Use `type` not `interface`
- Optional properties: `key?: string` (NOT `key: string | undefined`)

**Error Handling:** `rules/error-handling.md`
- Use `tryAsync` for async operations
- Use `trySync` for sync operations
- Services return `Result<Data, Error>` types

### Related Files (READ THESE FIRST!)

**Existing (for patterns):**
- `path/to/file.ts` (lines X-Y) - Why: Shows pattern we'll follow

**New (to create):**
- `path/to/new-file.ts` - Purpose: ...

### Patterns from Codebase (Agent Research)

[Include findings from codebase-pattern-finder agent]

**Service Organization Pattern:**
```typescript
// Example from apps/whispering
```

**Error Handling Pattern:**
```typescript
// Example from apps/whispering
```

### Documentation

- [Library Docs](https://example.com/docs#section)
  - Section: Specific feature name
  - Why: Implementation guide for X

### Code Patterns

**Hono Route Pattern:**
```typescript
app.post('/endpoint', zValidator('json', schema), async (c) => {
  const data = c.req.valid('json');
  // logic
  return c.json(result);
});
```

**Drizzle Query Pattern:**
```typescript
const record = await db.query.table.findFirst({
  where: eq(table.id, id),
});
```

**Svelte 5 Pattern:**
```typescript
let state = $state(initialValue);
let derived = $derived(computation);
```

---

## Implementation Tasks

### Task 1: [Name]

**FILE:** `path/to/file.ts` (CREATE/UPDATE)

**ACTION:**
- Step 1
- Step 2
- Step 3

**CODE:**
```typescript
// Example implementation
```

**VALIDATE:**
```bash
cd apps/api && bun run type-check
```

### Task 2: [Name]

[Same structure...]

---

## Dependencies

**Install:**
```bash
cd apps/api && bun add package-name
cd apps/whispering && bun add package-name
```

**Environment Variables:**
```bash
# .env (root)
NEW_VAR="value"
```

---

## Testing

**Unit Tests:**
- Test X
- Test Y

**Manual Validation:**
- Step 1: Do X
- Step 2: Check Y
- Expected: Z

---

## Validation Checklist

Execute in order:

```bash
# Type check
cd apps/api && bun run type-check

# Lint
bun run lint:check

# Format
bun run format:check

# Build (if backend)
cd apps/api && bun run build

# Manual test
curl http://localhost:3000/endpoint
```

---

## Acceptance Criteria

- [ ] Feature works as specified
- [ ] All validation commands pass
- [ ] Code follows project conventions
- [ ] No regressions
- [ ] Tests added (if applicable)

---

## Notes

- Architecture decisions
- Trade-offs
- Future improvements
```

---

## Output

**Save plan to:** `specs/<feature-name>.md`

**Naming:** Use kebab-case (e.g., `add-auth-middleware.md`)

---

## Quality Checklist

- [ ] All patterns identified
- [ ] Documentation linked
- [ ] Tasks ordered by dependency
- [ ] Each task has validation
- [ ] Specific file paths provided
- [ ] Code examples included

---

## Project-Specific Context

**Tech Stack:**
- Backend: Hono + Drizzle ORM + PostgreSQL (Supabase)
- Frontend: Svelte 5 + TanStack Query + Tauri 2
- Package Manager: Bun
- Monorepo: Workspaces (apps/*, packages/*)

**Key Services:**
- Supabase: Database + Auth
- Railway: Backend hosting
- ElevenLabs: Transcription API
- Stripe: Payments

**Conventions:**
- Follow rules in `rules/*.md`
- Error handling: Use `tryAsync`/`trySync` from wellcrafted
- Types: Use `type` not `interface`
- Validation: Use `arktype`

---

## Report Format

After creating plan, report:

**Summary:**
- Feature: <name>
- Complexity: Low/Medium/High
- Files affected: N new, M modified
- Dependencies: X packages

**Plan Location:** `specs/<feature-name>.md`

**Key Risks:**
- Risk 1
- Risk 2

**Confidence:** X/10 for one-pass success

**Next Steps:** Review plan, then execute with implementation command

---

## IMPORTANT: Use Living Context

If this command is being executed in an active conversation where technical decisions were discussed:

**INCLUDE context from discussion:**
- Infrastructure choices (Supabase Cloud vs local, Railway setup)
- Environment decisions (.env location, credentials status)
- Database decisions (which tables to create, RLS policies)
- Architectural decisions (auth strategy, deployment approach)
- Any other technical decisions made during discussion

**Reference discussion context in plan sections:**
- Prerequisites (what's already setup)
- Architecture Decisions (what was decided and why)
- Setup Instructions (use discussed approach)
