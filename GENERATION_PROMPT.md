# Synthetic APR instances — generation prompt

The five synthetic instances used by every `*-5custom` run in this repository were
produced from a **single prompt** asking for one repository per difficulty level.

The prompt below is a **reconstruction**. The original was not kept, and re-running it
would not reproduce the same five repositories anyway — an LLM does not return the same
output twice. It is recorded so the set can be regenerated in the same spirit, not
byte-for-byte.

## The prompt

> Create five synthetic custom instances for evaluating automated program repair, in the
> style of SWE-bench, spanning five levels of difficulty.
>
> Requirements:
>
> 1. The code must be in English.
> 2. Include a test that can be run with `pytest`.
> 3. Include a `README.md` describing the problem.

## What it produced

| Level | Module | Defect | Test cases |
|---|---|---|---|
| trivial | `is_even.py` | comparison against the wrong literal — `n % 2 == 1` instead of `== 0` | zero, positive even, positive odd, negative even |
| easy | `calculator.py` | `multiply` uses `+` instead of `*`; `add`, `subtract`, `divide` are correct | one per operation |
| medium | `rotate.py` | `rotate_left` slices without normalising `n` modulo `len(arr)`, so it returns `[]` once `n >= len(arr)` | basic rotation, `n = 0`, empty list, `n == len(arr)`, `n > len(arr)` |
| hard | `accumulator.py` | mutable default argument — `history=[]` is shared across calls, so state leaks between them | two independent default calls, explicit history, three successive default calls |
| expert | `cleanup.py` | the list is mutated while being iterated with `enumerate`, so indices shift on `pop` and elements are skipped | alternating negatives, consecutive negatives, all negative, none negative, empty |

Each instance lives in its own repository:
[trivial](https://github.com/OpenDraco/opendraco-instance-trivial),
[easy](https://github.com/OpenDraco/opendraco-instance-easy),
[medium](https://github.com/OpenDraco/opendraco-instance-medium),
[hard](https://github.com/OpenDraco/opendraco-instance-hard),
[expert](https://github.com/OpenDraco/opendraco-instance-expert).
The commit hash in each instance id — `custom-EvoMas-evomas-instance-trivial-18757fd`
and so on — is the base commit the runs were executed against.
