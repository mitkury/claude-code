## How `ApplyPatch` Works

`ApplyPatch` is the primary editing primitive for agents in this environment. It lets the agent propose precise, reviewable diffs that the platform applies safely. This document explains the structure of an `ApplyPatch` request, shows how different editing scenarios map onto the tool, and highlights best practices.

### Core Concepts
- **Single-file focus**: each `ApplyPatch` call targets exactly one file. Multiple files require multiple invocations.
- **Diff envelope**: every request starts with `*** Begin Patch` and ends with `*** End Patch`, framing the change for auditing.
- **Operation header**: inside the envelope, the agent chooses either `*** Update File: <path>` (modify existing) or `*** Add File: <path>` (create new). Paths are absolute.
- **Hunks with context**: modifications are expressed with `@@` headers followed by context lines (` `), removals (`-`), and additions (`+`). Supplying at least three lines of surrounding context keeps the diff unambiguous.
- **Text-only**: the tool expects ASCII-friendly text. Binary assets should be handled with other mechanisms.

### Update Example
The snippet below shows a real-world pattern for appending a tip to `docs/how-agents-edit-files.md`:

```diff
*** Begin Patch
*** Update File: /workspace/docs/how-agents-edit-files.md
@@
- ### Key Takeaways
- - Structured tools (`Read`, `ApplyPatch`, `Edit`, `EditNotebook`) are the backbone of file manipulation; they provide transparency and guardrails.
- - Shell usage is intentional and limited to tasks the higher-level tools cannot perform.
- - This tooling ecosystem encourages incremental, reviewable edits and keeps the agent aligned with repository conventions while it codes autonomously.
+ ### Key Takeaways
+ - Structured tools (`Read`, `ApplyPatch`, `Edit`, `EditNotebook`) are the backbone of file manipulation; they provide transparency and guardrails.
+ - Shell usage is intentional and limited to tasks the higher-level tools cannot perform.
+ - This tooling ecosystem encourages incremental, reviewable edits and keeps the agent aligned with repository conventions while it codes autonomously.
+ - Prefer `ApplyPatch` for source edits; fall back to other tools only when necessary.
*** End Patch
```

### Creating New Files
To add new documentation or code, the agent switches to the `*** Add File` header. For example, the command that created this document looked like:

```diff
*** Begin Patch
*** Add File: /workspace/docs/how-applypatch-works.md
+## How `ApplyPatch` Works
+...
*** End Patch
```

### Best Practices
- **Preview the current file**: read relevant sections with `Read` before crafting a patch to avoid stale context.
- **Keep changes minimal**: only touch the lines that need adjustment. Unnecessary context edits can introduce merge conflicts.
- **Maintain formatting**: respect existing indentation, line endings, and character sets.
- **Validate after editing**: run tests or linters (via `Shell`) if the change warrants verification, and review `ReadLints` diagnostics when available.
- **Fallbacks**: when `ApplyPatch` cannot accurately express the desired edit (for example, due to conflicting context), agents elevate to the `Edit` tool with focused instructions.

Using `ApplyPatch` consistently ensures edits remain explicit, reviewable, and easy to reason about during collaborative development.
