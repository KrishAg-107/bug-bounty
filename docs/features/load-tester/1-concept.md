---
abstract: |
  Step 1 of the load tester: what it is for, who uses
  it, and the seven rules it keeps, before any code. A
  tester enters a URL; the system checks it, prepares
  the requests, runs a load test of the chosen size and
  shows the numbers; and what it will not do.
date: 2026-10-10
keywords:
- load-testing
- locust
- concept
kind: explanation
sources:
- code/app.py
status: draft
subtitle: Step 1, the concept
title: "Load Tester: the Concept"
version: v0.1.0
---

> [!NOTE]
> This concept is subject to refinement during the
> development process. Until [the
> contract](2-contract.md) exists, and wherever it is
> silent, this page is the source of truth.

## 1 The Goal

**A tester learns how an application behaves under load
and under bad input,** before and while real traffic is
sent to it. The tester enters a URL. The system checks
it, decides whether it is an API or a website, prepares
the requests to send (hostile request bodies for an API,
real page paths for a website), runs a load test of the
chosen size, and shows the numbers. Two runs can be
compared side by side.

## 2 The People

| Who         | What they do                                   |
|-------------|------------------------------------------------|
| Tester      | Uses the web page: enters a URL, picks edge cases, starts a run, reads results |
| API (Flask) | Validates the request, records the job, hands the work on, answers status queries |
| Worker      | Takes a job from the queue and runs Locust against the target |
| Target      | The site or API being tested; receives the load |
| Reviewer    | Reads these pages and the tests to judge how the system was built |

## 3 The Rules

Numbered, so [the tests](3-tests.md) can say which each
one checks:

- **L1 Nothing is sent to an unchecked target.** Not
  until its URL is well-formed and the server answers
- **L2 An API target is tested with edge-case POSTs
  the tester has chosen.** A website is tested with
  GETs of real pages from its sitemap, or of the URL
  path itself when there is no sitemap
- **L3 The limits:** 1 to 1000 users, spawn rate 1 to
  20 per second, duration 1 to 300 seconds. Anything
  outside is refused before anything is recorded
- **L4 Up to 50 users is one job.** Above 50 the
  request becomes a group of child jobs that add up to
  the users asked, none above 50
- **L5 A job moves queued, then running, then ends as
  completed, failed or timeout.** A group's status is
  the worst of its children
- **L6 A finished group reports one merged result:**
  counts and throughput are summed, minimum and maximum
  are taken across children
- **L7 Every run is stored,** and any two can be
  compared

## 4 What It Will Not Do

- **Send load to an unchecked target.** L1
- **Run beyond the limits.** L3
- **Retry a failed job.** [The
  refinement](5-refinement.md) lists it

## 5 See Also

- [The contract](2-contract.md): next, then [the
  tests](3-tests.md)
- [The concept](../../conduct/the-cycle/concept.md):
  what a concept holds, in general
