# Product-focused deep-learning project plan

**Prepared 19 October 2026.** This report revises LoadSentry, QuietShift and RiskLens FX around the constraints you gave: an MSc and working Python/PyTorch experience, 16 hours/week, a laptop with GTX 1650 Ti 4GB VRAM and 16GB RAM, no budget for cloud or data, and no manually assembled or hand-labelled datasets. Your goal is product potential; conference submissions are out of scope.

The recommendation is **LoadSentry** as the most balanced product-shaped project. A credible public first release should come after **six months** (about 416 work hours at your pace), with a working end-to-end private alpha by month 9 and a product-readiness decision by month 12. The earlier four-week deadline is removed. Three months is the earliest reasonable public prototype; less time would mostly demonstrate a notebook, not a product workflow.

## What “no manual dataset creation” means here

All three plans use public, versioned data downloaded by a script, and/or synthetic examples created deterministically by code. They do not ask you to collect or label audio, meter readings, invoices, or financial observations by hand. You can still inspect failure cases and give synthetic examples to a willing reviewer. Public benchmark examples establish technical behavior; they cannot establish that a real buyer will pay for or trust a tool.

Synthetic data must have a declared generator, seed, schema and limitations. It can verify parsing, transformations, unit tests and known failure modes. It cannot establish that synthetic noise, defects, invoices or financial sequences represent real operating conditions. A future product still needs permissioned, customer-supplied data for its final validation. The customer can provide data to a private trial; you do not need to create the dataset manually.

### Product opportunity overview

Deep-learning models are readily available. Product value is more likely to come from fitting an existing workflow: taking the right input, making uncertainty visible, routing ambiguous cases for review, and integrating with a user's existing tools. Each project should therefore build a vertical slice around an actual job, not stop at a metric chart.

The risks are equally practical. Established energy-monitoring, predictive-maintenance, and financial-risk products already exist. Benchmark licensing may prevent the data from being used in a paid service. No interviews, integrations, or customer trials have occurred. Product-market fit is unknown for all three options.

## Project 1 — LoadSentry: meter-to-action energy review

### Product and customer

LoadSentry is a local-first tool for an energy consultant or small facilities team. It imports a folder of electricity-meter CSV files, checks data quality, forecasts the next day, and ranks unexplained deviations for human review. It does not issue equipment diagnoses or automate building controls.

The user pain hypothesis is that small teams spend time checking inconsistent meter files and sorting which buildings deserve attention. This is plausible, not confirmed. A generic energy dashboard is not differentiated. The product wedge is **reliable ingestion, an auditable forecast, uncertainty and a short, prioritized review list**.

### Public and synthetic data

