# Local performance benchmark

Measured October 18, 2026 against application source `50ee43445eb87ec338959ae283be708281c068eb`. The benchmark files and documentation were added afterward; application code was unchanged.

## Workload and scope

Authenticated reads of one synthetic patient timeline; shared session; login excluded; no network, disk database or native client benchmark.

Synthetic local loopback HTTP benchmark. Not production capacity, real users, clinical validation or user growth. These measurements describe one short workload on one machine. They do not establish a capacity ceiling, sustained service-level objective, p99 guarantee, production response time or adoption. No historical speedup is claimed unless a comparison is explicitly defined below.

Environment: Darwin 25.2.0, arm64, 12 logical CPUs, Python 3.12.14. openjdk version "21.0.6" 2025-01-21 LTS.

- 8 concurrent HTTP clients, 40 untimed warmup requests.
- 3 rounds of 800 measured requests: **2,400 total**.
- Requests use persistent local loopback HTTP connections. The client and server share one machine.
- Every response is checked for HTTP success and workload-specific correctness; failures abort without producing a successful report.
- All data is synthetic and isolated in a temporary directory.

## Results

| Round | Requests | Median latency | p95 latency | Requests/second | Failures |
|---|---:|---:|---:|---:|---:|
| 1 | 800 | 1.835 ms | 4.027 ms | 3542.96 | 0 |
| 2 | 800 | 1.304 ms | 2.894 ms | 4863.43 | 0 |
| 3 | 800 | 0.950 ms | 2.228 ms | 6250.42 | 0 |

Combined latency: **1.317 ms median**, **3.295 ms p95**. Median round throughput: **4863.43 requests/second**. All **2,400 requests passed** their checks. p95 is the nearest-rank 95th percentile; throughput includes client scheduling and validation.

The fixture is one patient with five seeded clinical records, backed by in-memory H2. It is not a PostgreSQL, large-record, multi-patient, mobile-device, authentication-throughput or clinical benchmark. Timeline reads execute the existing authorization, provenance and audit path.

## Reproduce

```sh
python3.12 -m venv .benchmark-venv
.benchmark-venv/bin/python -m pip install -r benchmarks/requirements.txt
cd apps/backend
# Use a Java 21 installation.
./mvnw -DskipTests package
cd ../..
.benchmark-venv/bin/python benchmarks/benchmark.py health-passport --output /tmp/health-passport-benchmark.json
```

Health Passport can select a Java executable with `--java /path/to/java`. Other benchmark modes need no Java. The script creates temporary data, starts only a loopback server and stops it after the run. Compare results only under controlled, comparable machine and workload conditions.

[Recorded JSON](results-2026-10-18.json) contains the source revision, environment, per-round results and comparison timings where applicable.
