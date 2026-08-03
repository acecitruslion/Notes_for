# Mini API Gateway — Interview Prep Guide

⚠️ **Fix this before your interview:** your GitHub repo's "About" section says ~2000 req/sec, but both of your reference docs (and everything below) say 600 req/sec. Pick one and make sure your resume, README, and answers all agree — a mismatch here is the fastest way to lose credibility with an interviewer who's read your repo.

---

## 1. Project Motivation & Overview

### Q1. This already exists (Nginx, Kong, Envoy). Why build your own?

Don't defend it as "better than Nginx" — that's not the point, and claiming it will get you picked apart. Frame it as a learning project that lets you own every line:

- **Understand the internals, not just configure a black box.** Nginx/Kong/Envoy are configured via YAML/config files — you never write the load-balancing algorithm, circuit breaker, or rate limiter yourself. Building it from scratch forces you to actually understand token buckets, EMA latency tracking, and 3-state circuit breakers instead of just setting `proxy_pass`.
- **Demonstrate distributed-systems fundamentals**, not tool proficiency. Anyone can install Kong. Fewer people can explain *why* a circuit breaker has three states or *why* rate limiting needs to be distributed.
- **It's intentionally small (~286 LOC) so it's fully explainable.** That's a feature for an interview, not a limitation — you can walk through every module because you wrote every module.
- **Be honest about scope**: this is a learning/portfolio project, not a production replacement for Envoy. If asked "would you use this in production," say no, and pivot to Q13/Q16/Q17 (what's missing, what you'd add).

### Q2. STAR format

- **Situation**: Needed a project to demonstrate backend/distributed-systems skills — specifically async programming, failure handling, and observability — beyond typical CRUD apps.
- **Task**: Build a working API gateway that could sit in front of multiple backend services, survive backend failures gracefully, and prove its performance claims with real load tests.
- **Action**: Built an async Python gateway (aiohttp) with 6 layered components — rate limiter (Redis token bucket), latency-aware load balancer (EMA), circuit breaker (3-state), retrying proxy (exponential backoff + jitter), health checker, and Prometheus metrics. Containerized with Docker Compose (3 backends, Redis, Prometheus, Grafana) and validated with k6 load tests (throughput, rate-limit, latency-distribution, stress, sustained-load, backend-distribution).
- **Result**: Achieved ~600 req/sec with ~80ms average latency, P95 <200ms, <1% error rate, and even 3-way backend distribution (~33% each), all backed by k6 test output rather than guesses.

### Q3. Walk through one request end-to-end

Use the 7-step flow — this is the single most important thing to have cold:

1. **Rate Limit Check (Redis)** — client's token bucket is checked/decremented atomically in Redis. No tokens → `429` immediately, request never reaches the backend.
2. **Load Balancer Selection** — filter backends to those that are (a) marked healthy by the health checker and (b) not blocked by an OPEN circuit breaker; pick `min(latency + random.uniform(0, 0.01))`. No healthy candidates → `503`.
3. **Proxy Forward** — headers/body copied, request sent to the chosen backend via the pooled `aiohttp.ClientSession`. On failure, retry up to 3 times with exponential backoff + jitter (0.3s, 0.6s, then give up).
4. **Circuit Breaker Update** — status ≥500 → `record_failure()` (bumps failure counter, flips to OPEN at 5); otherwise → `record_success()` (resets counter, state CLOSED).
5. **Latency Update** — `new_latency = (old_latency + response_time) / 2` (EMA), stored per-backend for the load balancer's next decision.
6. **Metrics** — `REQUEST_COUNT.inc()`, `REQUEST_LATENCY.observe()`, and `ERROR_COUNT.inc()` on 5xx, all exposed at `/metrics` for Prometheus to scrape.
7. **Response** — status/body/cleaned headers returned to the client.

Independently of this per-request flow, a **health-check loop** runs every 5 seconds in the background (async, non-blocking), GETting each backend with a 2s timeout and updating the list of "up" servers the load balancer is allowed to pick from.

### Q4. Explain it in one sentence to a non-technical person

"It's a smart traffic cop that sits in front of several servers — it sends each request to whichever server is fastest and healthiest right now, automatically stops sending traffic to servers that are struggling so they get a chance to recover, and keeps everyone from overwhelming the system if too many requests come in at once."

---

## 2. Technology Stack & Design

### Q5. Why Python/aiohttp/Redis/Docker/k6? What would you use in production?

