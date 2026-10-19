# Continual-ARC: runs

Every run reported in the paper *Continual-ARC: Measuring Skill Acquisition over a Working Lifetime* (Daniel Lenton,
2026; [the post](https://unify.ai/blog/why-we-built-continual-arc)): 257 runs, 198 on the ARC-AGI-1 track, 56 on
ARC-AGI-2 and 3 on ARC-AGI-3, of which 90 are the ablation rows of its Appendix F. For each run there are its records,
from which the paper's tables and figures are computed, and its trajectories: every message the benchmark sent, what
the learner did inside each exchange (its reasoning summaries, its messages and its tool calls with their output), its
reply and the action read from it, and what the learner kept, file by file and version by version.

The runs were written by the benchmark, [continual-arc](https://github.com/unifyai/continual-arc), running the
learners of [continual-arc-baselines](https://github.com/unifyai/continual-arc-baselines), whose
`scripts/paper-grid.txt` launches each of them again. Any run can be replayed instance by instance at
[unify.ai/continual-arc/runs](https://unify.ai/continual-arc/runs), where each instance links to its file here.

These are the files of the benchmark's [v0.1.0 release](https://github.com/unifyai/continual-arc/releases/tag/v0.1.0)
assets, `continual-arc-runs-v0.1.0.tar.gz` and `continual-arc-trajectories-v0.1.0.tar.gz`, as plain JSON in git: about
50,000 files, 2.1 GB on disk and about 650 MB to clone. The tag `v0.1.0` is that release.

```bash
git clone https://github.com/unifyai/continual-arc-runs
```

## Layout

- `runs.json`: the registry, one entry per run: its directory (`path`), row, model preset, track, order (`mode`),
  seed and label, the harness for the harness rows other than prime-agent, and `"ablation": true` for the ablation
  rows.
- `runs/<track>/<run>/`, one directory per run. Its records, as the benchmark wrote them:
  - `records.jsonl`: one line per instance, in the order served: the task, phase (stream or epilogue), exposure index
    and gap, whether it was solved, its cost, the demonstrations requested and wrong attempts, and its resources: the
    proxy's counters of model calls, tokens and charge (`llm_*`), used throughout the paper, and the harness's own
    reports (`system_*`), which are not comparable across harnesses.
  - `summary.json`: the score and diagnostics, with a `protocol` block that says whether the run is canonical.
  - `schedule.json`: every slot the run was served; `config.json`: the environment's settings; `meta.json`: the
    learner, the protocol and when the run started.

  And its trajectories, one format for every learner:
  - `run.json`: the run's instances in the order served, each with its task, phase, exposure index and gap, whether
    it was solved, its charge, the demonstrations it bought and its wrong attempts, the model calls, tokens and spend
    the tracking proxy counted, and `trajectory` and `steps` (whether the instance has a trajectory, and whether that
    holds the learner's own steps).
  - `i/<step>.json`: one instance's trajectory, `{run, step, task, alias, turns, elided?}`. `alias` is the task's name
    as the learner saw it. Each turn is one exchange: `kind` (`observation` for every message the benchmark sent,
    feedback included; `reprompt`; `verdict`; `finish_reprompt`; `background` for steps a harness ran on its own
    between exchanges; `program` for the program library's kept program, run before the model is asked), `phase`
    (`act`, or `store` from the final verdict on), `t0` and `t1` (Unix seconds), `message`, `reply`, `action`
    (`{type, grid?, correct?}`) and `steps`. A step is `reasoning` (the provider's reasoning summary), `text`, `tool`
    (`{name, input, output, error?, stored?, recalled?}`: `stored` lists the files the call wrote, `recalled` marks a
    call that read the learner's memory, skills or past sessions) or `event` (Prime Agent's memory digest and its
    automatic refinements). `agent` marks a step that ran outside the main conversation, in a subagent or a
    scheduled session such as OpenClaw's dreaming.
  - `memory.json`: what the learner kept, `{run, kind, note?, files}`, each file with its versions
    (`{step, phase, text, change?, final?}`): the text a write left, or the edit a call made, and the copy on disk at
    the end of the run. Prime Agent's one tool is a Python kernel, so each of its writes is read from the cell that
    made it, and where the cell built the text as it ran, `change` holds the cell itself.

A run's name is `<row>-<system>-<preset>[-<track>]-<order><seed>`. The rows are F, the two floors
(`BareModelLearner`, stateless, and `BareDemoCacheLearner`); M, the program library; H, a harness with every learning
mechanism it ships; H0, the same harness with its persistence switched off; and the ablation rows HA (asked to store),
HPL and HPL0 (told to keep a program library) and M0 (the program library with nothing kept). The presets are GPT-6
Luna at an effort (`luna6-low` to `luna6-max`) and Claude Opus 5.5 at low effort (`opus55-low`). The orders are
`ws` (workstreams), `seq` (sequential), `rnd` (random) and `fresh` (the transfer reference: the held-out epilogue
alone, the learner reset before each task).

## Reading them

```bash
pip install "continual-arc @ git+https://github.com/unifyai/continual-arc"
continual-arc report runs/arc1/H-prime-agent-luna6-low-ws0
continual-arc compare runs/arc1/H-prime-agent-luna6-low-ws0 runs/arc1/H-prime-agent-luna6-low-fresh0   # forward transfer
continual-arc learned runs/arc1/H-prime-agent-luna6-low-ws0 runs/arc1/F-BareDemoCacheLearner-luna6-low-ws0   # share learned
```

Task ids in the records are the benchmark's own; a learner saw them only under aliases drawn for its run.

## Where the trajectories come from, and what is missing

Each learner's exchanges are the adapter's log of the protocol, and their outcomes the benchmark's. A harness's own
steps come from its own store: Prime Agent's RPC event log, OpenClaw's transcript database and the sessions it
archived, Hermes Agent's session database. Each step is placed in the exchange whose time window holds it.

The rows that wipe the harness before every instance (H0, HPL0 and every `fresh` run) deleted OpenClaw's and Hermes
Agent's stores with everything else, so in those runs their reasoning and tool calls survive for the last instance
alone; the benchmark's messages and the harnesses' replies are complete. Prime Agent's RPC log sat outside its state,
so its wiped runs are complete. The floors and the program library are the model alone behind the protocol, so their
replies are the whole of what they did, and the program library's programs are in them. Tool outputs and messages
longer than 60,000 characters are cut, with a marker saying how much. One entry naming software of ours on the
runners' `PATH` was taken out of the environments two harnesses printed.

## Licence

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) (see `LICENSE`).
