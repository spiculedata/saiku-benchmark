# Saiku performance benchmark

The harness, query corpus and raw reports behind the performance numbers on
[saiku.bi/benchmark](https://saiku.bi/benchmark/). Everything here is published
so you can read the method and the results in full rather than take a marketing
figure on trust.

## What is measured

Saiku's OLAP engine can emit SQL two ways: the legacy Mondrian SQL emitter, and
the newer emitter built on [Apache Calcite](https://calcite.apache.org/). This
benchmark runs the same MDX corpus through both emitters, at two dataset scales,
and reports the difference.

It is a 2x2 matrix:

|                | Legacy emitter | Calcite emitter |
|----------------|----------------|-----------------|
| HSQLDB, ~87k rows   | A | C |
| Postgres, 86.8M rows | B | D |

The Postgres fact table is 1000x the HSQLDB one (86.8M vs 87k rows); the
dimensions are identical. The two ratios that matter are **C/A** (Calcite vs
legacy at toy scale, where engine overhead dominates) and **D/B** (Calcite vs
legacy at real scale, where the database dominates). A ratio below 1 means
Calcite is faster.

## Method

- Each cell runs every MDX in the corpus with **3 warm-up plus 5 timed
  iterations**, and a **cold Mondrian schema cache before each timed
  iteration**.
- The **median** of the 5 timed runs is reported per query.
- Hardware for the published run: Apple Silicon MacBook Pro, macOS 26.3.1.

## What is honest about it

This measures **engine emitter overhead** on a FoodMart-style OLAP corpus. It is
not a cross-vendor bake-off, and it does not measure raw warehouse scan speed. At
large scale the database dominates, so the honest headline there is parity, and
the win shows up on aggregate-heavy and summarised workloads.

## Files

- `scripts/perf/run-bench-matrix.sh` drives all four cells.
- `scripts/perf/render-bench-report.py` renders the per-query JSON into the
  tables you see in the reports.
- `reports/perf-benchmark-2x2.md` is the full 2x2 report (per-query median ms and
  ratios).
- `reports/perf-benchmark-y2.md` documents the planner-cache fix and the
  aggregate-hit parity slice.
- `src/test/java/mondrian/test/calcite/PerfBenchmarkTest.java` is the JUnit
  harness that runs the corpus for a given backend and emitter and writes the
  per-query JSON.

## Running it

The harness runs against the Saiku engine build (Apache 2.0). A single cell looks
like:

```
mvn -Pcalcite-harness -Dharness.runPerfBench=true \
    -Dharness.backend=postgres -Dharness.emitter=calcite test
python3 scripts/perf/render-bench-report.py
```

## Licence

Apache 2.0. Part of [Saiku](https://github.com/spiculedata/saiku).
