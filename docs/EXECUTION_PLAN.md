# LoadSentry execution plan

Status: **queued; planning only; not implemented**. Prepared 2026-10-22 from [the portfolio research](RESEARCH_BRIEF.md). This directory is the canonical preparation copy for the eventual implementation repository [dl-loadsentry](https://github.com/123ssk321/dl-loadsentry). The URL is an intended destination, not evidence of a release.

## Product and activation

LoadSentry will help an energy consultant or small facilities team turn hourly electricity-meter CSV files into an auditable next-day forecast and a short review queue. The job is to find files needing repair and meter-days deserving investigation. The hypothesis is reduced review effort; no user demand, accuracy or savings is established.

Activate only after the portfolio owner selects this project. The entire portfolio has **16 hours/week and one active project**. Relative month 1 starts at activation, not repository creation. All other projects remain queued. The first functional public local release is targeted at **month 6, week 26**; a local alpha at month 9 and a product decision at month 12 follow. Interviews, paid data, cloud spend, new manual labels and conference participation are not prerequisites. Hardware is the existing GTX 1650 Ti 4GB / 16GB RAM laptop; CPU operation is required.

The initial supported contract is hourly energy in kWh, one explicit timezone per file, and meter IDs with stable units. A configured CSV mapping may identify column names, but the software must not guess units or timezones. Power in kW requires a separately explicit interval integration rule and is otherwise rejected. Negative readings require a declared export-meter policy; the initial policy quarantines them. A forecast has 24 hourly targets at q=0.1, 0.5 and 0.9. It is advisory, with no equipment diagnosis, controls, promised savings or guaranteed anomaly labels.

## Architecture and contracts

Implement one Python package and CLI first. Modules: acquisition/provenance → bounded CSV parser → quality checks → causal windows/features → interchangeable forecasters → calibration/scoring → findings → static local report/export. A small SQLite-backed loopback application is the month-nine extension; avoid cloud services, message brokers and microservices.

Canonical reading: meter_id, timestamp_utc, local_timezone, interval_seconds, energy_kwh, source_file_hash, source_row and quality_flags. Preserve original timestamp/unit fields in local provenance. UTC fixes ordering; local calendar features handle DST explicitly. Duplicate timestamps, absent hours and unit changes have recorded dispositions rather than silent repair. Fit imputers/scalers only on training history; carry missingness masks. The default forecast needs 168 hourly historical values with declared minimum valid coverage. Missing-history or excessive missingness yields abstention. Zero consumption is a valid observation unless evidence says otherwise.

Forecast artifact: run_id, meter_id, origin, target timestamps, quantiles, model/config/input hashes, training cutoff and calibration version. Forecast creation may use only data available by origin. Calendar variables are allowed; realized future weather is excluded. A retrospective evaluation must simulate that cutoff. Findings appear only after relevant observations arrive; they contain type (data_quality or forecast_deviation), evidence period, observed/expected values, uncertainty, rank, reason and review status. A flagged deviation is not a verified fault. Quality-failed intervals are quarantined from confident deviation claims.

Month-nine local routes will include POST /jobs, GET /jobs/{id}, GET /findings, PATCH /findings/{id}, GET /exports/{run_id}, DELETE /workspaces/{id} and health status. The service binds loopback, validates origin and a per-session capability, uses server-owned paths and limits one worker plus four queued jobs. It exposes explicit failed/cancelled/interrupted states. This is a single-operator local tool, not a multi-tenant hosted service. Review edits use revisions to reject stale writes; user notes never silently become training data. Export JSON preserves values; CSV neutralizes spreadsheet formulas in string fields.

## Data admission and reproducibility

Public data must be admitted for a named use before becoming an experiment input. Store URL, exact revision, retrieval date, file checksum, license text/hash, attribution, modifications, original rights holder, intended training/evaluation/redistribution/commercial uses and approval/exclusion reason. Code licenses do not resolve data rights. Existing public annotations are allowed; newly assembled or hand-labeled datasets are excluded.

BDG2 is the candidate primary corpus. Its repository license has a 4.0 heading alongside legacy wording, so this plan **does not resolve it by assumption**. LS-002 must establish exact file coverage and permitted uses; unresolved assets remain excluded. BuildingsBench is a protocol/reference candidate, not permission to reuse every underlying dataset. Do not download its large pretraining corpus or assume pretrained weight rights. If no usable public source clears the bounded rights gate, continue on independently generated meter series and label the release synthetic-only. The reduced release proves software operation, not real-building forecasting utility; do not count its model metrics as industrial validation.

Select an initial 50-meter cohort by a fixed metadata/completeness rule, expanding to at most 100 only after the laptop smoke test. Keep storage at a proposed 10GB project ceiling and stop before download/extraction exceeds it. Streaming or subset extraction should avoid loading all data into RAM. No data is fetched during ordinary package import or CI.

Generate seeded independent schedules, seasonal load and noise controls with exact known fields. For admitted public series, paired corruption tests may insert missing hours, duplicate rows, timezone shifts, stale readings and unit changes; preserve unmodified source and all generator parameters. A transformed sample inherits source restrictions. Synthetic controls test mechanics and known errors; neither transforms nor fictional loads prove real waste/fault detection.

Use a fixed chronological split: first 60% training, next 15% model selection, next 10% interval calibration, final 15% sealed test, with exact dates frozen after cohort eligibility checks. Keep all windows' targets inside their partition. Reserve 20% of eligible buildings, not individual sibling meters, for transfer evaluation; these buildings never enter training, tuning or calibration. Separate historical context available at deployment from fitted state. For transfer, permit only documented past context at forecast origin and use training-fitted scaling rules. Freeze per-meter minimum lengths and exclusions before model results. Report near-constant series and MASE zero denominators explicitly. Test data is opened at release checkpoints; changes prompted by test failures require transparent versioning and cannot be disguised as fresh holdout performance.

## Models, measurement and cancellation gates

Compare previous-day/week seasonal naive, a bounded seasonal statistical regression and LightGBM to a compact causal TCN. All share origin/horizon and partitions. The neural proposal is 168 past hours plus masks/calendar features to 24 ordered quantile triples; it has no assumed pretrained weights. At most four validation configurations per fitted family and three seeds for the selected TCN limit tuning. Early stopping uses model-selection data. Empirical baseline residual intervals and neural calibration use the separate calibration period only.

LS-003 first measures synthetic smoke training, CPU inference, driver compatibility, wall time and peak memory. Proposed acceptance ceilings are 3.5GB VRAM, 12GB process RAM and under 30 minutes for a smoke run. Actual model/cohort choices follow those measurements. Full training is initially capped at four wall-clock hours per run; exceeding it triggers smaller channels/history/cohort, fewer trials or CPU baseline default. No benchmark has been run during planning.

Every evaluation includes commit, dependency lock, data manifest/generator hashes, seeds, exclusions, hardware, configuration, failed attempts and raw predictions. Report MAE, meaningful MASE, pinball loss, 80% interval coverage and width, per-horizon/per-meter distributions, seen-building versus transfer results, queue volume and end-to-end resources. Bootstrap meter/day blocks where supported; correlated hours are not independent observations. No fault precision/recall is reported without existing appropriate labels. Generated corruption detection has its own labeled suite.

| Gate | Proposed criterion | If unmet |
| --- | --- | --- |
| Internal month 3 | Folder → validation → baseline → report runs with typed invalid-input handling | Narrow CSV format/cohort; park if basic workflow remains unusable |
| Neural retention | At least 5% median per-meter pinball improvement over best non-neural model, block interval for improvement above zero, no material coverage collapse, and resource limits met | Keep reproducible TCN experiment; deploy best baseline and stop neural tuning |
| Calibration | Nominal 80% coverage within 75–85% on sufficiently large supported aggregate slices; publish all small/sparse slice counts and interval widths | Mark intervals unreliable/experimental, reduce supported cohort or abstain; never force a pass by retuning on test |
| Month 6 functional release | Clean install, offline CLI, quality report, forecast evidence, review/export, one neural experiment, two baseline families, frozen results and rights-cleared artifacts | Use baseline-default and narrower inputs; publish honest limitations; do not call a static report a functional release |
| Local resource target | 50-meter daily inference/report within 5 minutes CPU, peak RAM under 12GB; report cold start separately | Lower supported cohort; benchmark again before promising scale |
| Simulated utility | At least 20 generated tasks, ≥25% median review-time reduction versus spreadsheet procedure, no increase in final known-issue errors | Improve/narrow review workflow or park; builder-only n=1 cannot establish demand |
| Month 12 product path | Repeated workflow benefit, rights clarity and optional external repeat use (target ≥2 willing users completing ≥3 sessions each) | Maintain experimental OSS or park; absent users is unvalidated, not failed software |

These are decision targets, not measured achievements. A neural failure does not invalidate a useful evaluator/review tool. Conversely, good benchmark scores do not justify paid deployment. The authoritative month-12 outcome can be stop.

## Budget and execution order

| Milestone / phase | Relative window | Issues | Work h | Reserve h | Scheduled h |
| --- | --- | --- | ---: | ---: | ---: |
| LS-M1: Scope, rights and laptop feasibility / definition | Weeks 1–7, months 1–2 | LS-001–LS-006 | 62 | 24 | 86 |
| LS-M2: Internal end-to-end workflow / internal-slice | Weeks 8–13, months 2–3 | LS-007–LS-012 | 68 | 16 | 84 |
| LS-M3: Models and frozen evaluation / model-evaluation | Weeks 14–21, months 4–5 | LS-013–LS-018 | 72 | 32 | 104 |
| LS-M4: First functional public release / public-release | Weeks 22–26, month 6 | LS-019–LS-024 | 64 | 16 | 80 |
| LS-M5: Local alpha foundations / alpha-core | Weeks 27–32, months 7–8 | LS-025–LS-030 | 64 | 16 | 80 |
| LS-M6: Usable local alpha / alpha-release | Weeks 33–39, months 8–9 | LS-031–LS-036 | 60 | 16 | 76 |
| LS-M7: Product and operational validation / product-validation | Weeks 40–46, months 10–11 | LS-037–LS-042 | 62 | 12 | 74 |
| LS-M8: Final decision and handoff / handoff | Weeks 47–52, months 11–12 | LS-043–LS-048 | 56 | 12 | 68 |
| Total | 52-week decision horizon | 48 leaf issues | 508 | 144 | 652 |

The identity is **sum(issue.estimate_hours) + reserve_hours = total_hours = 652**. Reserve is **22.09%** of this plan. The year ceiling is 832 hours (52 × 16); this plan deliberately commits less, leaving 180 hours of unallocated calendar capacity for breaks, observation and scheduling flexibility, not hidden implementation. Each milestone's scheduled work plus reserve fits its listed weeks at no more than 16 hours/week. Full weekly utilization is not assumed.

Through the month-six release, work is **266h + 88h reserve = 354h**, below the 416h half-year ceiling. The 62h difference is unallocated capacity; it is not another contingency pool or a promised feature allowance. Release is scheduled at week 26 even if individual work finishes sooner. Post-release work plus reserve is 298h through the month-12 decision. Calendar waiting/observation adds no claimed labor hours, and repeated operational/feedback work must fit the corresponding issue estimate.


The proposed 10GB data ceiling and compute ceilings are resource limits, not purchases. No hosted deployment or cloud account is needed. Public release means a public code/package artifact with reproducible local execution; it does not require a live web service.

Every issue in [planning/backlog.json](../planning/backlog.json) is a leaf of 8–12 hours including implementation, focused testing, review evidence and issue documentation. Dependencies are a DAG and previous milestone gates must pass before later work starts. Every issue has concrete outputs, acceptance checks and a blocker procedure. Future output paths in the backlog describe work to create after activation, not files that already exist.

Reserve is shared contingency for unexpected data/license, hardware, parser, UI or packaging rework; it is not unlisted feature work. Record actual hours and reserve consumption through the portfolio tracking process. Unused reserve is slack, not hours worked. On estimate overrun, capture evidence and narrow/split remaining work within the same cap; escalate scope, rights and architecture changes with a concrete options/impact brief. If month 6 is threatened, drop optional watching, expanded cohort, transfer enhancements and neural deployment before dropping provenance, honest metrics, review/export or offline reproduction. If the minimum cannot fit, make an explicit re-scope/stop decision.

## Operations, privacy and adoption

Inputs can reveal occupancy. The default is local processing with no telemetry, no network during inference, explicit deletion and redacted operational logs containing random IDs, durations and error codes. Local workspaces are retained until the operator deletes them in v0.1, with clear UI/docs disclosure; month-nine/twelve retention adds a configurable default 30-day live TTL and startup/regular cleanup. Source originals outside the managed workspace are not deleted by an app action. Backups require their own documented lifetime and restore/delete procedure. Do not claim that live deletion erases external backups.

Verify bounded parsing, path traversal rejection, raw-data-free logs, network denial, CSV injection handling, resource limits and complete deletion. Test Windows clean installation, CPU fallback, interrupted jobs, migrations, restore and model/config rollback on generated workspaces. Builds contain code and rights-cleared examples only; model rights are checked independently. Updating a dependency requires a focused numerical/schema/privacy regression and an available previous lock.

Optional owner-arranged trials use customer-supplied existing readings with explicit processing permission and no hand labeling. No outreach is authorized by this plan. If unavailable, run generated task rehearsals and explicitly retain the domain-gap and demand uncertainty. Record distinct user count, repeated review sessions, successful exports, unresolved findings, completion time and support effort; downloads/stars are not adoption. A possible paid path is installation/support or a repeatable file adapter, contingent on evidence and rights, with no facility-control promise. At month 12, document continue/narrow/park, actual budget and future maintenance proposal; ongoing maintenance is not automatic additional capacity.

## Primary sources checked 2026-10-22

- [BDG2 official license](https://github.com/buds-lab/building-data-genome-project-2/blob/master/LICENSE): inspected current inconsistent heading/body; approval of exact downloaded resources remains unresolved.
- [BDG2 official repository](https://github.com/buds-lab/building-data-genome-project-2): candidate source and data provenance entry point.
- [BuildingsBench official repository](https://github.com/NatLabRockies/BuildingsBench): forecasting benchmark and reference comparators; software rights do not authorize underlying data. No downloaded corpus or model is approved by this link.
- [Source portfolio research](RESEARCH_BRIEF.md), prepared 2026-10-19: product hypothesis and original constraints. This execution plan makes budgets, gates and implementation sequence concrete.

Recheck resources at admission/release. Package versions, rights, hardware performance and adoption remain unverified until their issues execute.

