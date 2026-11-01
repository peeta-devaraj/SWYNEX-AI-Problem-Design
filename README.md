# Task 1 — AI Problem Design

**Project:** IssueRoute — Human-reviewed GitHub issue classification  
**Prepared for:** Devaraj Peeta | Artificial Intelligence Internship, SWYNEX  
**Date:** 1 November 2026  
**Status:** Design deliverable. Data collection, training, and evaluation are planned; no performance results are claimed.

## 1. Problem and intended user

Small open-source maintainers and student software teams receive issue reports with inconsistent wording. Manually deciding whether a report describes a defect, requests functionality, or asks for help adds repetitive triage work.

IssueRoute will recommend one of three categories from an English issue title and description: **bug**, **feature request**, or **question**. If the input is ambiguous, outside scope, or below a validation-selected confidence threshold, the interface will display **needs human review**. A maintainer confirms or corrects the recommendation. The initial problem is category recommendation, not automatic priority assignment or code repair.

**Problem statement:** Can a small text classifier categorize eligible GitHub issues more reliably than simple baselines while retaining a human decision for uncertain reports?

This is a practical learning project using an established task, rather than a claim of a novel research contribution. The user need is a hypothesis to validate with maintainers.

## 2. Inputs, outputs, and examples

Input consists only of issue title and body. Output includes a proposed category, an explicitly labelled model score, and a review flag. A score is not a guarantee of correctness.

| Illustrative input | Intended output | Reason |
| --- | --- | --- |
| “Export crashes when the output folder is missing” | Bug | Existing behavior fails |
| “Please add CSV export” | Feature request | Requests new functionality |
| “How do I change the export folder?” | Question | Requests usage guidance |
| “Export is confusing and sometimes fails; please redesign it” | Needs human review | Mixed intent |

These examples are authored illustrations, not collected records or evaluation results.

## 3. Data source and collection plan

Use a small, manually reviewed sample of public English GitHub issues accessed through GitHub's documented REST issues API [1]. Target 600 eligible records across three repositories, approximately 200 per class where feasible. Candidate repositories will be chosen after auditing category availability, issue-template dependence, reuse conditions, and text quality; no repository or dataset has yet been collected.

Select repositories with active issue histories and enough examples in all three categories. Pilot annotation on 30 records before full collection. If a repository lacks questions or feature requests, replace it rather than assuming its existing labels are complete. If 600 suitable records cannot be assembled, document the smaller sample and treat conclusions as exploratory.

Exclude pull requests, empty reports, spam, non-English records, and records that cannot be assigned a single intent. The issues endpoint can return pull requests, so explicitly exclude records containing the pull_request field [1]. Keep excluded-record counts and reasons.

Suggested internal fields: repository, issue number, source URL, creation date, cleaned title, cleaned body, assigned class, ambiguity flag, annotation notes, and duplicate-group ID. Maintain source provenance and collection date. Do not collect author profiles, emails, or unrelated comments.

Public availability does not automatically settle reuse rights. Review applicable repository and platform terms before collecting or redistributing text. Publish collection instructions, provenance, and aggregate statistics; publish raw text only where redistribution permission is established. Redact personal information and credentials before processing. Avoid private repository data and external model APIs in the MVP.

## 4. Annotation and quality control

Label by the report's primary intent: bug means unexpected behavior in an existing feature; feature request means new or changed functionality; question means a request for explanation or usage assistance. A requested repair of broken functionality remains a bug. Mixed intent and insufficient context go to manual review and are not forced into a training class.

Existing repository labels are candidate weak labels, not ground truth. Manually inspect every selected record. Obtain an independent second annotation for at least 20% if a reviewer is available, record agreement and disagreements, then adjudicate. If working alone, re-annotate that subset after a delay and disclose the absence of independent review.

Strip explicit category tags and template headings that reveal the target label. Remove URLs, user mentions, and duplicate boilerplate consistently, retaining useful technical descriptions. Report performance separately on reports that retain useful content after cleaning.

## 5. Proposed approach and baselines

Start with a majority-class baseline and a fixed keyword baseline. Train a scikit-learn pipeline using TF-IDF text features and logistic regression [2,3]. This offers a CPU-based implementation with inspectable features and no paid inference requirement. It remains a proposed choice; evaluation determines whether it is adequate.

Fit preprocessing statistics and vocabulary only on training data. Use class weighting only if justified by the training distribution. Compare a small predeclared set of configurations on validation data, such as word unigrams versus unigrams plus bigrams and regularization C values of 0.1, 1, and 10.

