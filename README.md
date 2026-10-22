# LoadSentry

**Planned product — queued, not implemented.** A planned local tool for energy consultants and small facilities teams: import hourly meter CSV files, validate quality, forecast the next day and review evidence-backed deviations.

The eventual implementation repository is [123ssk321/dl-loadsentry](https://github.com/123ssk321/dl-loadsentry). This preparation directory contains plans only; no application, trained model, benchmark or released package is claimed.

## Start here

- [Execution plan](docs/EXECUTION_PLAN.md): product contract, architecture, data/licensing gates, evaluation, release definition and operating boundaries.
- [Machine-readable backlog](planning/backlog.json): 48 dependency-ordered leaf issues, each 8–12h, with outputs, checks, review evidence and blocker procedure.
- [Authoritative research brief](docs/RESEARCH_BRIEF.md): source product hypothesis and portfolio constraints, copied by the portfolio coordinator.

## Schedule and limits

Activation is an explicit portfolio decision. Only one project runs at a time across the portfolio, with 16h/week total on the existing GTX 1650 Ti 4GB / 16GB RAM laptop. All dates are relative to activation. The first functional public local release targets month 6; local alpha month 9; continue/narrow/park decision month 12.

Budget: **508 task hours + 144 reserve = 652h**, including 22.09% reserve and below the 832h yearly cap. Month-six cumulative scheduled work plus reserve is **354h ≤416h**. There is no cloud/data budget and no manual recording, collection or labeling requirement. Actual laptop benchmarks precede committed model/cohort scope.

## Evidence and rights

BDG2 is a candidate, subject to resolving exact file rights and its inconsistent license wording. Independently generated meter series keep the software path executable if public data cannot be admitted. Forecast deviations are advisory, not equipment diagnoses or savings claims.

Model quality, resource performance, workflow savings and customer demand are unmeasured. Baselines can remain the default if the neural model fails its retention gate. Optional user trials can improve evidence; their absence does not block building a useful local release and must not be described as traction. No implementation or GitHub publication occurs merely by creating this plan.

