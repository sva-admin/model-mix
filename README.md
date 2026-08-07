<p align="center"><a href="https://loop.sv-academy.org"><img src="https://raw.githubusercontent.com/sva-admin/claude-skills/main/assets/sv-academy-512.png" width="88" alt="Silicon Valley Academy"/></a></p>

# model-mix

Two-direction model tiering for agent workflows: escalate hard decisions up to one expensive advisor, and fan read-heavy work down to parallel cheap workers.

An original skill by Silicon Valley Academy.

## What it does

- **Advisor (escalate up).** A cheap session hits a genuinely hard decision (two failed fixes, an architectural choice, a security-sensitive design) and spawns exactly one frontier-tier advisor to return a plan. The cheap loop keeps executing.
- **Orchestrator (delegate down).** A frontier-tier session stops reading raw sources itself. Read-heavy or parallelizable work fans out to Sonnet workers with isolated contexts, and the expensive model only plans, synthesizes, and verifies.
- **Self-applying.** The skill detects which direction fits the current session model and acts, instead of waiting to be asked.

The point is simple: spend expensive-model turns only where they change the outcome, and protect your usage limits on everything else.

## Install

Paste this to Claude Code:

```
Install the model-mix skill from github.com/sva-admin/model-mix
```

## Learn free at https://loop.sv-academy.org/tutorials

Free tutorials on building with Claude Code: skills, subagents, and the workflows that hold them together.

## License

MIT. See [LICENSE](LICENSE).

---

[All SV Academy Claude skills](https://github.com/sva-admin/claude-skills) | [sv-academy.org](https://sv-academy.org)
