---
abstract: |
  Step 4 of the load tester: how it is built. A Flask
  API with a Locust-based runner, an SQS worker, a
  SQLite store and a React page; each part named, with
  the rule or test that justifies it, and where each
  part runs.
date: 2026-10-10
keywords:
- load-testing
- implementation
- flask
- locust
- sqs
kind: explanation
sources:
- code/app.py
- code/load_test_runner.py
- code/worker.py
status: draft
subtitle: Step 4, the implementation
title: "Load Tester: the Implementation"
version: v0.1.0
---

> [!NOTE]
> Each row names a part and what justifies it: a rule of
> [the concept](1-concept.md) or a test of [the
> tests](3-tests.md).

## 1 The Shape

The system is a Flask API with a Locust-based runner, an
SQS worker, a SQLite store and a React page. The API
records every job in one SQLite file. In queue mode it
sends the job to SQS and a worker runs it; in simple
mode the API runs Locust itself.

## 2 Backend (`code/`)

| Where | What it does | Justified by |
|-------|--------------|--------------|
| `app.py` | the routes; a shared check of users, spawn rate and duration before anything is recorded; JSON error handlers for `404` and `500` | contract routes, limits; T0.*, T3.16, T3.17 |
| `analysis/sanity_check.py` | accepts only http(s) URLs with a host; tries HEAD, then GET, then POST; any answer below 500 means reachable | L1; T2.1 to T2.6 |
| `analysis/type_detector.py` | classifies by content type and path hints; looks for `/sitemap.xml`; follows a sitemap index one level, dropping assets, capped at 20 pages | T2.7 to T2.16 |
| `analysis/edge_cases.py` | from one sample builds missing, null, wrong-type, boundary and attack cases; each case is a deep copy with one field changed; a bool is checked before an int | L2; T1.1 to T1.12 |
| `load_test_runner.py` | starts Locust as a subprocess with `--headless` and `--csv`; stores the CSV when done; marks timeout after duration plus 60 seconds; runs locally in a thread or sends to the queue; splits large runs | L4, L5; T3.1 to T3.9 |
| `SPLIT_THRESHOLD = 50`, `MAX_CHILD_JOBS = 20` | child count is the smaller of 20 and the request divided by 50, rounded up; the remainder is spread one user at a time | L4; T3.3, T3.4 |
| `stats_aggregator.py` | merges child CSVs: sums counts and throughput, takes minimum and maximum, weights averages and percentiles by request count, adds an Aggregated row | L6; T3.13 to T3.15 |
| `job_store.py` | SQLite with two tables, `jobs` and `job_groups`; a lock around writes; `init_db` is safe to run repeatedly | L7; job and group shapes |
| `sqs_client.py` | send, long-poll receive (10 seconds) and delete for the queue; skips messages that are not valid JSON | queue message contract |
| `worker.py` | loop: receive one message, run the job, delete the message | queue mode |
| `locust_tests/dynamic_locustfile.py` | each simulated user waits 0.5 to 2 seconds, then POSTs a randomly chosen edge case, named by its label | L2 |
| `locust_tests/website_locustfile.py` | each simulated user waits 0.5 to 2 seconds, then GETs a random page path | L2 |

## 3 Frontend (`code/frontend/`)

| Where | What it does |
|-------|--------------|
| `src/App.jsx` | the whole flow in one page: analyze the URL, enter a sample body, pick edge cases, set users, spawn rate and duration, start a run, poll the status every 2 seconds, list past runs, compare two |
| `src/components/AnalyzeSummary.jsx` | shows reachable, status code, classification and sitemap found |
| `src/components/LoadTestConfig.jsx` | the three number inputs, clamped to the contract limits |
| `src/components/WebsiteLoadTestSection.jsx` | the website run button and a note on whether a sitemap will be used |
| `src/components/JobResultSummary.jsx` | a per-edge-case table plus the aggregated row, and the error when a run failed |
| `src/components/CompareView.jsx` | two runs side by side, from their aggregated rows |
| `src/utils/parseStatsCsv.js` | turns the stats CSV text into rows |

The page reaches the backend through one constant,
`BASE`, at the top of `App.jsx`. It is currently a fixed
tunnel address, `<backend-url>`.

## 4 Deployment

| Part | Where it runs | Notes |
|------|---------------|-------|
| Frontend | Vercel | static Vite build; calls the backend at `BASE` |
| API | container from `Dockerfile` on K3s (`k8s/deployment.yaml`) | port 5000, NodePort service, database on a persistent volume at `/data/load_tests.db`, `USE_SQS=true` |
| Worker | container from `Dockerfile.worker` (`k8s/worker-deployment.yaml`) | shares the same volume; reads AWS credentials from the secret `aws-sqs-credentials` |
| Queue | AWS SQS, region `eu-north-1` | one queue of jobs |
| Pipeline | `.gitlab-ci.yml` | test, build, deploy of the API image |
| Docs | md-preview, built by a GitHub workflow on push | these pages live under `docs/features/` |

Two ways to run for a demo are in `code/RUNBOOK.md`:
simple mode (backend and frontend on one computer, no
AWS) and queue mode (adds the worker and SQS).

## 5 See Also

- [The tests](3-tests.md): what holds this to the
  contract
- [The refinement](5-refinement.md): next, what is still
  rough
