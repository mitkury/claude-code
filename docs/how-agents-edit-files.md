## How Agents Read and Edit Files

Agents in this environment manipulate the codebase through a set of structured tools instead of relying on free-form shell commands. This keeps edits auditable, minimizes accidental changes, and lets the platform enforce safety rules (for example, respecting `.gitignore` or avoiding destructive operations). The sections below outline the key tools the agent uses to discover files, inspect their contents, and apply changes, along with notes on when shell commands enter the picture.

### Discovering and Inspecting Files
- **`LS`** lists non-hidden files and directories. Agents prefer it to `ls` so the platform can filter entries and enforce ignore globs.
- **`Grep`** wraps `ripgrep` for searching within the workspace while respecting ignore rules. Narrowing by pattern, file glob, or path keeps results concise.
- **`Read`** is the primary way to inspect file contents. It supports reading entire files or selected line ranges and can even preview images. Because each call returns line numbers, the agent can reference exact sections when summarizing or editing.
- **`ReadLints`** surfaces IDE diagnostics so the agent can see lint or type errors associated with files it touches.

### Editing Source Files
- **`ApplyPatch`** is the default editing primitive. The agent provides a patch with explicit context, and the tool enforces a diff-like format (`*** Update File` or `*** Add File` headers, `@@` hunks, and `+/-` lines). This prevents unintended edits and makes it easy to review changes. Creating new files is as simple as supplying an `*** Add File` block.
- **`Edit`** is a structured fallback used only when `ApplyPatch` is unsuitable (for example, when editing tools struggle with a tricky diff). The agent supplies succinct instructions and focused code snippets; the helper model applies the change.
- **`EditNotebook`** handles Jupyter notebooks. It operates at the cell level and ensures notebook JSON remains valid without exposing it directly to the agent.
- **Guidelines**: Agents default to ASCII when editing files, avoid undoing user changes, respect prior formatting, and only modify the minimum necessary code.

### Creating Supporting Artifacts
- Agents can generate additional resources (for example, documentation or scripts) via `ApplyPatch` with an `*** Add File` section. When desired, they can also create directories using a shell command (`mkdir -p`) before adding the file contents.

### When Shell Commands Are Used
- **`Shell`** gives access to a persistent terminal session. Agents use it sparingly—mainly to run commands not covered by dedicated tools (creating directories, running tests, installing dependencies). On the first use, they `cd` into the workspace to set the working directory. Long-running commands can be launched in the background via the `is_background` flag.
- Whenever a dedicated tool exists (for example, `Read` instead of `cat`, `Grep` instead of `rg`), agents must prefer the tool to keep the workflow safe and consistent.

### Typical Editing Workflow
1. **Understand the target**: use `LS`, `Grep`, and `Read` to locate relevant files and gather context.
2. **Plan the modification**: synthesize requirements, sometimes capturing them in `TodoWrite` tasks that the agent tracks during longer efforts.
3. **Apply the change**: craft an `ApplyPatch` diff (or invoke `Edit`/`EditNotebook` if appropriate) to modify or create files.
4. **Verify**: optionally run tests or linters through `Shell`, and check diagnostics with `ReadLints`.
5. **Summarize**: report back on what changed, referencing file paths and key symbols.

### Key Takeaways
- Structured tools (`Read`, `ApplyPatch`, `Edit`, `EditNotebook`) are the backbone of file manipulation; they provide transparency and guardrails.
- Shell usage is intentional and limited to tasks the higher-level tools cannot perform.
- This tooling ecosystem encourages incremental, reviewable edits and keeps the agent aligned with repository conventions while it codes autonomously.
