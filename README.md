# Action Chain

Action Chain compiles a successfully completed agent workflow into a portable Codex skill: agent judgment stays in `SKILL.md`, while repeatable deterministic work can be preserved as executable scripts.

> Status: experimental MVP. The current repository validates one core question: can reusing a compiled workflow reduce planning cost without reducing correctness?

## Why

Agents often plan the same recurring task from scratch:

```text
Request → planning → tool selection → command generation → execution → validation
```

Action Chain preserves the successful parts of that trajectory:

```text
First successful execution
  → Action Chain Compiler
  → portable SKILL.md + optional scripts
  → reuse
  → less repeated planning
```

It does not try to turn every step into code.

- `SKILL.md` keeps input interpretation, decisions, exception handling, and validation.
- `scripts/` keeps deterministic commands, transformations, file operations, and API calls when executable logic is actually reusable.

## Included skill

This repository currently contains [`action-chain-compiler`](skills/action-chain-compiler/SKILL.md), a meta-skill that analyzes a successfully completed task in the current conversation and creates a reusable skill from it.

### Install in Codex

Clone this repository, then copy the skill into your Codex skills directory:

```bash
git clone https://github.com/0minKoh/action-chain.git
cd action-chain
cp -R skills/action-chain-compiler ~/.codex/skills/
```

Restart Codex if the skill is not detected immediately.

### Use

First complete a task successfully in a Codex conversation. Then ask:

```text
Turn the workflow we just completed into a reusable skill.
```

The compiler will identify the successful final workflow, separate agent decisions from repeatable execution, remove user-specific configuration, and create a new `SKILL.md` with scripts only when they are useful.

## Portability and security

Generated skills should work for users other than the original author. The compiler therefore:

- replaces local paths, accounts, endpoints, and variable values with explicit inputs or configuration;
- documents required tools, packages, credentials, permissions, and setup;
- reads secrets through arguments, environment variables, or configuration instead of embedding them;
- detects missing configuration and reports what the user must provide;
- never copies API keys, tokens, cookies, or private identifiers into generated artifacts.

## MVP benchmark

We ran a small controlled proof of concept on 2026-08-29. The task normalized the same 20-row CSV under two conditions:

- **Baseline:** natural-language request with no temporary task skill installed.
- **Action Chain:** the same request with an explicitly invoked compiled task skill.

Both conditions used `gpt-5.6-sol` with its default reasoning setting. Each condition ran five times, and every output was independently compared with the exact expected rows.

### Results

| Metric | Baseline mean | Action Chain mean | Reduction |
|---|---:|---:|---:|
| Exact successful runs | 5/5 | 5/5 | No loss |
| Exact output rows | 100/100 | 100/100 | No loss |
| Completion time | 43.455 s | 12.041 s | 72.3% |
| Total tokens | 239,001 | 61,615 | 74.2% |
| Uncached input tokens | 20,108 | 11,029 | 45.2% |
| Output tokens | 2,113 | 410 | 80.6% |
| Reasoning tokens | 407 | 69 | 83.0% |
| LLM calls | 7.6 | 2.4 | 68.4% |
| Tool calls | 6.6 | 1.4 | 78.8% |

Because a few baseline runs were much slower than the others, medians are also useful:

| Metric | Baseline median | Action Chain median | Reduction |
|---|---:|---:|---:|
| Completion time | 25.855 s | 10.215 s | 60.5% |
| Total tokens | 152,085 | 51,561 | 66.1% |
| Uncached input tokens | 20,936 | 10,788 | 48.5% |
| Output tokens | 1,213 | 361 | 70.2% |
| Reasoning tokens | 173 | 55 | 68.2% |
| LLM calls | 6 | 2 | 66.7% |
| Tool calls | 5 | 1 | 80.0% |

The incremental compilation itself cost 502,395 tokens and 145.5 seconds. Based on these measurements, the observed compilation cost was recovered after approximately:

- **Mean savings:** 3 reuses by tokens, 5 reuses by time.
- **Median savings:** 5 reuses by tokens, 10 reuses by time.

The complete run-level measurements are available in [`benchmarks/csv-normalization.csv`](benchmarks/csv-normalization.csv).

### Important limitations

This is an early PoC, not a general performance claim. It used one task, one model, five runs per condition, and explicit skill invocation. Baseline runs were executed before Action Chain runs, so cache warming or run order may have helped the second condition. The substantial reduction in uncached input tokens is encouraging, but a future benchmark should randomize or alternate condition order, cover multiple workflow types, and use more repetitions.

## MVP scope

The current phase focuses only on skill compilation and repeatable evaluation. It intentionally excludes a custom agent runtime, marketplace, web UI, vector database, automatic retrieval, and visual workflow builder.

Planned follow-up work:

1. Benchmark 5–10 representative recurring workflows.
2. Alternate test order and add confidence intervals.
3. Improve automatic separation of agent decisions and deterministic actions.
4. Explore automatic skill retrieval only after compilation value is established.

## Repository layout

```text
action-chain/
├── LICENSE
├── README.md
├── benchmarks/
│   └── csv-normalization.csv
└── skills/
    └── action-chain-compiler/
        └── SKILL.md
```

## License

[MIT](LICENSE)