| Choice | Why for this project | What you'd reconsider for production |
|---|---|---|
| **Python + aiohttp** | Fast to build, async/await is native, easy to reason about and demo in an interview | Go or Rust for raw throughput — Python's GIL and interpreter overhead cap you well below what a compiled async runtime (Go's goroutines, Rust's Tokio) can do at the same hardware cost |
| **Redis** | In-memory, atomic ops, trivially shares rate-limit state across instances | Same choice in production — Redis (or a Redis-compatible service) is the standard answer here, maybe Redis Cluster for HA |
| **Docker/Docker Compose** | Reproducible multi-service local environment (gateway + 3 backends + Redis + Prometheus + Grafana) with one command | Kubernetes for real orchestration, auto-scaling, rolling deploys — Compose doesn't self-heal or scale |
| **k6** | Scriptable in JS, fast, gives real percentile data instead of guesses | Same tool works fine in production CI; might add a hosted/cloud load-testing tier for scale tests |

Be ready for the obvious follow-up: **"So why not just use Envoy/Nginx/Kong?"** — see Q1. This project proves you understand what those tools do internally, not that you're replacing them.

### Q6. How did you implement each component, and why that way?

- **Rate Limiter (token bucket, Redis)**: store `tokens` and `last_refill_time` per client in a Redis hash. On each request: `tokens = min(CAPACITY, tokens + elapsed*REFILL_RATE)`; if `tokens < 1` reject, else consume one and persist. Chose token bucket over a fixed window counter because it allows short bursts (up to CAPACITY) while still enforcing a steady average rate — a fixed window allows a burst of 2×limit right at a window boundary, which token bucket avoids.
- **Load Balancer (latency-aware + jitter)**: maintain a per-backend EMA latency; filter to healthy + circuit-closed backends; pick `min(latency + random.uniform(0,0.01))`. Chose "least-latency + jitter" over plain round robin because round robin sends equal traffic to a degrading server until it's fully dead; latency-awareness reacts within one request cycle.
- **Circuit Breaker (3-state)**: CLOSED → OPEN after 5 consecutive failures, OPEN blocks everything for 10s, then HALF_OPEN allows exactly one probe request; success resets to CLOSED, failure returns to OPEN. This is the standard Martin Fowler circuit-breaker pattern — chose it because it fails fast (no waiting on timeouts) and self-tests recovery instead of requiring a human to flip it back on.
- **Health Checker**: separate async loop, GET each backend every 5s with a 2s timeout, updates the "up" list the load balancer reads from. Deliberately decoupled from the circuit breaker — see Q13/Q18 for why you need both.
- **Retry + Exponential Backoff + Jitter**: `delay = base * 2^attempt + random.uniform(0, jitter)`, max 3 attempts. Exponential so retries don't hammer a struggling server immediately; jitter so many clients retrying the same failure don't all retry at the exact same instant (thundering herd).
- **Connection Pooling**: single shared `aiohttp.ClientSession` backed by `TCPConnector(limit=100)`, reused across all requests instead of opening a new TCP connection (and doing a new handshake) per call.

### Q7. What CS fundamentals / patterns / algorithms / data structures did you use?

