---
abstract: |
  Step 3 of the load tester: the 84 automated tests
  that check the contract, how to run them, what they
  stand on, what each group asks and expects, and what
  they do not cover.
date: 2026-10-10
keywords:
- load-testing
- tests
- pytest
kind: reference
sources:
- code/tests/
status: draft
subtitle: Step 3, the tests
title: "Load Tester: the Tests"
version: v0.1.0
---

> [!NOTE]
> The tests follow from [the concept](1-concept.md);
> every rule has at least one.

## 1 How to Run

84 automated tests check the contract, and all 84 pass.
They never touch the internet or AWS: the network and
the queue are stood in, and each run uses a temporary
database. Expect `84 passed`:

```
cd code
pip install -r requirements.txt pytest
python -m pytest -q
```

The tests live in `code/tests/`. Each test function name
starts with its ID, for example `test_T3_3_...`.

## 2 What the Tests Are Built On

| Piece | Role |
|-------|------|
| `tests/conftest.py` | points the database and results folder at a temporary directory and gives every test a Flask test client |
| `pytest.ini` | tells pytest to look only in `code/tests/` |
| stand-ins | network calls (`requests`) and queue calls (`send_job_message`) are replaced, so no test needs a live server |
| `code/scripts/` | the two manual probe scripts against a real site; kept, but not run by pytest |

## 3 Which Rule Each Group Checks

| Group | Checks |
|-------|--------|
| T0 | the service answers |
| T1 | L2: edge cases for an API target |
| T2 | L1 and L2: checking a target, telling an API from a website |
| T3 | L3 to L7: limits, splitting, statuses, merging, storage and comparison |

## 4 T0: the Service Answers

| ID | Question | Expected |
|----|----------|----------|
| T0.1 | `/health`? | `200` `{"status": "ok"}` |
| T0.2 | `/api/health`? | the same answer |
| T0.3 | An unknown route? | `404` `{"error": "not_found"}` |

## 5 T1: Edge Case Generation (17 tests)

| ID | Question | Expected |
|----|----------|----------|
| T1.1 to T1.2 | Missing and null, every field? | one case per field with that field removed, one with it null |
| T1.3 | A string field? | missing, null, wrong type, empty, very long, 3 SQL injection, 2 XSS; ten labels, no more |
| T1.4 | A number field? | missing, null, wrong type, very large, negative, zero; no attacks |
| T1.5 | A bool field? | missing, null, wrong type; not treated as a number |
| T1.6 | A nested object or array? | missing and null only |
| T1.7 | Labels? | unique |
| T1.8 to T1.9 | What leaks? | the sample is never changed; no case equals the sample |
| T1.10 to T1.12 | Shape and repeat? | exactly the four keys; same input gives same output; empty sample gives `[]` |
| T1.13 to T1.14 | Route answers? | `total_cases` equals the list length; a missing, empty, list or string sample is refused |
| T1.15 to T1.17 | Selection? | exactly the chosen cases come back; an unknown label, an empty selection or a missing field is refused |

## 6 T2: Target Analysis (20 tests; Network Stood In)

| ID | Question | Expected |
|----|----------|----------|
| T2.1 to T2.2 | Malformed URL? | not well-formed; only http and https count |
| T2.3 | A POST-only endpoint? | HEAD `405`, GET `405`, POST `200` is reachable, checked with POST |
| T2.4 to T2.6 | Errors? | `503` everywhere or no connection is unreachable; `404` everywhere still counts as the server answering |
| T2.7 to T2.11 | Type detection? | JSON is api, HTML is website, no signal is unknown with confidence 0, an API-looking path alone is api, confidence never above 1 |
| T2.12 to T2.14 | Sitemap pages? | assets dropped, capped at 20 (or the given limit), malformed XML gives none |
| T2.15 to T2.16 | Sitemap index? | followed one level to real pages with no `.xml` paths; an index inside an index is not followed |
| T2.17 to T2.20 | Routes? | empty and malformed refused; unreachable carries the sanity result; sitemap present for a website and absent for an API |

## 7 T3: Load Test Dispatch (44 Test Cases; Queue Stood In)

| ID | Question | Expected |
|----|----------|----------|
| T3.1 | Exactly 50 users? | one job, one queue message, status `queued` |
| T3.2 | 51 users? | a group; child count equals messages sent |
| T3.3 | 51, 99, 100, 101, 237, 500, 999, 1000 users? | children add up to the request and none is above 50 |
| T3.4 to T3.6 | Fair and the same? | child sizes differ by at most 1; all carry the same target, cases, rate and duration; ids are distinct and belong to the group |
| T3.7 | Website job? | the message carries paths, not cases |
| T3.8 | Group status? | seven combinations give the worst-of result in the contract |
| T3.9 | Finished group? | one merged CSV, identical on the second read |
| T3.10 to T3.12 | Status lookups? | unknown id is none and `404`; a group id is answered |
| T3.13 to T3.15 | Merging? | counts and throughput summed, minimum and maximum across children, an Aggregated row, merging nothing gives an empty aggregate |
| T3.16 | Bad input? | users 0 or 1001, spawn 0 or 21, duration 0 or 301, text for a number, bad or empty URL, empty or non-list cases: all `400` and nothing queued |
| T3.17 to T3.18 | Edges and shape? | 1000, 20 and 300 exactly are accepted; the `202` answer has the contract's keys |
| T3.19 to T3.20 | Website fallback? | no sitemap uses the URL's own path, or `/` for a bare host |
| T3.21 | Compare? | a missing id is `400`; unknown ids are `404` |

## 8 Not Covered by Automated Tests

- The React frontend
- The real SQS queue and the worker loop
- Locust itself and real load against a live target
- The Kubernetes manifests and the CI pipeline

## 9 See Also

- [The contract](2-contract.md): what these tests hold
  the system to
- [The implementation](4-implementation.md): next
