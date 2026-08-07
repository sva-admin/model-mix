---
name: model-mix
description: Two-direction model tiering. Advisor pattern (a cheap executor escalates hard decisions UP to one frontier-tier advisor) and Orchestrator pattern (the frontier model plans and fans work DOWN to parallel Sonnet workers). Trigger on "advisor pattern", "orchestrator pattern", "model mix", "escalate to opus", "fan out to sonnet workers", "use cheap workers", "save my usage limits", "model tiering", or when the user asks how to run a big task without burning their session limits. Also self-apply silently on qualifying tasks (see When to self-apply). Not for choosing which model to chat with. That's the user's /model choice.
---

# Model Mix: Advisor up / Orchestrator down

## Objective

Spend expensive-model turns only where they change the outcome. The expensive model **plans and decides**; the cheap model **reads and executes**. Two directions, same primitive (subagents with a `model` override):

- **Advisor (escalate up):** session runs on a cheap model, then on a genuinely hard decision it spawns ONE frontier-tier advisor subagent, the advisor returns a plan, and the cheap loop keeps executing. (Benchmark shape: ~92% of frontier quality at ~63% of cost on SWE-bench Pro.)
- **Orchestrator (delegate down):** session runs on the frontier model, read-heavy or parallelizable work fans out to Sonnet workers with isolated contexts, and the frontier model synthesizes and never touches raw pages/files on fan-out tasks. (Benchmark shape: ~96% of quality at ~46% of cost on BrowseComp.)

On a subscription plan the currency is **usage-limit headroom and wall-clock time**, not dollars. Same logic, same wins.

## Detect the direction (first step, every invocation)

Check which model this session runs on (you know your own model).

| Session model | Direction | Move |
|---|---|---|
| Frontier tier (Opus class) | **Orchestrator down** | Fan reads/execution out to `sonnet` (or `haiku` for mechanical) workers |
| Sonnet / Haiku | **Advisor up** | Escalate stuck decisions to one `opus` advisor call |

## Advisor pattern (escalate up)

**Escalate only at a hard decision point.** All of these qualify, nothing else does:
- Two failed fix attempts on the same bug (never grind a third time blind)
- An architectural or hard-to-reverse choice (schema, API contract, framework)
- Security-sensitive design
- Requirements ambiguity that judgment, not information, must resolve
- A tie between two plans you can't break with evidence

**Mechanics:** one subagent call, `model: "opus"`, run in the foreground. The prompt is a tight context package, not a transcript dump:

```
ROLE: You are the advisor. Return a decision + plan. Do NOT execute anything.
OBJECTIVE: <what done looks like, 1-2 sentences>
CONSTRAINTS: <hard limits: stack, contracts, perf, security>
TRIED: <attempt -> result, one line each>
QUESTION: <the single decision you need made>
RETURN: chosen option, why, step plan, and the risk you're accepting.
```

**Rules:** one escalation per stuck-point (if the advice fails, gather new evidence before re-escalating with it). Advisor advises; the main loop executes. Note in your reply that you escalated: the user should see where the expensive call went.

## Orchestrator pattern (delegate down)

**Fan out when** the task is read-heavy or splits into 3 or more independent units: multi-source research, per-competitor/per-file/per-page analysis, repo audits, bulk transformations, QA sweeps. **Don't fan out** single-file edits, sequential logic, or anything needing one coherent context.

**Mechanics:**
- 5 units or fewer: parallel subagent calls, `model: "sonnet"` (`haiku` for purely mechanical extraction).
- More than 5 units, or multi-stage (find, verify, synthesize): a workflow/orchestration tool with `model: 'sonnet'` per agent (add `effort: 'low'` for mechanical stages). Respect whatever opt-in rules that tool defines.
- Worker prompts must be self-contained: the worker knows nothing about this conversation. Include exact inputs, the questions to answer, and the return format (use structured output when a schema is available).
- The orchestrator does not read the raw sources itself on fan-out tasks: it plans, delegates, synthesizes, and verifies the merged result.
- If you already have research subagents installed (search-specialist, research-analyst, competitive-analyst, trend-analyst, and similar Sonnet-tier agents), prefer them: they are pre-built workers.

## When to self-apply (no user prompt needed)

- Session is frontier-tier AND the task would have you serially reading 3+ independent sources/files, so fan out and say so in one line.
- Session is on Sonnet AND you hit a qualifying hard decision, so escalate once and say so in one line.
- The user asks for something big while worried about limits, so propose the split explicitly before running.

## Done looks like / verify

- The expensive model's turns are visibly plan/decide/synthesize; reads and retries happened on the cheap tier.
- Every worker result actually got used or explicitly discarded: no silent drops.
- The reply names the pattern used and the model split (for example "fanned 6 Sonnet workers, synthesized here").

## Boundaries

- `/model` (the session's main model) is the user's choice: never switch it yourself; this skill only controls **subagent** models.
- A cross-vendor second opinion (a reviewer running on a different model family) is a different move from the Advisor here, which is same-family escalation with full context. Both can coexist; prefer the advisor for plan decisions and the cross-vendor reviewer for adversarial review.
- Don't stack patterns pointlessly: a frontier-tier session doesn't need a frontier advisor; a Sonnet session shouldn't orchestrate Sonnet workers for depth it lacks. Escalate instead.
