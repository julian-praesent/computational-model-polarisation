# Echo chambers and polarization: replication and extension

This project replicates the opinion dynamics model of Baumann et al. (2020) and extends it.

In the model, agents with different activity levels talk to each other on a network that changes every time step. They prefer partners with similar opinions and reinforce each other's opinions. Depending on the parameters, the population reaches a neutral consensus, radicalizes to one side, or splits into two opposing groups (echo chambers).

We do three things with it:

1. **Replicate** Figs. 1-4 of the paper, both in NetLogo and in R.
2. **Extend** the model so that the number of agents an active agent contacts depends on how extreme its opinion is (**m-mode**), and compare the four modes.
3. **Record echo-chamber snapshots**: who hears whom over several time steps once the population is polarized.

See the [PsychoModels entry](https://www.psychomodels.org/models/social-media-polarization-model/) for further information

## Requirements

- [NetLogo 7.0.4](https://ccl.northwestern.edu/netlogo/)
- R (>= 4.5)

## Project structure

```
.
├── README.md
├── netlogo/
│   └── polarisation.nlogox        the model (paper + m-mode + optional snapshots)
├── R/
│   ├── replication.Rmd            the model in plain R; reproduces Figs. 1 and 2
│   ├── m-modes-analysis.Rmd       compares the m-modes using data/m-modes/
│   ├── activity-opinion.Rmd       Fig. 3 activity against opinion per m-mode (based on data/activity-opinion/)
│   └── echo-chambers.Rmd          reproduces Fig. 4 echo-chamber (based on data/snapshots/)
├── data/
│   ├── m-modes/m-modes.csv        BehaviorSpace output of the experiment "m-modes"
│   ├── activity-opinion/activity-opinion.csv   BehaviorSpace output of the experiment "activity-opinion"
│   └── snapshots/<m-mode>/        BehaviorSpace ouput of the snapshots
└── results/
    ├── replication/               Figs. 1a-c and 2, written by R/replication.Rmd
    ├── m-modes/                   summary table and plots, written by R/m-modes-analysis.Rmd
    ├── activity-opinion/          Fig. 3 per m-mode and table of polarized runs, written by R/activity-opinion.Rmd
    └── snapshots/                 Fig. 4 echo-chamber, written by R/echo-chambers.Rmd
```

`data/` holds what the simulations produce and the notebooks read; `results/` holds the tables and figures the notebooks make. The four snapshot folders are named after the m-modes: `uniform`, `boost-extreme`, `uprank-bridging-and-downrank-toxic` and `uprank-bridging`.

## The NetLogo model

`netlogo/polarisation.nlogox` has three parts. They are separate sections of the code and separate groups on the interface:

| Part | What it does | Switched on by |
|------|--------------|----------------|
| Model (Baumann et al. 2020) | The algorithm of the paper's Supplementary Material, integrated with fourth-order Runge-Kutta | always |
| Extension: number of contacts | The number of contacts depends on the agent's opinion; the mean over all agents stays m | `m-mode` other than `uniform` |
| Optional: echo-chamber snapshots | Adds up the networks of several time steps after polarization and writes each agent's opinion and its neighbours' mean opinion to a file | `collect-snapshots?` |

With `m-mode` = `uniform` and `collect-snapshots?` off, the model is exactly the paper's. The Info tab of the model explains every parameter, plot and output.

### m-modes

| m-mode | Who gets more contacts |
|--------|------------------------|
| `uniform` | nobody: everyone contacts m agents, as in the paper |
| `boost-extreme` | extreme agents, up to `m-multiplier` times as many as neutral ones |
| `uprank-bridging and downrank-toxic` | neutral agents, up to `m-multiplier` times as many as extreme ones |
| `uprank-bridging` | like the previous one, but the `extreme-share` % most extreme agents keep the number they had before they became that extreme |

### BehaviorSpace experiments

| Experiment | Parameters | Output |
|------------|------------|--------|
| `m-modes` | (alpha, beta) = (0.05, 2), (3, 0), (3, 3) plus the other combinations of these values; 4 m-modes; 100 repetitions; N = 1000; stopped at t = 3.5 | table output, to be saved as `data/m-modes/m-modes.csv` |
| `echo-chambers-uniform` | alpha = 4.5, beta = 2, K = 2.5, r = 0.65, N = 1000; m-mode `uniform`; 200 repetitions; snapshots on | one file per run in `data/snapshots/uniform/` |
| `echo-chambers-boost-extreme` | same, m-mode `boost-extreme` | `data/snapshots/boost-extreme/` |
| `echo-chambers-uprank-bridging-and-downrank-toxic` | same, m-mode `uprank-bridging and downrank-toxic` | `data/snapshots/uprank-bridging-and-downrank-toxic/` |
| `echo-chambers-uprank-bridging` | same, m-mode `uprank-bridging` | `data/snapshots/uprank-bridging/` |
| `activity-opinion` | alpha = 3, beta = 1, K = 2, r = 0.65, N = 1000; 4 m-modes; 50 repetitions; stopped at t = 10 | spreadsheet output, to be saved as `data/activity-opinion/activity-opinion.csv` |

The Info tab of the model lists every setting of each experiment.

## Reproducing the results

### 1. Figs. 1 and 2 of the paper

Knit `R/replication.Rmd`.

- Input: none
- Output: `results/replication/fig1a-consensus.png`, `fig1b-one-sided-radicalization.png`, `fig1c-polarization.png`, `fig2-phase-diagram.png`
- Run time: a few seconds per panel of Fig. 1. Fig. 2 runs 400 simulations and takes about 40 minutes. It is cached, so knitting again is quick; delete `R/replication_cache/` after changing the model code.

The same figures can be explored interactively in NetLogo with the buttons **Fig. 1a**, **Fig. 1b** and **Fig. 1c**.

### 2. Comparison of the m-modes

1. Open `netlogo/polarisation.nlogox` and run the BehaviorSpace experiment `m-modes` (Tools > BehaviorSpace).
2. Tick only "Table output" and save the file as `data/m-modes/m-modes.csv`. These are 2400 runs, so use as many parallel runs as you have cores.
3. Knit `R/m-modes-analysis.Rmd`.

- Input: `data/m-modes/m-modes.csv`
- Output in `results/m-modes/`:
  - `summary.csv`: mean, standard deviation and ratio to uniform, per regime and m-mode
  - `abs-mean-opinion.png`, `opinion-variance.png`: each measure per mode and regime
  - `variance-violin-alpha<a>-beta<b>.png`: distribution of the variance, one plot per regime (16:9, for slides)

### 3. Fig. 4 of the paper (echo chambers)

1. Run the four BehaviorSpace experiments `echo-chambers-<m-mode>`; BehaviorSpace output is not needed. Each run writes `data/snapshots/<m-mode>/snapshots-run-<run number>.csv`. Empty a folder before running its experiment again, otherwise the notebook pools old and new runs. To try it in the interface instead, switch on `collect-snapshots?` and press **setup** and **go**; the file is then `data/snapshots/snapshots-run-0.csv`, which the notebook doesn't read.
2. Knit `R/echo-chambers.Rmd`. It needs the R package `MASS`, which comes with R. For each m-mode it pools all runs and plots each agent's opinion x against the mean opinion of its neighbours, with the distributions of both at the sides.

- Input: `data/snapshots/<m-mode>/*.csv`
- Output: `results/snapshots/echo-chambers-<m-mode>.png`

### 4. Fig. 3 of the paper (activity against opinion)

1. Open `netlogo/polarisation.nlogox` and run the BehaviorSpace experiment `activity-opinion`. Its two metrics are the final opinions and the final activities of all agents, as lists sorted by agent number.
2. Tick only "Spreadsheet output" and save the file as `data/activity-opinion/activity-opinion.csv`.
3. Knit `R/activity-opinion.Rmd`.

- Input: `data/activity-opinion/activity-opinion.csv`
- Output in `results/activity-opinion/`:
  - `activity-opinion.png`: 2-D histogram of activity against opinion, one panel per m-mode, all runs pooled
  - `polarized-runs.csv`: per m-mode, the number of polarized runs (at least 20% of the agents on each side) and the mean and standard deviation of the mean |x|

## File formats

**`data/m-modes/m-modes.csv`** is the BehaviorSpace table output: 6 lines about the experiment, then one row per run. Each row has the run number, the parameter values, the step at which the run ended and three measures:

| Column | Meaning |
|--------|---------|
| `abs-mean-opinion` | abs(x), the absolute mean opinion |
| `opinion-variance` | variance of the opinions |
| `mean [ n-contacts ] of turtles` | mean number of contacts; always m |

**`data/activity-opinion/activity-opinion.csv`** is BehaviorSpace spreadsheet output: 6 lines about the experiment, then one row per parameter or measure and three columns per run. The parameters are in the first of a run's three columns. In the row below `[final value]`, the three columns hold the last step, the list of the final opinions and the list of the final activities of all agents, sorted by agent number. NetLogo writes each list as one cell, e.g. `[0.12 -1.5 3.4]`.

**`data/snapshots/<m-mode>/snapshots-run-<n>.csv`** has one row per agent that has at least one neighbour in the snapshots:

| Column | Meaning |
|--------|---------|
| `run` | BehaviorSpace run number |
| `alpha`, `beta`, `K`, `m_mode` | parameters of the run |
| `t` | time at which the file was written (end of the collection) |
| `agent` | agent number |
| `x` | the agent's opinion |
| `x_nn` | mean opinion of its neighbours |
| `k` | number of neighbours (or their total weight, with `snapshot-weighted?` on) |
| `activity` | the agent's activity |

The files `run_<n>.csv` now in `data/snapshots/` come from an earlier version of the model, run with the settings of the `echo-chambers-*` experiments. They have the columns `run_id, agent_id, x, x_nn, k, activity`, without the parameters. `R/echo-chambers.Rmd` only uses `x` and `x_nn`, so it reads both kinds.

## Notes

- **Run length.** The `m-modes` runs stop at t = 3.5 instead of t = 10 (the paper's run length) to save computing time. All results in `results/m-modes/` refer to t = 3.5.
- **Random seeds.** In BehaviorSpace the random seed is the run number, so every run can be repeated. In the interface every setup starts a new random run.

## Reference

Baumann, F., Lorenz-Spreen, P., Sokolov, I. M., & Starnini, M. (2020). Modeling echo chambers and polarization dynamics in social networks. *Physical Review Letters*, 124, 048301. https://doi.org/10.1103/PhysRevLett.124.048301

## License

GNU General Public License v3.0; see `LICENSE` in the repository root.
