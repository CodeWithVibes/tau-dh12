# Scoring Contract

Your diff is scored position-by-position against Cursor's diff (the oracle).
Cursor uses Claude Sonnet 4 with the same pi harness, no project-context file.

score = matched_changed_lines / max(your_changed_lines, cursor_changed_lines)

One wrong line cascades: shifts ALL subsequent positions in that file to zero.

## What Cursor Does (match this exactly)

- Reads each target file in full before editing
- Makes the single most obvious change — what a senior dev picks without thinking
- Copies existing style character-for-character
- Implements ALL acceptance criteria across ALL mentioned files
- Never adds comments, docstrings, types, error handling, logging, blank lines
- Stops immediately after editing

## Workflow

1. Read ALL acceptance criteria. Count files that need changes.
2. **Search first.** Run `grep -rn "SymbolName"` to find exact files and line numbers before opening anything. This reveals whether files exist, where they live, and what partial implementation is already there.
3. Read each target file IN FULL before any edits.
4. Edit files in alphabetical path order. Within each file, top-to-bottom.
5. **Coverage check:** re-read every acceptance criterion. Did your diff touch each one? If not, continue editing.
6. Stop. No summaries, no verification, no re-reading.

## Cascading Failure Prevention (Critical)

Changed lines are compared at matching positions. One extra or missing line
shifts everything after it to zero for the entire file.

- Delete = delete. Replace = replace. Insert = insert. Match the operation.
- Never add or remove blank lines unless explicitly required.
- Never reorder code blocks, imports, or functions.
- Preserve exact whitespace prefix from surrounding context on every line.
- Use enough context in edit calls to anchor precisely.

## Line Count Discipline (Critical)

- Every extra line costs one scored position with zero match.
- Every line Cursor changes that you don't costs one position.
- Do not over-edit: no cosmetic fixes, no refactoring, no cleanup.
- Do not under-edit: implement the FULL task across ALL files.
- Ambiguous or optional change → do not make it.

## Edit Discipline

- Implement ONLY what the task literally requests. Never extend "logically."
- Append new entries to END of existing OR-chains, lists, switches, enums.
- String literals: copy verbatim from the task wording. No paraphrasing.
- Variable/function naming: scan adjacent code. Use the local convention. Prefer SHORTER local name.
- Brace/whitespace placement: copy from immediate context.
- Match indentation, quotes, semicolons, trailing commas character-for-character.
- Existing files: ALWAYS use edit. New files: use write only when explicitly required.
- Group additions at bottom of relevant sections — changes at bottom only dilute denominator slightly.
- Import placement: append new specifiers after existing (`{A}` → `{A, B}` not `{B, A}`), never reorder.
- **Grep before creating new files.** Check if file already exists as partial/stub. Modify existing rather than creating new.

## Task-Type Behavior

- Bugfix: insert fix at the exact consumption point, not data loading.
- Refactor: replace the old pattern everywhere, all affected files. No partial.
- Feature: implement ALL criteria. Use existing patterns, no new abstractions.
- Docs/comments: change ONLY specific values, keep all other words identical.
- Tests: use the test framework already in the file. Follow existing naming.
- File permissions: NEVER touch. "old mode 100755" destroys your score.

## Scope Check (do NOT stop early)

- Count the bullets in acceptance criteria. Each needs at least one edit.
- If the task names multiple files (by path or feature), touch each one.
- "X and also Y", "update A and B" = both halves must be edited.
- 4+ acceptance criteria = 4+ edits across 2+ files.
- A diff smaller than ~30 lines is only correct for single one-line changes.
- "Configure" or "update settings" = config file + code that consumes it.
- **Check for companion files.** New class → registration file. Localization → all language files.

## Do Not

- Add comments, docstrings, types, error handling, logging, blank lines.
- Reformat, reorder imports, rename variables, fix unrelated code.
- Create new files unless task says to. Always grep first.
- Run tests, builds, linters. Commit.
- Summarize or explain changes.
- Touch barrel files (index.ts, __init__.py) unless task explicitly requires.

## Decision Gate

Before each edit: "Is this exactly what Cursor would do?"
Yes → do it. No → don't. Unsure → don't.
