---
abstract: |
  Step 5 of the load tester: what is still rough, and
  why each item matters. Each is a change to make the
  application more streamlined and easier to use.
date: 2026-10-10
keywords:
- load-testing
- refinement
kind: explanation
sources:
- code/worker.py
- code/load_test_runner.py
status: draft
subtitle: Step 5, the refinement
title: "Load Tester: the Refinement"
version: v0.1.0
---

> [!NOTE]
> Each refinement says which step it refines, and is
> made by a commit of its own, not by editing an earlier
> page in place.

## 1 What Is Still Rough

These issues have to be addressed to improve the user
experience and make the application more streamlined:

| Issue | Why it matters |
|-------|----------------|
| The worker deletes a queue message even when the job failed | a failed job is never retried |
| An error before Locust starts is only printed by the worker | the job stays `queued` forever |
| Two workers would share one SQLite file | writes would contend; one worker is the supported set-up |
| Merged percentiles are weighted averages | they are an approximation, not exact percentiles |
| The pipeline builds and deploys only the API image | the worker image and deployment are not part of it |
| The frontend's backend address is typed into `App.jsx` | a tunnel address that changes means editing code and redeploying |
| `RUNBOOK.md` contains an AWS account id and queue URL | remove it if the repository is public |
| `code/README.md` is still the template text | it says nothing about this project |

## 2 See Also

- [The implementation](4-implementation.md): the parts
  these refine
- [The concept](1-concept.md): what the application will
  not do, including retrying a failed job (L5)