- [Building Data Genome 2 (BDG2)](https://github.com/buds-lab/building-data-genome-project-2) and its [paper](https://pmc.ncbi.nlm.nih.gov/articles/PMC7591488/), published **27 October 2020**, provide hourly historical data for 3,053 meters across 1,636 non-residential buildings. Download a selected cohort using a script and save the exact files, license and checksums. The repository's current LICENSE is headed Attribution-ShareAlike 4.0 with inconsistent legacy text. Do not describe this dataset as MIT or bundle it in a commercial image until the actual downloaded files' terms are resolved.
- [BuildingsBench](https://github.com/NatLabRockies/BuildingsBench), associated with **NeurIPS 2023**, provides data loaders, real-building evaluation datasets and persistence/transformer comparisons. Its repository states the evaluation bundle is under 1 GB and the pretraining corpus is around 110 GB. The pretraining corpus is unnecessary here. Code and underlying datasets have separate licensing.
- Generate synthetic data-quality cases from downloaded time series: missing hours, duplicate rows, timezone offsets, unit changes, stale readings and sudden gaps. Preserve the unmodified source as the control. Use the generator for resilience tests, not as a substitute for real faults.

No hand-built dataset is required. The data downloader, cohort rule, synthetic corruption script and time splits should run from one command.

### Deep-learning system and engineering

Pipeline: scripted public-data download → manifest and license capture → timestamp/unit/schema validation → chronological split → naive forecast → compact PyTorch temporal convolutional network (TCN) producing 24 hourly quantiles → calibration and error analysis → local report and review queue. Add a folder-watcher or file-drop interface after the CLI works.

Baseline first: previous-week same hour, then a seasonal statistical model and LightGBM. The TCN is a candidate, not a required winner. Fit all transformations on training periods. Keep windows and future targets within their chronological partition. Report future periods on known meters separately from transfer to meters excluded from training. Never use realized future weather in an “operational” forecast.

For each flagged day, display the input period, forecast interval, observed reading, data-quality checks and an explicit reason such as “missing history” or “forecast deviation.” The product should be able to say “insufficient evidence.” A meter deviation is not evidence of a broken device, waste, misconduct or savings opportunity.

### Evaluation and success criteria

Use MAE/MASE where meaningful, quantile loss, interval coverage and width, errors by meter and forecast horizon, alert volume, and CPU/GPU latency and memory. Include the simple baselines. A score averaged across meters can hide a poor experience for individual sites, so show distributions.

Suggested technical gate at month 6: clean install and one-command reproduction; 50–100 downloaded meters if the local storage budget allows; tests for leakage, units and invalid timestamps; at least two baseline families and one compact neural model; a local output that helps a reviewer distinguish data-quality issues from forecast deviations. Measure actual memory and runtime on your laptop before promising them.

### Product path, competition and risks

Potential users: independent energy consultants, small property operators, university facilities and sustainability teams. Established analytics products already combine data with equipment context and fault workflows. The prototype has no right to claim parity. A later product could sell per-site onboarding, a private installation or recurring reports. Open-source the benchmark adapter and keep a paid layer for integrations only if repeated customer use reveals a shared need.

Privacy risk: occupancy patterns can be inferred from meter data. Keep the first version local, minimize retention, document deletion and avoid sending readings to hosted services. Operational risk: a wrong alert can waste staff time or encourage unsafe action. Keep recommendations advisory and show thresholds and evidence.

**Commercial assessment: uncertain, potentially moderate as a narrow service.** The opportunity is not verified without a buyer, data connection and repeated workflow. Treat the business as a hypothesis until a potential user agrees to test the local alpha and confirms that the review report changes their work.

### 12-month plan at 16 hours/week

| Month | Deliverable | Decision |
|---|---|---|
| 1–2 | Automated data retrieval, license manifest, schema checker, naive forecast and split tests | Is the public data usable and the task definition correct? |
| 3 | Internal vertical slice: CSV folder → validation → baseline forecast → static report | Is the end-to-end workflow already useful to inspect? |
| 4–6 | Compact TCN, transfer tests, generated data faults, resource profile, public reproducible release | Do neural forecasts improve any relevant outcome over simple baselines? |
| 7–9 | Local alpha with file drop, review queue, export, versioned configuration and error recovery | Can a reviewer complete a task without reading source code? |
| 10–12 | Optional user trial using their data, privacy/retention controls, installer and product decision | Did a real workflow recur, and would someone pay for support or integration? |

Target the first **public** release around month 6, not month 3. By month 3, aim for a private internal proof that determines whether to continue. If a model improvement is absent, retain the stronger simpler baseline; the value may lie in the workflow.

### CV and portfolio evidence

Produce a versioned data manifest, tests, small trained model, inference package, benchmark report, local demonstration video, product screenshots and limitations page. Example CV wording after measurement: “Built a local-first meter ingestion and 24-hour quantile forecasting tool; evaluated [N] meters with chronological and held-out-meter tests; reduced [metric] by [measured amount] versus [baseline] at [latency] on [hardware].”

Interview topics: temporal leakage, site transfer, calibration, data quality, choosing a non-neural model, uncertainty UX, deletion/privacy choices and how to turn an offline experiment into a maintainable local application.

### Verdict

**Best default choice.** It fits your skills, has public data and a coherent product workflow. Product potential remains uncertain until a buyer validates the review process.

## Project 2 — QuietShift: acoustic self-test for machine-monitoring integrators

### Product and customer

QuietShift helps an ML engineer or system integrator test whether a machine-sound anomaly detector remains useful across specified operating and noise conditions. The first product is an **offline commissioning and regression test tool**: choose a detector, point it at normal reference clips and a public evaluation set, receive threshold, failure and resource reports. The product does not identify root cause or predict component failure.

This is a safer product boundary than selling an unvalidated predictor. Industrial machine monitoring is a real problem area, but demand for a separate test tool has not been confirmed.

### Public and synthetic data

- [DCASE 2026 Task 2](https://dcase.community/challenge2026/task-first-shot-unsupervised-anomalous-sound-detection-for-machine-condition-monitoring) supplies a normal-only anomaly-detection protocol under domain changes with near/far audio. The challenge's **15 June 2026** deadline has passed. Reproducing it now is a benchmark study, not challenge participation.
- The [DCASE 2026 development data](https://zenodo.org/records/19336329) was released **1 April 2026**, corrected **8 April**, and is listed as CC BY-NC-SA 4.0. Start with one machine category (Fan) and automate its download. Check the corrected ToyCar package if you use it later. Do not use this noncommercial benchmark data as the dataset in a paid service without obtaining rights.
- Generate controlled transformations automatically from source clips: additive environmental-noise mixtures at documented levels, gain changes, equalization and time shifts. Preserve clean/paired references and identify each corruption. These transformations test known perturbations; they do not recreate a factory or prove field reliability.

### Deep-learning system and product workflow

Implement log-mel feature extraction, a PCA reconstruction baseline and a compact trainable PyTorch autoencoder. A small local batch evaluator accepts a model adapter and reports metrics on fixed public splits. Choose the threshold from normal calibration data only. Later add the second channel only after verifying the paired audio and task protocol support the comparison.

Product output: a saved test plan, versioned report, per-condition false-positive/recall counts, threshold provenance, selected audio failure examples, runtime and memory. A developer can use this before deploying or updating a detector. Keep all audio local.

### Evaluation and success criteria

Report AUROC and official partial-AUC conventions, confusion counts at a fixed calibration threshold, source/target results, performance under each generated perturbation, latency and peak resource use. Use grouping that prevents clips from the same recording/session leaking across partitions. Do not estimate false alarms per operating hour from short balanced clips. Do not calibrate using anomalous test labels.

By month 6, release a public benchmark runner, one compact trained detector, at least one non-neural baseline and reproducible domain-sliced results. At month 9, aim for a local package that evaluates another compatible detector. The challenge closed before this research; future DCASE timing is unverified and is not part of the plan.

### Product path, competition and risks

Potential users are audio-ML teams, equipment integrators and researchers who need to validate detector versions. Existing DCASE reports and libraries provide baselines; Anomalib is strong in visual anomaly detection, while audio tooling remains a narrower niche. Product differentiation would need to be in usable test plans and local integration, not the autoencoder.

Industrial recordings can contain speech or confidential facility information. The prototype uses public audio and local processing. Customer data later needs explicit permission, retention controls and security. False negatives could affect maintenance decisions; do not position the tool as a safety certification.

**Commercial assessment: uncertain.** A paid commissioning/test service is possible only after integrators validate the workflow and the product has data rights for the intended use. A public benchmark app alone is unlikely to prove a market.

### 12-month plan

| Month | Deliverable |
|---|---|
| 1–2 | Automated DCASE subset retrieval, data manifest, audio sanity checks, PCA baseline |
| 3 | Internal local runner for one category with calibration and held-out scoring |
| 4–6 | Compact autoencoder, generated shift tests, resource report and reproducible public release |
| 7–9 | Model adapter interface, report export, packaging and invalid-input handling |
| 10–12 | Optional private pilot under explicit data permission; decide between OSS tool, service or stop |

First public release: **month 6**. Hardware and runtime estimates must be measured rather than inferred from parameter count.

### CV and verdict

Show the sound pipeline, trained model, fixed splits, threshold tests, error audio, reproducible package and local demonstration. This is the strongest of the three for research-style robustness work and could serve audio-integrator workflows. It is the riskiest for product conversion because commercial customer data and operating feedback are essential later.

## Project 3 — RiskLens FX: sequential model-risk ledger

### Product and customer

RiskLens FX is a local tool for a quant researcher or risk analyst to compare probabilistic FX forecasts, register predictions before outcomes, and monitor calibration changes over time. It is **model-validation software**, not a trading system, investment recommendation or execution service.

The product hypothesis is that researchers benefit from a disciplined, auditable forecasting ledger with proper scoring and clear input timestamps. Existing open-source quant platforms and econometric libraries make a broad platform a poor target; keep the wedge narrow.

### Public and synthetic data

- The [ECB euro reference-rate page](https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/index.en.html) provides historical CSV/XML downloads. It is a public reference-rate series and the ECB discourages its use for transactions. Preserve the ECB's reuse conditions, attribution and any transformation disclosure.
- The historical download may not preserve what was available at every past forecast origin; archive downloaded snapshots and disclose revisions. Begin prospective forecasts after the daily publication time. They are reference observations, not bid/ask or executable prices.
- Generate synthetic return sequences with known volatility regimes, heavy tails, gaps and correlation using fixed-seed scripts. These are unit and calibration tests, never evidence about real markets.

### Deep-learning system and workflow

Build an automated ECB fetcher, timestamp/quote-direction checks, causal return features, fixed chronological folds, a small shared GRU outputting return quantiles, EWMA and Student-t GARCH comparators, and an append-only prediction ledger. Each forecast records origin timestamp, input hash, model version, target observation and quantiles before the outcome becomes available.

Start with one pair and one-step-ahead next-published-observation forecasting. Extend to a fixed cohort only after the single-pair pipeline is correct. Implement rolling calibration only after preserving a final untouched historical holdout and prospective test stream.

### Evaluation and success criteria

Use pinball loss, negative log likelihood where distributions are specified, interval coverage and width, tail-exceedance counts, time-block uncertainty and per-period results. Do not treat correlated pairs as independent samples. No Sharpe ratio/profit result is a success metric. By month 6, a user should be able to reproduce three model families from a fixed snapshot and inspect every issued prediction.

### Product path and risks

Potential users: independent quant researchers, MSc/PhD labs, model-validation teams. A paid path could be private data adapters, audit exports or scheduled validation, but there is no verified willingness to pay. Open-source the local evaluator first.

Reference-rate limitations, data revisions, look-ahead bias, overfitting from repeated model selection and user confusion about trading advice are the main risks. The UI must state the quote convention and publication timestamp. Do not make personalized recommendations or connect to brokerage execution as part of this project.

**Commercial assessment: low for the benchmark alone; uncertain for a validation service.** Strong fit if finance keeps you motivated, but weaker product evidence than a workflow tool with a recurring operational job.

### 12-month plan and verdict

Months 1–3: automatic data ingestion, one-pair baseline and ledger. Months 4–6: GRU/GARCH comparison and public release. Months 7–9: fixed multiyear cohort, calibration and reporting. Months 10–12: prospective forecast record and product decision. First public release: **month 6**.

This is the best finance-interest option and a strong software-evaluation project. It is not the best default product bet.

## Comparison and recommendation

| Dimension | LoadSentry | QuietShift | RiskLens FX |
|---|---|---|---|
| Main user job | Review meter data and forecast deviations | Test a sound detector before integration | Compare and audit sequential forecasts |
| Data creation | Automated public download + scripted corruptions | Automated public download + scripted audio transforms | Automated ECB download + simulated unit-test series |
| Product potential | Uncertain, perhaps moderate service | Uncertain, specialist service | Low/uncertain service |
| Solo feasibility at 16h/week | High | Medium | High technically |
| Technical stretch | Temporal validation + local workflow | DSP + normal-only detection + thresholds | Econometrics + calibration + audit ledger |
| First public release | Month 6 target | Month 6 target | Month 6 target |
| 12-month product milestone | Optional user trial | Optional integrator trial | Prospective ledger; buyer validation |
| Best reason to choose it | Broad applied ML/product engineering | Research depth and audio modality | Finance motivation |

**Default: LoadSentry.** Best overall balance of data access, task clarity and workflow potential. **Research alternative: QuietShift.** **Finance alternative: RiskLens.** Do not develop all three in parallel. Give one option a six-month public release and decide at month 12 whether a product-specific user trial justifies another six months.

## What counts as a product milestone

At 16 hours/week, six months is roughly 416 scheduled hours and twelve months is roughly 832, before breaks and applications. Use this to reach a product-quality prototype, not to promise a company launch. A public report, demo, automatic dataset pipeline, local installation and tested user workflow are evidence of a product-shaped tool. A product requires a recurring user problem, permission to process its data, acceptable error rates, support plan and evidence someone will adopt or pay for it.

You can avoid manually creating training data; you cannot replace customer validation with synthetic examples. If you still prefer no customer contact, keep the result open source and describe its commercial potential as unvalidated.

## Shared build and evaluation practices

For each repository: one-command dataset download with checksum; separate data/asset license inventory; versioned configs and splits; CPU smoke test; optional bounded GPU training; tests for leakage and malformed inputs; no raw third-party data bundled without permission; failure gallery; model card and limitations; local demo and screenshot/video; release notes. Use a small CI fixture generated automatically by code, never represented as benchmark evidence.

Use PyTorch and scikit-learn first, with pinned dependencies. Keep model sizes and input batches within measured 4GB VRAM. Profile early; CPU-only evaluation is acceptable. Avoid dependence on a paid inference API or a cloud dashboard.

Suggested weekly allocation: 8 hours building, 3 hours experiments/evaluation, 2 hours tests and packaging, 2 hours product workflow and documentation, 1 hour user/problem evidence. If you do not want user contact, use that hour to document assumptions and seek maintainer feedback—but accept the reduced product confidence.

## Product reality check

The available data are sufficient to build and evaluate the technical components automatically. They are not customer-specific. Synthetic perturbations test only the transformations deliberately encoded. Public benchmark results do not establish ROI, safe deployment, demand or willingness to pay.

The strongest portfolio outcome is a public product prototype with automatic data provenance, robust local inference, a defined human review step, meaningful baselines and measured limitations. Keep CV wording precise: “built and benchmarked an independent prototype on public/synthetic data,” not “deployed commercially” or “validated with customers” unless those later events actually happen.
