---
name: luna-delegation
description: "Consent-gated workflow for delegating bounded, low-complexity subtasks to GPT-5.6 Luna at max reasoning while the primary agent owns scope, verification, judgment, and delivery. Invoke immediately without reconfirmation when the user names `$luna-delegation`, selects the skill entry, or explicitly says `使用 Luna 委派技能`. When the primary agent proposes using this skill without such an explicit request, first obtain affirmative, task-specific user consent and invoke it only after consent. Ordinary simplicity, repetition, summaries, retrieval, parallelism, or efficiency requests never count as consent."
---

# Luna Delegation

Keep the primary agent responsible for understanding the complete request, choosing
scope, resolving ambiguity, verifying results, making judgments, and delivering the
final answer. Delegate only a useful, independent, low-complexity part.

## Confirm authorization

Treat an explicit request as authorization for the current task. If the user names
`$luna-delegation`, selects its skill entry, or says `使用 Luna 委派技能`, use the
workflow without asking for confirmation again.

When the user has not explicitly requested this skill, the primary agent may propose
it only when delegation would provide a real benefit. Before invoking the workflow or
creating any Luna worker, ask one concise consent question, explain the proposed
subtask and that the primary agent will verify it, and wait for an affirmative answer.
Silence, ambiguity, or a general request for speed or parallel work is not consent.

Make consent task-specific. Do not carry it into later tasks or materially expanded
scope. If the user declines, continue without this skill and do not ask again for the
same scope.

## Decide whether to delegate

Delegate bounded work such as:

- Locate files, search keywords, and screen materials.
- Produce preliminary webpage or document summaries.
- Classify, deduplicate, format-check, or organize lists.
- Extract simple data into a specified structure.
- Summarize logs and flag obvious errors for review.
- Perform low-risk repetition with directly verifiable output.
- Implement a small code change whose files, interfaces, and acceptance checks are
  already fully specified by the primary agent.

Do the work directly when it is a one-step task and delegation overhead would exceed
the benefit. Never invent a subtask merely to use this skill.

Keep these matters with the primary agent; never ask Luna for the final judgment:

- Legal, financial, disclosure, or compliance conclusions.
- Final verification of important facts.
- High-risk or irreversible operations.
- Deletion, overwriting, publication, commits, pushes, or external messages.
- Architecture, scope definition, or material tradeoffs.
- Ambiguous work or work requiring coordination across tasks.
- Any part the user explicitly asks the primary agent to perform personally.

## Spawn a leaf Luna worker

Use the native `spawn_agent` capability for every execution worker and explicitly set:

```yaml
fork_turns: "none"
model: "gpt-5.6-luna"
reasoning_effort: "max"
```

Use a unique, descriptive `task_name` and provide the complete task in `message`. Do
not depend on inherited conversation context or a custom agent TOML. If the exact
model or reasoning effort is unavailable or rejected, stop that delegation and report
the limitation; do not substitute another model.

Make Luna a leaf worker. Include this instruction in every worker message:

> Act as a leaf execution agent. Do not delegate, spawn subagents, call delegation
> tools, or coordinate other agents. Complete the bounded task directly. If ambiguity,
> risk, a scope conflict, or unverifiable output prevents completion, stop and return
> the issue to the primary agent without widening scope.

## Write a complete task packet

Give every Luna worker all six sections below:

```text
GOAL
<One observable outcome.>

INPUT OR CHECK RANGE
<Exact files, URLs, records, pages, commands, or owned code paths.>

BOUNDARIES
<Excluded scope, settled decisions, allowed mutations, preserved interfaces, and the
leaf-worker instruction.>

REQUIRED EVIDENCE
<Citations, file paths, line references, command output, counts, or diff evidence.>

RETURN FORMAT
<Exact headings, fields, table columns, or schema.>

ACCEPTANCE CRITERIA
<Concrete conditions the primary agent can independently verify.>
```

For code changes, assign exact non-overlapping files, state all interfaces and
verification commands, and require a file-by-file change report. Preserve user and
concurrent edits. Parallelize only genuinely independent subtasks; never let workers
modify the same file or cover overlapping scope.

## Verify and deliver

Treat Luna's report as unverified claims. Inspect the cited evidence and actual changed
files, rerun proportionate checks, confirm scope discipline, and resolve discrepancies
before using the result. The primary agent must make every final fact, legal, financial,
disclosure, compliance, architecture, risk, and acceptance judgment.

Do not automatically escalate to Terra or request a fresh Sol review. Use another
model or review lane only when the user separately requests it in the current task.

In the final answer, briefly identify what was delegated to Luna and what the primary
agent independently verified. Report unresolved limitations instead of hiding them.

## Explicit-request examples: use without reconfirmation

- `使用 $luna-delegation，先让 Luna 定位仓库中所有旧接口引用，再由你核验并修改。`
- `使用 Luna 委派技能，把这批日志的明显错误按模块归类，主代理复核后给我结论。`
- `我已选择 Luna Delegation 技能入口；请并行初筛三组互不重叠的材料，并由你汇总。`

## No-authorization examples: do not invoke yet

- `请快速总结这个文档。` — Do not invoke merely because summarization is suitable;
  ask for consent first only if delegation would materially help.
- `并行搜索仓库中的重复代码，尽量提高效率。` — Parallelism is not consent; ask
  before using this skill.
- `这个任务很简单，把表格中的公司名称去重。` — Complete it directly when
  delegation overhead is higher; do not invoke or seek consent merely to use Luna.