- **Design patterns**: Circuit Breaker (state machine / behavioral pattern), Proxy (structural — the gateway itself is a reverse proxy), Strategy-like selection in the load balancer (pluggable "pick a backend" policy).
- **Data structures**: hash maps (per-backend latency dict, per-client Redis hash for token bucket state), a small finite state machine (3 states + transition table) for the circuit breaker.
- **Algorithms**: token bucket (rate limiting), exponential moving average (latency smoothing — O(1) space per backend, no need to store history), exponential backoff with jitter (retry scheduling), min-selection with randomized tie-breaking (`min(latency + jitter)` — a greedy online algorithm, related to "power of two choices" load balancing ideas).
- **Concurrency model**: single-threaded event loop (cooperative multitasking) instead of OS threads/processes — see Q9.
- **Systems concepts**: idempotent-ish retries, backpressure (rate limiting), graceful degradation, self-healing feedback loops (health checker + circuit breaker both feed back into the load balancer's candidate set).

---

## 3. Performance & Async Programming

### Q8. How did you test it, what did you measure, how did you optimize?

Tested with **k6** across six scenario types: basic throughput (30s), rate-limit correctness (3s), latency distribution (60s, checking P95/P99), stress/ramp (110s, finding the breaking point), sustained load (5min, checking for degradation/leaks), and backend distribution (30s, checking the load balancer spreads load evenly).

Metrics tracked: `http_reqs` (throughput), `http_req_duration` (avg/P95/P99), `http_req_failed` (error rate), and custom `checks` (assertions like "status is 200"). k6 `thresholds` encode pass/fail SLAs directly, e.g. `p(95)<500` and `rate<0.1`.

Results: ~600 req/sec, ~85ms avg latency, P95 ~150ms, P99 ~250ms, 0% error rate at target load, and even backend distribution (28–38% each, <5% variance).

Optimization loop was: run test → find bottleneck (see Q10's checklist) → fix (connection pooling saved ~1-2ms/req, latency-aware routing gave ~20% latency improvement over naive round robin, fast-failing circuit breaker saved ~5ms per failed request vs waiting for a timeout) → re-test to confirm the number moved.

### Q9. Why async instead of threads/multiprocessing?

This is an I/O-bound workload (waiting on network calls to backends and Redis), not CPU-bound, so:

- **Memory**: a thread costs ~4-8MB of stack space; 1000 concurrent threads ≈ several GB of RAM. An async event loop handles 1000s of concurrent connections in one thread with tens of KB of overhead per coroutine.
- **Context-switch cost**: OS thread switches are expensive (kernel involvement); coroutine switches inside one event loop are cheap, cooperative, and happen exactly at `await` points, which you control.
- **The GIL**: Python threads don't give you true CPU parallelism anyway (only one thread executes Python bytecode at a time) — so threading buys you concurrency without buying you parallelism, and you pay the thread overhead for nothing extra during I/O waits. Async gets you the same I/O concurrency without that overhead.
- Async is used throughout: the aiohttp server handler (`async def handle`), the proxy's outbound calls (`async with session.request(...)`), the health-check loop, and `asyncio.sleep` for backoff — none of these block the event loop while waiting.
- Multiprocessing wasn't the right tool here because the bottleneck is waiting on network I/O, not CPU computation — multiprocessing helps when you need parallel CPU work and are willing to pay for separate memory spaces and IPC.

### Q10. How does connection pooling help? What if you didn't use it?

Every new TCP connection costs a 3-way handshake (~1-2ms locally, more over a real network) plus, for HTTPS, a TLS handshake. `TCPConnector(limit=100)` keeps a pool of already-established connections and reuses them across requests instead of opening/closing a socket per call.

Without it: every single proxied request pays handshake latency on top of actual processing time, throughput drops because sockets spend time in handshake/TCP-slow-start instead of transferring data, and under load you risk exhausting ephemeral ports or hitting OS file-descriptor limits from constantly creating/destroying sockets. At 600 req/sec that's not catastrophic, but at higher scale it becomes a real ceiling — this is exactly the kind of thing that shows up as "mystery latency" in `http_req_blocked` during a k6 run (see Q's bottleneck checklist below).

---

## 4. Redis & Reliability

### Q11. Why Redis? What's actually stored? Why not another DB or in-memory?

**What's stored**: per-client hash `bucket:{client_id}` → `{tokens: float, last: timestamp}`. That's it — small, ephemeral, doesn't need to survive forever.

**Why Redis over in-memory dict**: the whole point is *distributed* rate limiting. If each gateway instance kept its own in-memory counter, running 2 gateway instances behind a load balancer would let a client get 2× the intended limit (10 requests to instance A + 10 to instance B = 20, when the SLA says 10). Redis gives every instance a single shared source of truth.

**Why Redis over a full DB (Postgres/MySQL)**: rate-limit checks happen on the hot path of *every single request* — you need sub-millisecond, in-memory reads/writes with atomic increment/compare semantics, not disk-backed ACID transactions. Redis's `HSET`/`HGETALL` are fast and its single-threaded execution model gives you atomicity for free (no need for explicit locking, unlike a naive read-modify-write against a SQL row under concurrent access).

**Trade-off to acknowledge**: Redis is a single point of failure for rate limiting unless you run it in a cluster/HA setup — see Q12/Q17.

### Q12. What happens if Redis / a backend / any component fails?

- **A backend fails**: circuit breaker records failures → opens after 5 → that backend is fast-failed (no wasted timeouts) → health checker independently stops listing it as "up" within 5s → when it recovers, health checker re-adds it and the circuit breaker's next HALF_OPEN probe (after the 10s cooldown) confirms recovery and closes the circuit. Two independent, overlapping safety nets (see Q13).
- **Redis fails**: as written in the reference docs, the naive implementation would raise an exception on every rate-limit check and crash the request handler — a bad failure mode. The fix (and what you should say you'd do, or already did) is to wrap the Redis call in try/except and either (a) fail open (allow the request, log the incident, accept temporary DDoS exposure) or (b) fail closed (reject everything, safer but harsher), or (c) fall back to a local in-memory limiter per instance (not distributed, but still *some* protection). Be honest if your current code doesn't handle this — it's a great "what I'd improve" answer for Q12/Q20.
- **Health checker itself becomes slow**: it's fully async and decoupled from request handling, so a slow health check cycle doesn't block requests — worst case, the "up" server list is stale for longer (see Q18), but the circuit breaker (which reacts within a single request, ~5ms) is the real first line of defense; health checking is the cleanup/recovery layer, not the primary one.

### Q13. How do Circuit Breaker + Health Checks + Retry + Backoff work together?

They're layered, not redundant, and each protects against something the others don't:

| Mechanism | Detects in | Protects against |
|---|---|---|
| Retry + backoff | Single request (0.3–0.7s) | *Transient* failures (one dropped packet, momentary blip) |
| Circuit breaker | ~5 failed requests (fast, per-backend) | *Sustained* backend failure — stops wasting time/resources retrying a backend that's actually down, and stops cascading load onto it |
| Health checker | Every 5s (independent loop) | Confirms/refreshes which backends are reachable at all, independent of live traffic; a backend can be re-admitted even if no requests happen to be routed to it to trigger a circuit-breaker probe |

Retry handles the "maybe it was a blip" case cheaply; if the blips keep happening, the circuit breaker escalates to "stop trying entirely" so you fail fast instead of piling up 3× retries × concurrent requests against a dead server; the health checker runs independently in the background so recovery isn't purely dependent on live traffic happening to probe the right backend at the right time.

---

## 5. Scalability & Production System Design

### Q14. How would you scale this to millions of users?

- **Horizontal scale the gateway itself**: run N gateway instances behind a load balancer (nginx, or a cloud LB) in front of them — this works cleanly *because* rate-limit state already lives in shared Redis, not in-process memory.
- **Redis becomes the next bottleneck** at scale — move to Redis Cluster (sharded) or a managed HA Redis so it isn't a single point of failure/throughput ceiling.
- **Backend discovery**: replace the hardcoded 3-backend list with dynamic service discovery (Consul, etcd, Kubernetes Service/Endpoints) so backends can be added/removed without redeploying the gateway.
- **Kubernetes** for the gateway and backends: horizontal pod autoscaling based on CPU/latency/request-rate, rolling deploys, self-healing (crashed pods restarted automatically) — this replaces a lot of what Docker Compose can't do.
- **Push work out of the hot path**: async logging/metrics shipping (don't block requests on it), consider caching idempotent GET responses.
- **Split concerns further at extreme scale**: dedicated rate-limiting service, dedicated service mesh sidecars (Envoy) instead of one monolithic gateway process handling everything.

### Q15. API Gateway vs Load Balancer — why both?

A **load balancer** does one job: distribute traffic across a set of backend instances of the *same* service, usually at L4 (TCP) or basic L7 (HTTP), based on an algorithm (round robin, least connections, latency).

An **API Gateway** is a superset — it's the single entry point for potentially *many different backend services*, and adds cross-cutting concerns that a plain load balancer doesn't: rate limiting, authentication/authorization, request routing by path/version, protocol translation, circuit breaking, observability, and (often) it uses a load balancer internally as one of its components. In this project, the load balancer (`loadbalancer.py`) is literally one layer inside the larger gateway — that's the relationship in miniature: gateway ⊃ load balancer.

In short: every API gateway load-balances, but not every load balancer is a gateway.

### Q16. How would you secure, deploy, monitor, and log this in real production?

- **Security**: authn/authz (API keys, OAuth2/JWT — see the topics list), TLS/HTTPS termination, input validation on proxied requests, security headers (HSTS, CSP where relevant), secrets management (not hardcoded config) for Redis/backends credentials.
- **Deploy**: containers behind Kubernetes (or ECS/similar) instead of a single Compose file — rolling updates, health-check-based readiness/liveness probes, autoscaling, multi-AZ for HA.
- **Monitor**: keep Prometheus + Grafana, but add alerting rules (e.g., page on error rate >5% or P95 >500ms for 5 minutes), and dashboards per backend, not just aggregate.
- **Log**: structured request/response logging (who, what, when, status, latency) shipped to a centralized system (ELK/Loki), correlated with the OpenTelemetry tracing that's already scaffolded but unused in the current build — wiring that up gives you distributed tracing across gateway → backend.
- **Config**: move constants (CAPACITY, REFILL_RATE, failure_threshold, etc.) out of hardcoded values into environment variables / a config service so they're tunable without a redeploy.

### Q17. If this were an enterprise-grade gateway, what else would you add?

- Authentication/authorization layer (API keys, OAuth2, mTLS between gateway and backends)
- Request/response transformation and versioned routing (route by path, header, or API version)
- Per-client / per-route rate limiting (not just per-client-global) and quota management
- Caching layer for cacheable GET responses
- Distributed tracing actually wired up (OpenTelemetry is present but unused — see Q13 in the "production readiness" list)
- Canary/blue-green deployment support, weighted/A-B routing
- WAF-style request validation and abuse protection
- Multi-region failover, not just multi-backend failover within a region
- A proper admin/config API instead of hardcoded constants

---

## 6. Engineering Decisions

### Q18. Biggest challenges / trade-offs?

Frame trade-offs honestly — interviewers want to see you understand costs, not just benefits:

- **EMA latency (`(old+new)/2`) is simple but crude.** It weights the single most recent sample at 50%, which reacts fast to change but can also overreact to one slow outlier and is noisier than, say, a proper weighted moving average or a percentile-based tracker. Trade-off: simplicity and O(1) memory vs. accuracy/stability.
- **Circuit breaker thresholds (5 failures / 10s recovery) are fixed constants, not adaptive.** Easy to reason about and demo, but not tuned to real traffic patterns — a production system would likely make these configurable per-backend or adaptive to traffic volume.
- **Redis as a single dependency for rate limiting is a trade-off between simplicity/correctness (real distributed state) and introducing a new single point of failure** (see Q12/Q17).
- **In-process constants instead of config management** — fast to build, but not operationally flexible.

### Q19. Hardest bug / issue, and how you debugged it

Answer this with your *own real story* — this is the one question you can't fake from the docs. If you don't have a specific memorable bug, use the bottleneck-diagnosis process as a template for how you'd approach it live:

- Reproduce under load (k6), not just by hand — some bugs (thundering herd, circuit-breaker flapping, uneven distribution) only show up under concurrency.
- Isolate the layer: is it rate limiter (check for unexpected 429s), load balancer (check distribution %), circuit breaker (`docker logs gateway | grep OPEN`), or backend itself (`curl` it directly with timing)?
- Check the obvious infra first: `docker-compose ps`, `docker stats`, `docker logs redis | grep slow`.
- Prepare to talk about *one specific instance* — e.g., "the load balancer kept sending traffic to a backend that had just recovered but whose EMA latency was still artificially high from its outage, so it never got picked even though it was healthy" is exactly the kind of self-healing-but-slow-to-adapt issue this architecture can produce (see the "self-healing" EMA decay example in your reference doc — recovery from 275ms → 80ms took several requests, not one).

### Q20. Biggest current limitation / bottleneck?

- Rate limiter has no graceful Redis-failure fallback (see Q12) — that's a real gap, not a hypothetical.
- Hardcoded backend list — no dynamic discovery, can't scale backends without redeploying.
- Circuit breaker state and failure counters are in-process — with multiple gateway instances, each instance has its *own* circuit breaker view of each backend, which isn't shared (unlike rate limiting, which *is* shared via Redis). That's an inconsistency worth calling out proactively — a sharp interviewer will spot it.
- No auth, no TLS, no per-route limits (see Q13/Q17 "missing" list).
- Single Redis instance (no HA) is a SPOF for rate limiting.
- Constants aren't configurable without a code change.

### Q21. If you started over — what would v2.0 look like?

- Move circuit-breaker state into Redis too, so it's consistent across gateway instances (fixes Q20's inconsistency).
- Config-driven constants (env vars or a config file) instead of hardcoded thresholds.
- A pluggable load-balancing strategy interface (round robin / least-connections / latency-aware) instead of one hardcoded algorithm — useful for comparing strategies in tests.
- Wire up OpenTelemetry tracing end-to-end instead of leaving it scaffolded-but-unused.
- Add a proper test suite (unit tests per module + integration tests), which the current project doesn't have.
- Consider Go or Rust if raw throughput mattered more than approachability/demo-ability.

---

## 7. Project Ownership

### Q22. How would you onboard another engineer?

Walk them through it top-down, same order as this doc: start with the one-request flow (Q3) as the mental model, then go module-by-module (`server.py` orchestrates → `rate_limiter.py` → `loadbalancer.py` → `circuit_breaker.py` → `proxy.py` → `health_checker.py` → `metric.py`), pointing out that each file has a single responsibility (easy to test/modify in isolation), then show `docker-compose up` + a k6 run so they see it working end-to-end before diving into code.

### Q23. Can you explain every module/function in detail?

Know these cold (they're all in your reference docs — re-read the code blocks, not just the prose, before the interview):
- `rate_limiter.py::is_allowed()` — token bucket math against Redis hash
- `loadbalancer.py::get_next_server()` / `update_latency()` — candidate filtering + min-latency+jitter selection, EMA update
- `circuit_breaker.py::CircuitBreaker` — `allow_request()`, `record_failure()`, `record_success()`, the state transitions
- `proxy.py::forward_request()` — retry loop with exponential backoff
- `health_checker.py::health_check_loop()` — the 5s async loop
- `server.py::handle()` — the orchestrator tying all 7 steps together (this is the one to be able to narrate line-by-line)
- `metric.py` — the three Prometheus metrics (Counter × 2, Histogram × 1) and what each is for

### Q24. Can you justify every claim on your resume?

Cross-check each number against your actual k6 output, not the reference doc's example numbers — if your resume says "600 req/sec, 80ms latency," be ready to show (or at least accurately describe) the k6 run that produced it. Also resolve the 600-vs-2000 req/sec discrepancy between your PDFs and your GitHub README *before* anyone asks — see the warning at the top of this doc.

---

## Paper / Whiteboard Coding Questions

These are the kinds of "implement it live" questions that naturally fall out of this project. Practice writing these without your reference docs open.

**1. Implement a token bucket rate limiter (in-memory, single instance)**
```python
import time

class TokenBucket:
    def __init__(self, capacity: float, refill_rate: float):
        self.capacity = capacity
        self.refill_rate = refill_rate  # tokens per second
        self.tokens = capacity
        self.last_refill = time.monotonic()

    def allow(self) -> bool:
        now = time.monotonic()
        elapsed = now - self.last_refill
        self.tokens = min(self.capacity, self.tokens + elapsed * self.refill_rate)
        self.last_refill = now
        if self.tokens >= 1:
            self.tokens -= 1
            return True
        return False
```
Follow-ups to expect: "make it thread-safe" (add a lock), "make it distributed" (move state to Redis with an atomic Lua script instead of separate GET+SET, to avoid race conditions), "implement leaky bucket instead" (see topic list below for the conceptual difference).

**2. Implement a circuit breaker state machine**
```python
import time

class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_time=10):
        self.failure_threshold = failure_threshold
        self.recovery_time = recovery_time
        self.state = "CLOSED"
        self.failure_count = 0
        self.last_failure_time = None

    def allow_request(self) -> bool:
        if self.state == "OPEN":
            if time.time() - self.last_failure_time > self.recovery_time:
                self.state = "HALF_OPEN"
                return True
            return False
        return True  # CLOSED or HALF_OPEN

    def record_success(self):
        self.failure_count = 0
        self.state = "CLOSED"

    def record_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = "OPEN"
```
Follow-up: "what if two requests hit HALF_OPEN simultaneously?" — real answer: your naive version above lets both through, which defeats the "test with exactly one request" intent; fix with a lock/flag so only one probe request is allowed through while in HALF_OPEN.

**3. Pick the backend with minimum latency + jitter, from a list**
```python
import random

def pick_backend(candidates: dict[str, float]) -> str:
    # candidates: {backend_name: latency_ms}
    return min(candidates, key=lambda b: candidates[b] + random.uniform(0, 0.01))
```
Follow-up: "why not always pick the strict minimum?" — thundering herd: without jitter, every gateway instance (or every request in a tight loop) picks the exact same "fastest" server, overloading it and making it the new slowest server.

**4. Exponential backoff with jitter (retry wrapper)**
```python
import asyncio, random

async def retry_with_backoff(fn, retries=3, base_delay=0.3, jitter=0.1):
    for attempt in range(retries):
        try:
            return await fn()
        except Exception:
            if attempt == retries - 1:
                raise
            delay = base_delay * (2 ** attempt) + random.uniform(0, jitter)
            await asyncio.sleep(delay)
```

**5. Implement a simple EMA tracker**
```python
class LatencyTracker:
    def __init__(self):
        self.latency = {}

    def update(self, backend: str, response_time: float):
        old = self.latency.get(backend, response_time)
        self.latency[backend] = (old + response_time) / 2
```
Follow-up: "generalize to a weighted EMA with configurable alpha" — `new = alpha * current + (1 - alpha) * old`, where alpha=0.5 is the special case implemented above.

**6. Classic adjacent-topic questions worth rehearsing** (not in your codebase, but a natural jump from "you built a rate limiter/LB"): implement an LRU cache (for a hypothetical response cache), implement round-robin and least-connections load balancing as alternatives to your latency-aware one, implement a sliding-window rate limiter and be ready to compare it to token bucket.

---

## Quick-Reference: Follow-up Topics

Know the one-line answer to each — you listed these as "don't write separate answers," so here they are compressed to that:

- **Asyncio Event Loop**: single-threaded loop that runs coroutines cooperatively, switching at `await` points instead of OS-level preemption.
- **aiohttp**: async HTTP client/server library for Python; used here for both the gateway's server and its outbound proxy client.
- **Reverse Proxy**: a server that sits in front of backend servers and forwards client requests to them, hiding backend topology from the client — this gateway *is* one.
- **API Gateway**: a reverse proxy plus cross-cutting concerns (auth, rate limiting, routing, observability) — see Q15.
- **Load Balancing Algorithms**: Round Robin (rotate evenly, ignores server load), Least Connections (send to whoever has fewest active requests), vs. this project's Latency-Aware (send to whoever's fastest recently).
- **Token Bucket vs Leaky Bucket**: token bucket allows bursts up to capacity then throttles to the refill rate; leaky bucket smooths output to a constant rate regardless of burstiness (like a queue draining at a fixed rate) — token bucket is more burst-friendly, leaky bucket is stricter/smoother.
- **Circuit Breaker States**: CLOSED (normal) → OPEN (blocking, after N failures) → HALF_OPEN (single test request) → CLOSED or back to OPEN.
- **Health Checks**: periodic liveness probes (independent of live traffic) used to include/exclude backends from the routable set.
- **Retry & Exponential Backoff**: retry failed calls with increasing delay (`base * 2^n`) plus random jitter to avoid synchronized retries.
- **Connection Pooling**: reuse TCP (and TLS) connections across requests instead of paying handshake cost every time.
- **Redis**: in-memory data store used here for shared, atomic, fast rate-limit state across gateway instances.
- **Docker & Docker Compose**: containerize each service; Compose orchestrates multi-container local/dev deployments (not a substitute for Kubernetes at scale).
- **k6**: JS-scripted load-testing tool; measures throughput/latency/error-rate against defined pass/fail thresholds.
- **RPS**: requests per second — your throughput metric.
- **Latency**: time from request sent to response received.
- **Throughput**: volume of requests successfully handled per unit time.
- **p50/p95/p99**: percentile latencies — p95 means 95% of requests were faster than this value; used for SLAs because averages hide tail latency.
- **Horizontal vs Vertical Scaling**: horizontal = more instances (needs shared state, e.g. Redis); vertical = bigger machine (simpler, but has a ceiling and a single point of failure).
- **JWT / OAuth / API Keys**: none currently implemented in this project (flagged as a gap in Q13/Q17) — JWT is a signed, stateless token; OAuth is a delegated-authorization framework; API keys are simple static credentials per client.
- **Prometheus / Grafana**: Prometheus scrapes and stores time-series metrics (pull model); Grafana visualizes them in dashboards.
- **Kubernetes**: container orchestration with self-healing, autoscaling, and rolling deploys — the natural next step beyond Docker Compose (see Q14/Q16).
- **Nginx vs Kong vs Envoy**: Nginx is primarily a web server/reverse proxy/LB; Kong is an API gateway built on Nginx with a plugin ecosystem (auth, rate limiting, etc. as plugins); Envoy is a modern L7 proxy built for service-mesh/microservices use, with native observability and dynamic config — your project is a minimal, from-scratch version of what Kong/Envoy give you out of the box.
