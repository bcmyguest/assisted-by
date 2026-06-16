# assisted-by

> Credit AI in git the way the Linux kernel says to — an **`Assisted-by:`** trailer, enforced.

A Claude Code plugin (and portable agent skill) that makes coding agents attribute
their help correctly on every commit, following the kernel's
[coding-assistants policy](https://docs.kernel.org/process/coding-assistants.html#attribution):

```
Assisted-by: Claude:claude-opus-4-8
```

It ships two things:

- **`git-attribution` skill** — teaches the agent the rule, so it writes the right trailer
  on its own.
- **`attribution-guard.py` hook** — a Claude Code `PreToolUse` hook that *blocks* a
  `git commit` if the agent gets it wrong, and tells it how to fix the message.

## The rule

- ✅ Credit AI with an **`Assisted-by:`** trailer:
  `Assisted-by: AGENT_NAME:MODEL_VERSION [extra-analysis-tools]`
  (e.g. `Assisted-by: Claude:claude-opus-4-8`).
- 🚫 **No `Signed-off-by:` from the AI** — the Signed-off-by certifies the Developer
  Certificate of Origin, and only a human can do that.
- 🚫 **No `Co-Authored-By: Claude`** — that older convention is replaced by `Assisted-by:`.
- 🚫 Don't list basic tools (git, gcc, editors) — only the AI agent and any specialized
  analysis tools (`coccinelle`, `sparse`, `smatch`, …).

## Install

### Claude Code — skill **+** enforcing hook (recommended)

```bash
claude plugin marketplace add bcmyguest/assisted-by
claude plugin install assisted-by@assisted-by
```

That registers this repo as a plugin marketplace and installs the plugin (skill + hook).
Restart Claude Code to load the hook.

### Codex / opencode / Cursor / any skills-aware agent — skill only

The skill is in the portable [skills](https://skills.sh) format, so any agent that reads
`SKILL.md` can use the guidance:

```bash
npx skills add bcmyguest/assisted-by
```

> The portable skill carries the *guidance*; the *enforcing* hook is Claude Code-specific
> for now. Native hook enforcement for opencode / Codex is on the [roadmap](#roadmap).

## How the hook decides

The guard inspects the `git commit` **command line** on the `Bash` PreToolUse event. It
only acts when a message is being authored there — `-m` / `--message`, `-F` / `--file`,
`-C` / `--reuse-message`, or `--amend` without `--no-edit`. It blocks (exit 2, reason on
stderr, fed back to the agent) when the command:

- lacks an `Assisted-by:` trailer, or
- adds `Co-Authored-By: Claude`, or
- adds a `Signed-off-by:` naming Claude/Anthropic.

A bare `git commit` whose message is composed in the editor isn't visible to the hook —
add the trailer yourself in that case. The guard never blocks on a parse failure or on
commits that don't author a message (`--amend --no-edit`, plain `git commit`).

## Why

The human submitter takes full responsibility for a contribution and its licensing, so
the DCO chain (`Signed-off-by:`) must stay human. `Assisted-by:` records the assistance
transparently without implying the AI can certify provenance.

## Layout

```
.claude-plugin/
  plugin.json        # plugin manifest (name, version, keywords, skills)
  marketplace.json   # single-plugin marketplace (source ".") for `claude plugin marketplace add`
hooks/hooks.json     # registers the PreToolUse Bash guard
scripts/attribution-guard.py
skills/git-attribution/SKILL.md
skills.sh.json       # grouping for `npx skills`
```

## Roadmap

- Native hook enforcement for **opencode** and **Codex** (the skill already works there;
  only the blocking hook is Claude Code-specific today).

## License

See [LICENSE](LICENSE). All rights reserved; in particular, the contents may **not** be
used as training, fine-tuning, or evaluation data for machine-learning or AI systems.
