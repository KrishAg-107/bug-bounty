---
abstract: |
  The load tester: a Flask API, a Locust runner, an SQS
  worker and a React page, which check an API or a
  website and show how it behaves under load and under
  bad input. What it is, the five pages that document
  it, and what you need to run it. Built in the
  project's five steps.
date: 2026-10-10
keywords:
- load-testing
- locust
- flask
- cycle
kind: explanation
sources:
- code/app.py
- code/load_test_runner.py
- code/worker.py
status: draft
subtitle: Load testing an API or a website
title: Load Tester
version: v0.1.0
---

## 1 What It Is

- **An API, in Flask:** checks a URL, tells an API from
  a website, prepares the requests, validates the size
  of the run and records every job
- **A runner, with Locust:** sends the load, either in a
  local thread or through a queue to a worker, and
  splits large runs into child jobs of at most 50 users
- **A page, in React:** analyze a URL, pick edge cases,
  start a run, read the numbers, and compare two runs
  side by side
- **84 tests,** written against the contract, which
  never touch the internet or AWS

## 2 Five Steps

This feature follows [the project's
cycle](../../conduct/the-cycle/README.md), one page a
step. Read them in order.

1.  **[The concept](1-concept.md).** What the load
    tester is for, who uses it, and its seven rules
2.  **[The contract](2-contract.md).** The routes, the
    limits, the job and queue shapes and the
    configuration: what other parts rely on
3.  **[The tests](3-tests.md).** The rules as 84 tests,
    and what each group checks
4.  **[The implementation](4-implementation.md).** Each
    part, and the rule or test that justifies it
5.  **[The refinement](5-refinement.md).** What is still
    rough, and why each item matters

## 3 Before You Start

- Python 3 and `pip`, to run the backend and the tests:
  the steps are in [the tests](3-tests.md) §1
- To try it by hand, simple mode runs the backend and
  the page on one computer with no AWS; queue mode adds
  the worker and SQS

## 4 See Also

- [The cycle](../../conduct/the-cycle/README.md): the
  five steps in general
- [The project's conduct](../../conduct/README.md): how
  we work, and how we write it down
