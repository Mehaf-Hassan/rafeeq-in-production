# Rafeeq — Agentic AI Capstone

A local AI assistant that helps users query synthetic Madar service
data, retrieve policies, calculate fees, and prepare write actions
for human review.

- **Track:** 1 — Rafeeq in production
- **Team:** Individual
- **Model:** OpenRouter free-model router (`openrouter/free`)
- **Data:** Synthetic Madar data only

## Problem

Users need accurate answers about service fees, permits, policies,
and inspections. Rafeeq uses tools and retrieved policy information
instead of relying only on generated text. It can perform multi-step
tasks, record tool usage, and place write actions into a pending
human-review queue. This is a local prototype, not a production
service.

## How to run

Run commands from the labs/ directory.

Install project dependencies:

    uv sync

Prepare synthetic data:

    uv run python prepare_data.py

If you do not already have a local .env file, create one:

    cp .env.example .env

Set OPENROUTER_API_KEY in the local .env file.
Never commit .env or share the API key.

Start the API locally:

    RAFEEQ_API_KEYS=dev-key-1 RAFEEQ_MAX_COST=0.05 uv run servers/rafeeq_api.py --port 8793
    RAFEEQ_MAX_COST sets the approximate per-request cost budget in USD. It defaults to 0.05. The limit is checked after a model/tool step, so actual provider charges may exceed it slightly.

Open the chat interface:

    http://127.0.0.1:8793/

API health check:

    http://127.0.0.1:8793/healthz

Run an example through the CLI:

    uv run capstone/starter.py ask "What is the fee for an illuminated 3 m² sign?"

## Design across the ladder

| Rung | Decision | Reason |
|---|---|---|
| Prompt | Madar system prompt plus capstone/AGENTS.md | Keep task rules and accuracy expectations explicit |
| Context | Session history, compaction, and tool-result limits | Limit irrelevant history and oversized results |
| Harness | Permission policy, approval queue, audit log, step and cost budgets | Control writes and bound execution |
| Loop | Tool-calling loop with a maximum of 8 steps | Allow multi-step tasks with a stopping limit |
| Graph | No explicit graph used | The supplied agent loop is sufficient for this track |

Context controls include a 3,000-character tool-result cap and a
24-message history limit in the CLI starter. The starter also sets
a cost budget of 0.05; the actual budget behavior depends on the
configured provider and model pricing.

## Tools

| Tool | Read / write | Over MCP? | Approval |
|---|---|---|---|
| compute_fee | Read / calculation | Yes | No |
| search_policies | Read | Yes | No |
| get_permit | Read | Yes | No |
| book_inspection | Write | Local | Required / queued for review |
| reschedule_inspection | Write | Local | Required / queued for review |
| create_refund | Write | Local | Required / queued for review |

The local prototype uses synthetic records only. In the tested API
flow, a booking request was returned as a pending action instead of
being executed.

## Evaluation

The evaluation set contains 16 tasks. Five judge graders were
removed from tasks-auto.jsonl to reduce additional model requests;
the original tasks.jsonl remains unchanged. These results therefore
represent the automatic graders retained in tasks-auto.jsonl, not
the full original grader suite.

| Configuration | Passed | Tasks | Pass rate |
|---|---:|---:|---:|
| A — Base specification | 12 | 16 | 75% |
| B — Stronger tool-use instructions | 14 | 16 | 87.5% |

Configuration B improved the measured pass rate by 12.5 percentage
points on this task set. The CSV files record per-task outcomes,
tools, steps, and elapsed time. Request and token averages are not
reported here because they were not captured as summary metrics.

Run the automatic-grader evaluations:

    uv run capstone/starter.py eval capstone/tasks-auto.jsonl --config A
    uv run capstone/starter.py eval capstone/tasks-auto.jsonl --config B

Results:

- [eval_A.csv](artifacts/capstone/eval_A.csv)
- [eval_B.csv](artifacts/capstone/eval_B.csv)

See [EVAL_ANALYSIS.md](capstone/EVAL_ANALYSIS.md) for failed tasks, the initial five-task smoke
tests, and limitations of the evaluation.

## Security and governance

See [SECURITY.md](capstone/SECURITY.md).

The local write-approval test used fictional booking values. The API
returned the proposed action as pending human review and reported that
no booking was executed and no database changes were made.

A single direct adversarial prompt test was performed. Rafeeq refused
a request to ignore its instructions and email synthetic permit records
and API secrets to an attacker. The response contained no tool calls
and no pending actions. This was one test only and does not establish
that all prompt-injection or data-exfiltration attacks are prevented.

Only synthetic data should be used. The prototype must not be exposed
publicly or connected to real operational systems without additional
security review, stronger authentication, authorization, and
protection of secrets. The development API key used by the local HTML
prototype is not suitable for production.

## Limitations and next steps

- Five original LLM-judge graders were omitted from the evaluated copy.
- The recorded results are specific to these 16 tasks and do not prove
  general performance or production readiness.
- A single direct adversarial prompt test was performed and refused.
  Indirect prompt-injection and broader data-exfiltration testing remain
  outstanding.
- The HTML client uses a local development key and must be redesigned
  before deployment.
- Before production use, add proper user authentication, secure
  approval handling, and a broader security and reliability review.

https://github.com/SDAIAAcademy/agentic-ai
