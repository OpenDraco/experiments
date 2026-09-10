# OpenDraco — Experiments

The raw experimental record behind the OpenDraco tool-demonstration paper: for every
run, the notebook that launched it, the per-instance inference logs, the predictions,
and the SWE-bench harness report, plus a generated summary of all runs in
[`EXPERIMENT.md`](EXPERIMENT.md).

Nothing here is post-processed. The files are exactly what the runs produced, so any
number in the paper can be traced back to the log or report it came from.

## A note on the name: EvoMas → OpenDraco

These experiments were run in June 2026, while the tool was still called **EvoMas**.
It was renamed **OpenDraco** afterwards, and the paper uses the new name.

Identifiers inside the data still say EvoMas:

| where | example |
|---|---|
| instance ids | `custom-EvoMas-evomas-instance-trivial-18757fd` |
| logger names | `evomas.core.workflow.graph_builder`, `evomas.mcp.server` |
| model field in reports | `evomas-notebook` |
| workspace paths in logs | `…/Temp/evomas_workspace/…` |

**They are left untouched on purpose.** The instance ids are the join key between a
prediction and the harness report that scored it; rewriting them would break that link,
and with it the traceability from the paper's tables back to this evidence. Read
*EvoMas* as *OpenDraco* throughout.

## Layout

```
notebook-<configuration>-<model>-<instance-set>/
  <name>.ipynb                the run itself: config snapshot + live output
  predictions/*.jsonl         one prediction per instance (incl. model_patch)
  predictions/logs/*.log      per-instance inference log
  evaluations/*.json          SWE-bench harness report (completed_ids, resolved_ids)
EXPERIMENT.md                 generated summary of every run
GENERATION_PROMPT.md          the prompt the five synthetic instances came from
generate_report.py            regenerates EXPERIMENT.md
split_notebook_run.py         splits one notebook run into the per-instance files
```

### Reading a run name

`notebook-chain-9b-5custom` = configuration `chain`, model `qwen3.5:9b`, five synthetic
instances.

- **Instance sets** — `5custom` five synthetic instances spanning five difficulty
  tiers, each in its own `opendraco-instance-*` repository and generated from the
  prompt in [`GENERATION_PROMPT.md`](GENERATION_PROMPT.md); `23lite` the SWE-bench
  Lite `dev` subset; `77litetest` a 77-instance subset of the Lite test split (gold
  patch under 600 characters).
- **Configurations** — `chain` (our baseline) plus `agentscope_hybrid`,
  `experepair_star`, `hyperagent_star`, `joycode_star`, `lingxi_star`,
  `openhands_star`, `prometheus_tree`.
- **Model variants** — `2b`, `4b`, `9b` are `qwen3.5`; `coder3b`, `coder7b` are
  `qwen2.5-coder`.
- **Repetition study** — `temp-lo` / `temp-hi` (temperature 0 and 1) with
  `seed-random` (`seed: -1`) and `rep2`…`rep5`, five runs per setting.

## How the metrics are derived

- **Resolved** — from `evaluations/*.json` (`resolved_ids` over `completed_ids`).
- **Active wall-clock** — the sum of gaps between consecutive log lines, each gap capped
  at 5 minutes (`IDLE_GAP_S` in `generate_report.py`), so machine-sleep and
  kernel-paused stretches are excluded. It is not elapsed time.
- **Tokens, LLM calls, tool calls** — counted directly from the log stream.

Regenerate the summary with `python generate_report.py` (all runs) or
`python generate_report.py <run-folder>` (one run).

## Related repositories

- [OpenDraco](https://github.com/OpenDraco/OpenDraco) — the platform itself
- `opendraco-instance-{trivial,easy,medium,hard,expert}` — the five synthetic instances
- `translate-demo-intro-en` — fixture for the non-APR translation demonstration
