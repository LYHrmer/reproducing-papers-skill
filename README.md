# reproducing-papers

A portable agent skill that guides an AI coding agent through **reproducing a
research paper's method into an existing codebase**. It works in **both Claude
Code and Codex** — they share the same `SKILL.md` format, so one install serves
either tool.

The skill turns "reproduce this paper" from a guess-and-patch exercise into four
disciplines, in order:

1. **Pin the mechanism** — state in one sentence *what* the method changes and
   *which call site / data structure in your code* it hooks into, and confirm it
   before designing. The obvious interpretation is usually wrong.
2. **Plan with a checklist** — write a plan doc capturing the core invariant,
   design decisions, known trade-offs, and a complete file-by-file checklist
   that becomes your definition of done.
3. **Implement completely** — implement every checklist item; diff your changes
   against the plan before declaring done.
4. **Validate by seeing it** — add visualization / instrumentation scoped to the
   current state so you can confirm the effect on real data and tune parameters.

It also bakes in the recurring traps: filtering the wrong entity, mistaking a
behavior-change for an acceleration, leaving planned items unimplemented, missing
lifecycle/removal paths (state that only grows), and visualizing all-history
instead of the current frame.

## Install

```bash
git clone <repo-url> reproducing-papers
cd reproducing-papers
./scripts/install.sh          # copies into ~/.claude/skills and ~/.codex/skills (whichever exist)
./scripts/install.sh --dev    # symlink instead of copy (for editing the skill in place)
```

Windows:

```powershell
powershell -ExecutionPolicy Bypass -File install.ps1
```

The installer looks for `~/.claude/skills` and `~/.codex/skills`. It installs
into whichever of Claude Code / Codex is present and **skips the one that is
missing** (it reports `skip: ... 不存在`). `--dev` symlinks the repo instead of
copying, so edits to the skill take effect without re-installing.

After installing you may need to **restart the agent**. The agent discovers the
skill by its name and description — once loaded, `reproducing-papers` becomes
available automatically when your request matches (reproduce a paper, implement
an algorithm from a paper, port a method into your framework, "参考 XX 论文添加…").

## What's inside

- `SKILL.md` — the skill itself (tool-neutral; loaded by Claude Code or Codex).
- `templates/plan_template.md` — copyable plan skeleton for step 2: mechanism,
  invariant, design decisions, trade-offs, the implementation checklist, and the
  validation plan.

## Uninstall

```bash
rm -rf ~/.claude/skills/reproducing-papers ~/.codex/skills/reproducing-papers
```
