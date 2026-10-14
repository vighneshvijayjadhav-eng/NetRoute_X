# Agent Instructions — NetRoute X

## Project goal
Build NetRoute X, a Go-based reverse proxy and experimental adaptive load balancer. The project compares established routing strategies and one carefully defined custom policy under controlled backend and network faults.

## Current scope
### MVP
- Go HTTP reverse proxy using `net/http` and `httputil.ReverseProxy`.
- Multiple configurable backend servers.
- Strategies: Round Robin, Least In-Flight Requests, EWMA latency, Peak EWMA (faithful implementation or clearly documented approximation), and a custom policy.
- Backend health checks and exclusion/recovery.
- Circuit breaker and conservative retries for eligible idempotent requests only.
- Metrics for request count, latency, errors, in-flight requests, backend selection, health and recovery.
- Toxiproxy-based fault scenarios.
- k6 load tests using an open-loop `constant-arrival-rate` executor where appropriate.
- Python scripts to analyze results and plot p50/p95/p99 latency, throughput, error rate, and per-backend distribution.
- Reproducible configuration and documentation.

### Not in MVP
- Client authentication / JWT / API keys (keep middleware insertion point clean, but do not implement yet).
- Request hedging, eBPF, React dashboard, OpenTelemetry, TLS termination, Kubernetes, paid services.
- Do not claim AI or algorithmic novelty without evidence.

## Engineering principles
1. Inspect the existing repository before changing code. Do not assume files, dependencies, or behavior.
2. Prefer a clean implementation using Go standard libraries over copying entire repositories. Treat external repositories as references unless a clear reason to reuse code exists.
3. Keep routing strategies behind a small interface so they can be tested and swapped.
4. Keep telemetry collection separate from routing decisions.
5. Use bounded timeouts, bounded retries, and explicit context cancellation.
6. Never automatically retry non-idempotent requests unless the application explicitly establishes safe semantics.
7. Do not send requests to unhealthy backends.
8. Avoid retry storms, unbounded queues, and unbounded metric labels.
9. Make shared state race-safe. Run `go test ./...` and `go test -race ./...` where supported.
10. Do not fabricate benchmark results. Mark unrun tests as not run.
11. Separate observed backend response time from path RTT and backend processing time. Do not claim to detect real network congestion based only on total response latency.
12. Prefer small, reviewable changes with tests and acceptance criteria.

## Algorithm guidance
- Establish baselines before designing the custom policy.
- Include Round Robin, Least In-Flight, EWMA, and Peak EWMA or document any simplification.
- Consider Power of Two Choices as a robust base for the custom policy.
- The custom policy must have a precise written definition, configurable parameters, and tests.
- Compare fairly under identical workloads and fault scenarios.
- A custom policy performing worse is a valid experimental result; report it honestly.
- Use normalization, smoothing, stale-sample handling, and hysteresis only when justified and documented.

## Security and HTTP correctness
- Validate configuration and backend URLs.
- Avoid forwarding hop-by-hop headers incorrectly; rely on standard-library proxy behavior where possible.
- Set request deadlines/timeouts.
- Do not log secrets or full sensitive request bodies.
- Authentication is a future optional middleware; the current MVP does not promise authentication or authorization.
- Backend services should not be exposed publicly by default in deployment examples.

## Workflow for coding agents
For each task:
1. State the intended change briefly.
2. Inspect relevant files.
3. Implement the smallest complete change.
4. Add or update tests.
5. Run formatting (`gofmt`) and relevant tests.
6. Report files changed, commands run, results, limitations, and next step.
7. Never claim a command passed unless it was actually run.

## Definition of done
- Feature behavior is documented.
- Tests cover normal behavior and relevant edge cases.
- Errors are handled explicitly.
- Configuration has safe defaults and validation.
- Metrics and logs do not misrepresent measurements.
- README commands are reproducible on a clean local setup.
