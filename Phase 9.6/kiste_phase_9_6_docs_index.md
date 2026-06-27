# Kiste Phase 9.6 Docs — Git-Native Update Engine with QC/QA Review Gates

Status: Phase documentation index  
Phase: `9.6`  
Theme: Git-native update, review validation, QC/QA evidence, and safe mutation gates

---

## 1. Phase 9.6 Goal

Phase 9.6 makes Kiste safe enough to update repositories through Git.

The phase introduces a Git-native update engine where all file changes are planned, reviewed, validated, quality-checked, and only then applied through controlled Git operations.

Core sentence:

```text
Phase 9.6 makes Git mutation reviewable, validated, testable, auditable, and QC/QA controlled.
```

---

## 2. Main User Story

```text
As a Kiste user,
I want Kiste to propose repository changes through Git,
so that every change is traceable, reviewable, reversible, and safe before it reaches a branch, commit, push, or pull request.
```

---

## 3. Phase 9.6 Lifecycle

```text
read
  -> inspect
  -> plan
  -> review / validate / QC gate
  -> deploy approved Git mutation
  -> monitor PR / CI / drift
```

Important rule:

```text
No Git mutation without review validation.
No mutating validation without approved SelfTestPlan.
No push or pull request without approved GitUpdatePlan.
```

---

## 4. Required Docs

Recommended Phase 9.6 docs:

```text
Phase 9.6/
  README.md
  git-native-update-engine.md
  qc-qa-review-gates.md
  minimum-core-capabilities.md
  git-update-plan-schema.md
  patchset-schema.md
  review-validation-schema.md
  self-test-plan-schema.md
  acceptance-tests.md
```

---

## 5. Existing Main Spec

The main spec should be:

```text
kiste_v0_9_6_git_update_qc_qa_review_gates_spec.md
```

It covers:

```text
GitUpdatePlan
PatchSet
BranchPlan
CommitPlan
PullRequestPlan
RollbackPlan
ReviewValidationReport
SelfTestPlan
SelfTestReport
ApprovalRecord
QC reports
QA evidence
Traceability matrix
Quality gate decision model
Acceptance criteria
```

---

## 6. Minimum Capabilities for Phase 9.6

Phase 9.6 needs these minimum capabilities:

```text
workspace.read
repo.discover
git.state_read
schema.validate
capability.registry_read
capability.resolve
policy.read
policy.validate
graph.build
plan.generate
review.validate
validate.plan
audit.write
output.write
monitor.local
```

To mutate Git locally:

```text
git.patchset
git.branch_create
git.commit_create
```

To support PR workflow:

```text
git.push_branch
git.pull_request_create
git.status_read
```

For QC/QA:

```text
qa.process_check
qa.traceability_check
qa.approval_workflow_check
qa.evidence_required
qa.audit_trail

qc.patchset_check
qc.diff_check
qc.file_boundary_check
qc.secret_check
qc.branch_policy_check
qc.commit_policy_check
qc.pr_policy_check
qc.rollback_check
qc.test_result_check
```

For validation:

```text
validate.git_update_plan
validate.patchset
validate.branch_plan
validate.commit_plan
validate.pull_request_plan
validate.rollback_plan
validate.secret_ref
validate.file_boundary
validate.branch_policy
validate.commit_policy
```

For review:

```text
review.diff_summary
review.risk_score
review.policy_check
review.blocker_report
review.warning_report
review.decision_record
```

For self-test:

```text
selftest.patch_apply
selftest.scratch_worktree
selftest.test_branch
selftest.test_commit
selftest.draft_pr
selftest.ci_probe
selftest.cleanup
```

---

## 7. Non-Mutating vs Mutating Review Checks

### Non-mutating checks

These can run directly during review:

```text
schema validation
PatchSet validation
file boundary check
secret reference check
branch policy check
commit message check
PR policy check
rollback plan check
traceability check
approval scope check
```

### Mutating checks

These require an approved SelfTestPlan:

```text
apply patch in scratch worktree
create temporary branch
create test commit
push temporary branch
open draft PR
trigger CI workflow
run integration test
clean up temporary branch
```

Rule:

```text
If QC changes Git, CI, cloud, runtime, or external systems, it must be handled as an approved SelfTestPlan.
```

---

## 8. QC/QA Artifacts

Kiste should write:

```text
.kiste/qc/
  patchset-qc-report.json
  diff-qc-report.json
  file-boundary-report.json
  secret-check-report.json
  rollback-check-report.json
  test-result-report.json

.kiste/qa/
  process-compliance-report.json
  traceability-matrix.json
  approval-record.json
  evidence-index.json
  audit-trail.json

.kiste/review/
  review-decision.json
  blocker-report.json
  warning-report.json
  risk-report.json

.kiste/selftest/
  git-self-test-plan.json
  self-test-report.json
```

---

## 9. Phase 9.6 Acceptance Criteria

Phase 9.6 is accepted only if:

```text
1. Kiste can read Git state.
2. Kiste can produce a PatchSet.
3. Kiste can produce a GitUpdatePlan.
4. Kiste can validate a PatchSet.
5. Kiste can validate a GitUpdatePlan.
6. Kiste can detect file-boundary violations.
7. Kiste can detect plaintext secret risk.
8. Kiste can block direct main mutation.
9. Kiste can produce QC reports.
10. Kiste can produce QA evidence.
11. Kiste can produce a traceability matrix.
12. Kiste can record a review decision.
13. Kiste can separate non-mutating checks from mutating self-tests.
14. Mutating self-tests require approved SelfTestPlan.
15. Kiste can create branch/commit only after quality gate passes.
16. Kiste can monitor PR/CI/Git drift after mutation.
```

---

## 10. Final Rule

```text
Phase 9.6 makes Git updates QC/QA controlled.

QA ensures the process is repeatable, traceable, policy-compliant, and auditable.

QC checks the actual PatchSet, GitUpdatePlan, BranchPlan, CommitPlan, PullRequestPlan, RollbackPlan, and self-test evidence.

Non-mutating checks can run directly during review.

Mutating checks must become approved SelfTestPlans.

No Git mutation is allowed unless the quality gate passes.
```
