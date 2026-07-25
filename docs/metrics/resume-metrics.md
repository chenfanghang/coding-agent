# Pico Resume Metrics

## Key Numbers
- Experiment mode: synthetic
- Model backends: 3
- Tool types: 7
- Fixed benchmark tasks: 12
- Fixed benchmark pass rate: 100.00%
- Aggregated runs: 30
- Average tool steps per run: 0.73
- Average attempts per run: 1.73
- Cache hit rate: 0.00%
- Synthetic prompt chars (full vs no context reduction): 5738 / 6800
- Memory repeated reads (on vs off): 0 / 3
- Large-scale memory tasks: 12
- Context matrix configs: 12
- Security scenarios: 10

## Resume Highlights
- Built a fixed benchmark harness with 12 tasks and automated pass/fail, verifier, and budget summaries.
- Recorded 3 run artifacts per execution and structured runtime metadata across 30 aggregated runs.
- Observed prompt-cache telemetry with average cached tokens of 0.0 and cache-hit rate of 0.00% when available.
- In a synthetic long-context stress scenario, context reduction shrank prompt size from 6800 to 5738 chars.
- In the memory dependency experiment, repeated follow-up reads dropped from 3 to 0.
- In the large-scale memory experiment, repeated reads dropped from 60 to 0 across 12 tasks.

