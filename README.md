# ClaudeAgents

Claude sub-agent benchmarks: optimizing sub-agent utilization and running sub-agents **asynchronously**
with an orchestrator, measured across every Claude Gen 5 model and effort level.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Thazzeus-ML/ClaudeAgents/blob/claude/dazzling-mayer-5r8o68/notebooks/gen5_subagent_benchmark.ipynb)

## What is in here

| Path | Purpose |
|---|---|
| `notebooks/gen5_subagent_benchmark.ipynb` | The lab notebook: setup, model × effort sweep, async sub-agent orchestrator, findings |
| `results/<run_id>/` | Per-run outputs (`sweep.jsonl`, `orchestrator.jsonl`, CSV summaries). Commit the runs you want to keep |

## Running it

1. Open the notebook in Colab with the badge above.
2. Add your key as a Colab secret named `ANTHROPIC_API_KEY` (Colab sidebar → Secrets). The notebook falls back to a prompt.
3. Run Part 1. The smoke test makes one Haiku call.
4. In Part 2 keep `QUICK = True` for the first pass, read the cost estimate, then set `GO = True`.
5. In Part 3 set `GO_ORCH = True` to compare **solo**, **blocking**, and **async** delegation on the fan-out task.
6. Part 4 produces the per-category recommendation table and effort curves. Record conclusions in the findings log with the run id.

## Models covered

Fable 5.1, Opus 5.5, Sonnet 5.5, Haiku 5.5 are enabled by default. Fable 5, Opus 5 and Sonnet 5 are in the
registry and can be switched on with `enabled=True`. Effort levels: `low`, `medium`, `high`, `xhigh`, `max`.

## Harness rules

The harness follows the current Gen 5 API contract: adaptive thinking is always on and controlled only through
`output_config.effort`; no `budget_tokens` or sampling parameters; no forced `tool_choice`; strict tool schemas;
append-only histories with assistant content echoed back unchanged; refusal handling with server-side fallbacks on
by default; usage, latency, served model and cost recorded on every call.

## Async sub-agent pattern

The orchestrator gets three client-side tools: `spawn_subagent` (returns immediately with a handle),
`check_subagents` (non-blocking status) and `wait_for_subagents`. Finished results are pushed to the lead in the
next user message whether or not it asked, so the lead can keep working while sub-agents run.
