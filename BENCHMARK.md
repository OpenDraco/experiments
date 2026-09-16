# Task instantiations and benchmark

What OpenDraco can be pointed at, and what it scored when it was. Moved out of the
[OpenDraco](https://github.com/OpenDraco/OpenDraco) README so the tool's own docs stay
about running the tool, and the results live next to the evidence for them.

The runs behind these numbers are the `notebook-*` directories in this repository —
[`EXPERIMENT.md`](EXPERIMENT.md) summarises every one, and
[`README.md`](README.md#layout) explains how to read a run name.

## Task instantiations

OpenDraco is task-agnostic. A task is defined by three drop-in artifacts — tools, a topology config, and an evaluator — that the framework auto-discovers, with no edits to `opendraco/`, `api/`, or `app/` (see [Extending to a new problem type](https://github.com/OpenDraco/OpenDraco#extending-to-a-new-problem-type)). Three instantiations ship with the repo:

| Instantiation | What it does | Ships as |
|---|---|---|
| **Automated program repair** — the benchmarked case, see [Benchmark](#benchmark-apr-on-swe-bench) | Runs a topology over SWE-bench instances and scores the emitted patches through the official Docker harness. | 8 predefined topologies (`chain`, `hyperagent_star`, `openhands_star`, `prometheus_tree`, …) + `run_swebench_evaluation.py` / `apply_and_test.py` |
| **Repository translation** | Locates files in a repo, rewrites them in the target language, reviews the output; graded by BLEU against `.gold` sidecars. | `translate.json` + `opendraco/tools/translate/` + `translate_eval.py` |
| **Tool-assisted web search** | Answers a free-form question with web-search tools instead of repo tools. | `websearch.json` + `opendraco/tools/websearch/` + `websearch_eval.py` |

The translation and web-search demos exist precisely to show that tasks beyond APR need no changes to the core implementation.

## Benchmark: APR on SWE-bench

APR is the instantiation OpenDraco has been measured on, integrated end-to-end with the [SWE-bench](https://www.swebench.com/) evaluation harness:

- Across **eight topologies** with a fixed local model, the number of resolved synthetic instances ranged from **three to five out of five**, while execution time ranged from **17 to 42 minutes**.
- On **23 SWE-bench Lite instances**, a sequential topology and a more complex one both resolved **two** issues — but the sequential topology required **39.78% less time** and **47.32% fewer tokens**.

The insight in this setting: **more agents do not necessarily produce better outcomes**, and topology introduces measurable effectiveness–cost trade-offs. Making that variable cheap to change, re-run, and compare is the point of the platform.

### The five synthetic instances

The "five out of five" above is a difficulty ladder of purpose-built repositories, each seeding exactly one bug with a pytest suite that fails until it's fixed. They exist because SWE-bench Lite instances are slow and coarse for topology comparison: a ladder from *one wrong operator* to *a bug that reads correctly line-by-line* separates topologies that a pass/fail on two Lite issues cannot.

| # | Difficulty | Repo | The seeded bug |
|---|---|---|---|
| 1 | Trivial | `OpenDraco/opendraco-instance-trivial` | `is_even.py` returns `n % 2 == 1` where it should return `n % 2 == 0`. `test_is_even.py` exercises positive, negative and zero inputs. |
| 2 | Easy | `OpenDraco/opendraco-instance-easy` | A small calculator module carrying one deliberate arithmetic bug. |
| 3 | Medium | `OpenDraco/opendraco-instance-medium` | `rotate.py:rotate_left` slices `arr[n:] + arr[:n]`, which silently breaks once `n >= len(arr)` — `rotate_left([1,2,3], 3)` yields `[]` instead of `[1,2,3]`. One-line fix: normalize with `n = n % len(arr)`. |
| 4 | Hard | `OpenDraco/opendraco-instance-hard` | `accumulator.py:accumulate(value, history=[])` — a mutable default argument shared across every call, so state leaks between them. Fix is `history=None` plus an `if history is None` guard. |
| 5 | Expert | `OpenDraco/opendraco-instance-expert` | `cleanup.py:remove_negatives` pops from the list it is iterating with `enumerate`, so indices shift and consecutive negatives are skipped. Every line reads correct in isolation; only the output values expose it. |

Each row is a **custom-subset** instance (`subset` = `split` = `custom`) pinned to a `base_commit`, so the ladder is reproducible across runs. Custom rows carry no `test_patch` / `FAIL_TO_PASS`, so the SWE-bench Docker harness can't grade them — score these with **`apply_and_test`** (clone → apply patch → pytest), which is what the Evaluation page selects automatically when every prediction row is custom.

<details>
<summary>JSONL rows — append to <code>swebench_instances.jsonl</code></summary>

```jsonl
{"repo": "OpenDraco/opendraco-instance-trivial", "instance_id": "custom-OpenDraco-opendraco-instance-trivial-410cfc2", "base_commit": "410cfc2d1866e446fa7c9344e29e13c45cf239b6", "problem_statement": "opendraco-instance-trivial\nSynthetic SWE-bench instance for OpenDraco APR evaluation - trivial difficulty.\n\nA one-function Python module (`is_even.py`) returns the wrong boolean: `n % 2 == 1` should be `n % 2 == 0`. A failing pytest suite (`test_is_even.py`) exercises the bug across positive, negative and zero inputs.", "hints_text": "", "subset": "custom", "split": "custom"}
{"repo": "OpenDraco/opendraco-instance-easy", "instance_id": "custom-OpenDraco-opendraco-instance-easy-545dd63", "base_commit": "545dd636e34c150813443b2697f0cd33946e8fb3", "problem_statement": "opendraco-test-instance\nSynthetic test repository for OpenDraco APR evaluation.\n\nContains a simple Python calculator module with a deliberate bug for testing automated program repair.", "hints_text": "", "subset": "custom", "split": "custom"}
{"repo": "OpenDraco/opendraco-instance-medium", "instance_id": "custom-OpenDraco-opendraco-instance-medium-7a85cec", "base_commit": "7a85cececdc1a04dcb9c5315f74a28891272b355", "problem_statement": "opendraco-instance-medium\nSynthetic SWE-bench instance for OpenDraco APR evaluation - medium difficulty.\n\n`rotate.py:rotate_left(arr, n)` slices the input with `arr[n:] + arr[:n]`. This works for `n < len(arr)` but silently breaks for `n >= len(arr)`: e.g. `rotate_left([1, 2, 3], 3)` returns `[]` instead of `[1, 2, 3]`, and `rotate_left([1, 2, 3], 5)` returns `[]` instead of `[2, 3, 1]`. The fix is one line - normalize `n` modulo the array length before the slice (`n = n % len(arr)`).", "hints_text": "", "subset": "custom", "split": "custom"}
{"repo": "OpenDraco/opendraco-instance-hard", "instance_id": "custom-OpenDraco-opendraco-instance-hard-06ca51f", "base_commit": "06ca51f2b8e9ef4b1ff8ce8697651587bc1cb52f", "problem_statement": "opendraco-instance-hard\nSynthetic SWE-bench instance for OpenDraco APR evaluation - hard difficulty.\n\nClassic Python pitfall: `accumulator.py:accumulate(value, history=[])` uses a mutable default argument, so every call without an explicit `history` shares the same list object. The test `test_independent_default_calls` fails because state leaks across calls. The fix is `history=None` + `if history is None: history = []`.", "hints_text": "", "subset": "custom", "split": "custom"}
{"repo": "OpenDraco/opendraco-instance-expert", "instance_id": "custom-OpenDraco-opendraco-instance-expert-211e9b4", "base_commit": "211e9b4ccbac39d09acce6ee987684ea8e083e99", "problem_statement": "opendraco-instance-expert\nSynthetic SWE-bench instance for OpenDraco APR evaluation - expert difficulty.\n\n`cleanup.py:remove_negatives` mutates the list while iterating over it: after `items.pop(i)` every subsequent index shifts down by one but `enumerate(items)` keeps marching forward, so consecutive negative values get silently skipped. The function appears correct line-by-line - only the output values reveal the iterator-semantics bug. A correct fix uses a list comprehension, reverse iteration, or builds a new list.", "hints_text": "", "subset": "custom", "split": "custom"}
```

Or add them one at a time — the Inference page's custom-repo box does the same thing. `--subset` / `--split` are required but ignored in custom mode: the row is always tagged `custom`/`custom`. Omit `--custom-base-commit` to pin the current remote HEAD instead.

```bash
opendraco run instances --subset lite --split dev --output swebench_instances.jsonl \
  --custom-repo OpenDraco/opendraco-instance-trivial \
  --custom-problem "is_even.py returns n % 2 == 1 where it should return n % 2 == 0." \
  --custom-base-commit 410cfc2d1866e446fa7c9344e29e13c45cf239b6
```

</details>
