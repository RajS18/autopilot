<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## Documentation Requirement

For every feature branch that completes a task, create branch-dependent documentation in the `docs/` directory:

**File naming:** `docs/{branch-name}.md` (e.g., `docs/feature-project-auth-setup.md`)

**Document Structure:**
1. **Requirement** — What was asked to be done and why
2. **What Was Added** — List all new files created and files modified
3. **How It Was Added** — Terminal commands executed and exact file changes with context
4. **How It Was Tested** — Testing methodology, commands run, and results
5. **Expected Results** — What should work after implementation
6. **Status** — Completion status
7. **Next Steps** — Optional enhancements or follow-up work

**Purpose:** These documents serve as learning references and decision records for why certain steps were performed and how. If the branch changes, create a new documentation file rather than modifying existing ones.

**When to Create:** Upon task completion, before or after committing changes.

# Database types

Derive database types from the Drizzle schema — never hand-write custom or partial
shapes for table rows. Export `typeof table.$inferSelect` (and `$inferInsert` when
needed) from `lib/schema.ts` and import it. When a consumer needs only some
columns, narrow with `Pick<Row, ...>` / `Omit<Row, ...>` rather than redeclaring a
literal type. Don't add an insert type where `db.insert(...).values()` already
enforces the shape.