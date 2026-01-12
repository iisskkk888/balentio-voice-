# Project Context

**Balentio Voice** is a commercial SaaS product (NOT open-source). It consists of:
- Desktop app (macOS/Windows) - Tauri + Svelte 5
- Backend API - Hono + PostgreSQL + Supabase
- Database - Drizzle ORM

This is a **private repository** for commercial development.

# Planning with specs/

For **complex features or phases** that need detailed planning, create a spec file:
- Format: `specs/DD.MM.YY - [feature-name].md` (e.g., `specs/15.01.26 - Phase 1 Backend Foundation.md`)
- Include: problem description, solution, implementation plan, testing strategy
- NOT needed for simple tasks (small bug fixes, minor UI tweaks)

During implementation, follow domain-specific rules from `rules/*.md`:
- Backend work → load `@rules/backend.md`, `@rules/database.md`, `@rules/security.md`
- Desktop work → load `@rules/typescript.md`, `@rules/svelte.md`, `@rules/styling.md`
- Git operations → follow `@rules/git.md`

# Standard Workflow

## For Complex Features (requiring spec):

1. **Think through the problem**: Read codebase for relevant files and context
2. **Write a plan**: Create `specs/DD.MM.YY - [feature-name].md` with:
   - Problem description
   - Solution and approach
   - List of relevant files to read
   - Implementation plan with todo items
   - Testing strategy
   - Acceptance criteria
3. **Get approval**: Check in before starting work
4. **Execute**: Work on todo items, mark complete as you go
5. **Communicate**: Give high-level explanations of changes at each step
6. **Simplicity first**: Make every change as simple as possible, impact minimal code
7. **Review**: Add review section to spec with summary of changes and learnings

## For Simple Tasks (no spec needed):

1. Read relevant code and rules
2. Make minimal, focused changes
3. Follow domain-specific guidelines
4. Commit with conventional commit messages

# Expensive/destructive actions

1. Always get prior approval before performing expensive/destructive actions (tool calls).
   - Expensive actions require extended time to complete. Examples: test, build.
     - Why: Unexpected tests/builds just waste time and tokens. The test results are often innaccurate ("It works!" when it doesn't.)
   - Destructive actions result in permanant changes to project files. Examples: commit to git, push changes, edit a GitHub PR description.
      - Why: changes should be verified before adding to permanent project history. Often additional changes are needed.
2. Instead, you may automatically show a plan for the tool call you would like to make.
   - Commit messages should follow the conventional commits specification.
3. Then either the plan will be explicitly approved or changes to the plan will be requested.
4. Unless otherwise stated, any approval applies only to the plan directly before it. So any future action will require a new plan with associated approval.

# Human-Readable Control Flow

When refactoring complex control flow, mirror natural human reasoning patterns:

1. **Ask the human question first**: "Can I use what I already have?" → early return for happy path
2. **Assess the situation**: "What's my current state and what do I need to do?" → clear, mutually exclusive conditions
3. **Take action**: "Get what I need" → consolidated logic at the end
4. **Use natural language variables**: `canReuseCurrentSession`, `isSameSettings`, `hasNoSession`: names that read like thoughts
5. **Avoid artificial constructs**: No nested conditions that don't match how humans actually think through problems

Transform this: nested conditionals with duplicated logic
Into this: linear flow that mirrors human decision-making

# Honesty

Be brutally honest, don't be a yes man.
If I am wrong, point it out bluntly.
I need honest feedback on my code.

# Spec Placement

Specs always live at the root level of their scope (not inside `docs/`):

- **`/specs/`** - Cross-cutting features, architecture decisions, phases
- **`/apps/[app]/specs/`** - Features specific to one app only
- **`/packages/[pkg]/specs/`** - Package-specific implementation details

**For Balentio Voice**: Most specs go in `/specs/` since features involve both Desktop + Backend.

## When to Create a Spec

**Create a spec when:**
- Starting a new development phase (Phase 1, Phase 2, etc.)
- Building a complex feature spanning multiple files/services
- Making architectural decisions
- Need to coordinate Desktop ↔ Backend integration

**Skip spec for:**
- Small bug fixes
- Minor UI adjustments
- Simple refactoring
- Adding a single utility function

# Script Commands

The monorepo uses consistent script naming conventions:

| Command | Purpose | When to use |
|---------|---------|-------------|
| `bun format` | **Fix** formatting (biome + prettier) | Development |
| `bun format:check` | Check formatting | CI |
| `bun lint` | **Fix** lint issues (eslint + biome) | Development |
| `bun lint:check` | Check lint issues | CI |
| `bun check` | Type checking (tsc, svelte-check, astro check) | Both |

**Convention:**
- No suffix = **fix** (modifies files)
- `:check` suffix = check only (for CI, no modifications)
- `check` alone = type checking (separate concern, cannot auto-fix)

**After completing code changes**, run type checking to verify:
```bash
bun check
```

This runs `turbo run check` which executes the `check` script in each package (e.g., `tsc --noEmit`, `svelte-check`).
