---
abstract: |
  Step 2 of the load tester: what other parts may rely
  on. The HTTP routes, the limits, the job and queue
  shapes, and the configuration. Nothing here says how
  it is done.
date: 2026-10-10
keywords:
- load-testing
- contract
- routes
- limits
kind: reference
sources:
- code/app.py
status: draft
subtitle: Step 2, the contract
title: "Load Tester: the Contract"
version: v0.1.0
---

> [!NOTE]
> A released contract is not changed in place: add
> beside it. Where [the concept](1-concept.md) and this
> page disagree, this page wins once it exists.

## 1 Conventions

- Requests and responses are JSON
- Every error is `{"error": "..."}`. `400` means the
  caller sent something wrong, `404` an unknown route or
  id, `500` is `{"error": "internal_server_error",
  "detail": "..."}`
- An unknown route answers `404` with
  `{"error": "not_found"}`

## 2 Routes

| Route | In | Out on success | Refused with `400` when |
|-------|----|----------------|-------------------------|
| `GET /health` and `GET /api/health` | - | `200` `{"status": "ok"}` | never |
| `POST /api/sanity-check` | `{"url"}` | `200` sanity result | url missing, or malformed (the sanity result with `valid_format: false`) |
| `POST /api/analyze` | `{"url"}` | `200` `{"sanity", "type_detection"}`, plus `"sitemap"` only when classified website | url empty, malformed or unreachable (body carries `"sanity"`) |
| `POST /api/generate-edge-cases` | `{"sample_input": {...}}` | `200` `{"total_cases", "cases"}` | sample_input missing, empty or not an object |
| `POST /api/confirm-selection` | `{"sample_input", "selected_labels": [...]}` | `200` `{"confirmed_count", "confirmed_cases"}` | either field missing or empty, or a label that does not exist |
| `POST /api/start-load-test` | `{"url", "confirmed_cases": [...], "users"?, "spawn_rate"?, "duration_seconds"?}` | `202` `{"job_id", "status": "started", "is_group"}` | url not http(s) with a host, cases missing or empty, a number not an integer or out of range |
| `POST /api/start-website-load-test` | `{"url", "sitemap_raw"?, "users"?, "spawn_rate"?, "duration_seconds"?}` | `202` as above, plus `"paths_used": [...]` | url not http(s) with a host, a number not an integer or out of range |
| `GET /api/load-test-status/{id}` | a job id or a group id | `200` job or group (§5) | never; an unknown id is `404` `job_id not found` |
| `GET /api/jobs` | nothing | `200` `{"jobs": [...], "job_groups": [...]}`, newest first, up to 50 of each | never |
| `POST /api/compare-jobs` | `{"job_id_a", "job_id_b"}` | `200` `{"job_a", "job_b"}` | either id missing; an unknown id is `404` |

## 3 Limits and Defaults

L3. A request outside these is refused with `400`, and
records nothing and queues nothing.

| Field | Default | Allowed |
|-------|---------|---------|
| `users` | 5 | integer, 1 to 1000 |
| `spawn_rate` (users started per second) | 1 | integer, 1 to 20 |
| `duration_seconds` | 30 | integer, 1 to 300 |

## 4 Edge Case

Every edge case has exactly `label`, `category`,
`description`, `payload`. The category is one of
`missing_field`, `null_value`, `type_mismatch`,
`boundary_value`, `known_attack`. A label names one case
only. The sample input is never modified.

## 5 Job and Group

- **Job:** `job_id`, `target_url`, `users`,
  `spawn_rate`, `duration_seconds`, `status`,
  `stats_csv`, `stdout`, `stderr`, `error`, `group_id`,
  `created_at`, `completed_at`
- **Group:** `group_id`, `target_url`, `job_type` (`api`
  or `website`), `users`, `spawn_rate`,
  `duration_seconds`, `child_count`, `status`,
  `aggregated_stats_csv`, `error`, `created_at`,
  `completed_at`. A status answer for a group adds
  `child_jobs`
- **Splitting (L4):** up to 50 users is one job. Above
  50 the request becomes a group of at most 20 children;
  child sizes add up to the request, none above 50, and
  differ by at most 1

## 6 Statuses

L5.

| Status | Meaning |
|--------|---------|
| `queued` | recorded, not yet picked up |
| `running` | Locust has started |
| `completed` | Locust finished and its statistics were stored |
| `failed` | Locust or its setup raised an error; `error` says why |
| `timeout` | Locust ran longer than `duration_seconds` plus 60 seconds |

A group is `failed` if any child failed, else `timeout`
if any timed out, else `completed` if all completed,
else `running` if any is running, else `queued`.

## 7 Queue Message

One per job, in queue mode only: `job_id`, `job_type`
(`api` or `website`), `target_url`, `users`,
`spawn_rate`, `duration_seconds`; plus `target_path` and
`cases` for `api`, or `paths` for `website`.

## 8 Configuration

| Setting | Meaning | Default |
|---------|---------|---------|
| `DB_PATH` | where the SQLite file lives | `load_tests.db` |
| `USE_SQS` | `true` sends jobs to the queue; anything else runs them in a local thread | off |
| `SQS_QUEUE_URL` | the queue to send to and read from | none |
| `AWS_DEFAULT_REGION` | queue region | `eu-north-1` |
| AWS access key and secret | credentials for the queue (never stored in the repository) | none |

## 9 See Also

- [The concept](1-concept.md): the rules these routes
  keep
- [The tests](3-tests.md): next, the questions the
  contract raises