Select the abstention threshold using validation data and report the resulting coverage. Do not present uncalibrated probabilities as measured reliability. If confidence quality is poor, validate calibration using training-only cross-validation or rely on a conservative review workflow.

## 6. Evaluation design

Deduplicate exact and near-duplicate reports before splitting, and keep each duplicate group within one partition. Split approximately 60/20/20 into training, validation, and locked test sets, stratified where possible. At a 600-record target this is approximately 360/120/120.

Primary evaluation measures ordinary within-repository performance. A separate leave-one-repository-out experiment tests transfer to unseen repositories; report it separately because its training size and difficulty differ. If a class is absent in a held-out repository, disclose that limitation rather than treating the experiment as a full three-class test.

Freeze preprocessing, hyperparameters, and review threshold before opening the test results. Evaluate all baselines on the identical test records.

| Measure | Proposed acceptance criterion |
| --- | --- |
| Primary macro F1 before abstention | At least 0.80 and at least 0.10 absolute above the stronger baseline |
| Recall for each of the three classes | At least 0.70 before abstention |
| Accuracy among accepted recommendations | At least 0.90 at coverage of at least 0.60 |
| Review behavior | Every low-score or invalid input visibly requests review |
| Local inference latency | p95 below 1 second over 100 warm predictions on a documented laptop CPU |

These are targets, not achieved results. Macro F1 weights classes equally. Report precision, recall, F1, support, a confusion matrix, full-test accuracy, accepted-case accuracy, coverage, and rejection rate. Calculate coverage as accepted predictions divided by all eligible test examples. Abstentions must not disappear from the report.

Include bootstrap intervals where feasible and acknowledge that a small test sample cannot establish production reliability. Inspect at least 20 errors, or all errors if fewer, and categorize ambiguity, missing context, vocabulary shift, and annotation issues. Report failed targets honestly rather than tuning on test data.

For usefulness, seek feedback from three volunteer developers on ten reports each, using counterbalanced manual and assisted conditions. Record time and corrections as exploratory observations. Do not claim productivity gains before conducting this study.

## 7. Scope, constraints, and failure handling

MVP scope: English text only; three categories; a small offline dataset; local training and inference; manual paste input; visible review flag; no automatic GitHub changes. Limit input to 10,000 characters and explain truncation or reject overlong input consistently. Reject blank input and warn when insufficient detail prevents meaningful classification.

Out of scope: autonomous issue edits, urgency or severity prediction, code execution, code repair, multilingual support, organization deployment, and large-model training. Treat issue text as data; never execute embedded commands. A missed bug or incorrect feature label must remain reversible through maintainer correction.

Main risks are noisy labels, class imbalance, template leakage, duplicate leakage, domain shift, and overstated confidence. Mitigations are manual review, per-class reporting, cleaning, grouped splits, repository-held-out testing, and abstention. Budget assumption: an existing laptop and open-source Python tooling. Zero paid API spend is a design constraint, not a measured total project cost.

## 8. Path through the remaining internship tasks

| Stage | Planned deliverable |
| --- | --- |
| Task 1: problem design | This specification, data plan, constraints, and measurable acceptance criteria |
| Task 2: model or API integration | Reproducible dataset preparation, trained baseline pipeline, saved model, and evaluation report |
| Task 3: intelligent feature | Validated confidence-based review and correction workflow |
| Task 4: final AI application | Small interface, documented setup, limitations, test results, and demonstration |

The later stages are a proposed progression; their detailed requirements remain locked on the dashboard and must be checked when available.

## 9. Reproducibility plan

Record Python and package versions, source selection, collection dates, annotation guide, random seed 42, split manifest, preprocessing rules, hyperparameters, and evaluation commands. Keep an untouched test manifest. Do not commit tokens or personal data.

Suggested later repository layout: README.md, docs/problem-design.md, docs/data-card.md, src/, tests/, requirements.txt, and reports/. Task 1 requires documentation; a trained model is not claimed or needed for this design submission.

## 10. Sources

1. GitHub Docs, REST API endpoints for issues: https://docs.github.com/en/rest/issues/issues — authoritative source for issue retrieval and the pull-request distinction.
2. scikit-learn, text feature extraction documentation source: https://github.com/scikit-learn/scikit-learn/blob/main/doc/modules/feature_extraction.rst — explanation of TF-IDF features for document classification.
3. scikit-learn, LogisticRegression reference: https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html — estimator API and configuration.

Sources checked on 1 November 2026. Project targets and dataset sizes are design decisions, not facts asserted by these sources.
