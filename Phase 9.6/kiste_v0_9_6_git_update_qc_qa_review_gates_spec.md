# Kiste v0.9.6 — Git-Native Update Engine with QC/QA Review Gates

Status: Canonical engineering specification  
Target implementation: `KisteBox/kiste-py`  
Spec repository: `KisteBox/kiste_spec`  
Release: `0.9.6`  
Phase: `9.6`

---

## 1. Release Theme

Kiste v0.9.6 introduces the Git-native update engine.

The Git-native update engine turns approved Kiste plans into safe branch, patch, commit, push, and pull-request workflows across one or more repositories.

This spec expands Phase 9.6 with QC/QA-friendly review gates.

One-sentence definition:

```text
Kiste v0.9.6 makes Git updates reviewable, validated, testable, traceable, and auditable before any branch, commit, push, or pull request is created.
```

---

## 2. Core Rule

```text
No Git mutation without QC/QA review validation.
No mutating validation without approved SelfTestPlan.
No push, pull request, or branch mutation without approved GitUpdatePlan.
```

The Git update engine must not behave like an automatic patcher.

It must behave like a controlled change system.

---

## 3. Lifecycle Position

The stable Kiste lifecycle remains:

```bash
kiste read
kiste inspect
kiste plan
kiste review
kiste deploy
kiste monitor
```

For Phase 9.6, the lifecycle becomes:

```text
Read     -> collect Git/repo state
Inspect  -> check repo safety and policy boundaries
Plan     -> create PatchSet and GitUpdatePlan
Review   -> run QC/QA validation gates and record approval
Deploy   -> execute approved Git mutation
Monitor  -> track PR, CI, branch, drift, and evidence
```

Short form:

```text
plan -> review/validate -> optional self-test -> approved Git mutation -> monitor
```

---

## 4. QC vs QA in Kiste

Kiste should use QC/QA language because platform engineering customers need evidence, sign-off, and traceability.

```text
QA = process assurance
QC = output checking
```

For Kiste:

```text
QA checks whether the process is safe, repeatable, traceable, policy-compliant, and auditable.

QC checks whether this specific PatchSet, GitUpdatePlan, BranchPlan, CommitPlan, PullRequestPlan, RollbackPlan, and evidence set is correct and safe.
```

---

## 5. Quality Gate Model

Phase 9.6 introduces quality gates:

```text
Gate 1: Read Gate
Gate 2: Inspect Gate
Gate 3: Plan Gate
Gate 4: Review / Validation Gate
Gate 5: Approved Git Mutation Gate
Gate 6: Monitor / Evidence Gate
```

Each gate produces evidence.

Each gate may produce:

```text
passed
passed_with_warnings
failed
blocked
needs_review
unsupported
```

Deploy must be blocked if required gates fail.

---

## 6. Phase 9.6 Main Objects

Phase 9.6 owns these Git update objects:

```text
PatchSet
GitUpdatePlan
RepoState
WorktreeState
BranchPlan
CommitPlan
PullRequestPlan
MergePolicy
RollbackPlan
ReviewValidationReport
QCReport
QAReport
TraceabilityMatrix
EvidenceIndex
SelfTestPlan
SelfTestReport
ApprovalRecord
ReviewDecision
MonitorReport
```

---

## 7. Capability Groups Added in Phase 9.6

Phase 9.6 adds or formalizes these capability groups:

```text
validate.*
review.*
qa.*
qc.*
selftest.*
readiness.*
approval.*
audit.*
```

These capabilities are used by `kiste inspect`, `kiste plan`, `kiste review`, `kiste deploy`, and `kiste monitor`.

The command surface stays stable.

The behavior expands through capabilities.

---

## 8. Git-Specific Validation Capabilities

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
validate.pr_policy
validate.capability_resolution
validate.workspace_access_policy
```

Validation is non-mutating by default.

Validation checks whether an object is structurally valid, policy-compatible, capability-compatible, and safe to pass to the next lifecycle stage.

---

## 9. QC Capabilities

QC capabilities check the actual output.

```text
qc.patchset_check
qc.diff_check
qc.file_boundary_check
qc.secret_check
qc.branch_policy_check
qc.commit_policy_check
qc.pr_policy_check
qc.rollback_check
qc.test_result_check
qc.plan_consistency_check
qc.generated_file_check
qc.multi_repo_order_check
```

Example QC questions:

```text
Does every planned file change map to an approved intent?
Does the PatchSet touch only allowed paths?
Does the PatchSet contain plaintext secrets?
Does the branch plan avoid protected branches?
Does the commit plan follow the commit policy?
Does the pull request plan include risk and rollback sections?
Does the rollback plan exist and match the PatchSet?
Does the multi-repo order respect spec -> core -> service -> GitOps sequencing?
```

---

## 10. QA Capabilities

QA capabilities check the process.

```text
qa.process_check
qa.traceability_check
qa.approval_workflow_check
qa.evidence_required
qa.audit_trail
qa.reapproval_required_check
qa.policy_compliance_check
qa.review_scope_check
qa.lifecycle_compliance_check
```

Example QA questions:

```text
Was the plan produced by Kiste lifecycle?
Was the plan reviewed before mutation?
Is the approval scope recorded?
Is there evidence for each required quality gate?
Would a plan change invalidate approval?
Can an auditor reconstruct why this change was made?
```

---

## 11. Review Capabilities

Review capabilities produce human- and machine-readable decision support.

```text
review.diff_summary
review.risk_score
review.policy_check
review.blocker_report
review.warning_report
review.rollback_check
review.evidence_summary
review.approval_summary
```

Review is not only approve/reject.

Review is a capability-bearing lifecycle stage.

It validates evidence, runs quality gates, records decisions, and determines whether Git mutation is allowed.

---

## 12. Readiness Capabilities

Readiness capabilities decide whether the plan may move forward.

```text
readiness.git_ready
readiness.pr_ready
readiness.deploy_ready
readiness.blocker_report
readiness.warning_report
```

For Phase 9.6, `readiness.git_ready` means:

```text
The GitUpdatePlan has passed required validation, QC, QA, review, and approval checks and is safe to execute within its approved scope.
```

---

## 13. Non-Mutating Review Validation

Non-mutating validation can run directly during `kiste review`.

It only reads and checks.

It does not change:

```text
working tree
Git index
local branches
remote branches
pull requests
CI systems
cloud resources
runtime systems
external systems
```

Allowed non-mutating validation:

```text
schema validation
PatchSet validation
file boundary check
secret reference check
branch policy check
commit policy check
pull request policy check
rollback plan check
traceability check
approval scope check
risk score
blocker/warning report generation
```

Example command behavior:

```text
kiste review .kiste/plans/git-update-plan.json
  -> validate.git_update_plan
  -> validate.patchset
  -> qc.diff_check
  -> qc.secret_check
  -> qa.traceability_check
  -> review.risk_score
  -> readiness.git_ready
```

---

## 14. Mutating Review Validation

Some validation is really a self-test because it changes something.

Examples:

```text
apply patch to a scratch worktree
create temporary branch
create test commit
push temporary branch
open draft PR
trigger CI workflow
create temporary GitHub check
run integration test branch
clean up temporary branch
```

These are useful, but they mutate Git, remote Git, CI, or external systems.

They must not run silently inside normal validation.

---

## 15. Mutating Validation Rule

```text
If review validation mutates Git, remote Git, CI, cloud, runtime, or external systems, it must be represented as an approved SelfTestPlan.
```

This means:

```text
Non-mutating validation can run directly.
Mutating validation requires its own reviewed and approved self-test plan.
```

Bad:

```text
kiste review automatically creates a temporary branch and pushes it.
```

Good:

```text
kiste review detects that a temporary branch test is useful.
kiste review generates GitSelfTestPlan.
User approves the self-test.
Kiste runs the approved self-test.
Evidence is recorded.
```

---

## 16. GitSelfTestPlan

Example:

```json
{
  "schema": "kiste.git-self-test-plan/v0.9.6",
  "target": "review",
  "mutation": true,
  "requires_review": true,
  "requires_approved_plan": true,
  "actions": [
    {
      "id": "git-test-001",
      "capability": "selftest.patch_apply",
      "operation": "apply_patch_to_scratch_worktree"
    },
    {
      "id": "git-test-002",
      "capability": "selftest.test_branch",
      "operation": "create_temporary_branch"
    },
    {
      "id": "git-test-003",
      "capability": "selftest.ci_probe",
      "operation": "trigger_ci_on_test_branch"
    },
    {
      "id": "git-test-004",
      "capability": "selftest.cleanup",
      "operation": "delete_temporary_branch"
    }
  ]
}
```

---

## 17. Self-Test Capabilities

```text
selftest.patch_apply
selftest.scratch_worktree
selftest.test_branch
selftest.test_commit
selftest.draft_pr
selftest.ci_probe
selftest.integration_probe
selftest.cleanup
```

Self-tests may be:

```text
non_mutating
mutating
```

Mutating self-tests require approval.

---

## 18. Stress Test Position

Stress testing is a high-intensity self-test.

For Phase 9.6 Git engine, stress testing may include:

```text
large PatchSet application
multi-repo update ordering
CI fan-out behavior
many-file diff review
branch conflict simulation
merge conflict simulation
rollback simulation
```

Stress test capabilities may include:

```text
stress.patchset_size
stress.multi_repo_order
stress.ci_fanout
stress.merge_conflict
stress.rollback_simulation
```

If stress testing creates branches, pushes commits, triggers CI, or modifies external systems, it requires approved SelfTestPlan.

---

## 19. GitUpdatePlan Shape

Example:

```json
{
  "schema": "kiste.git-update-plan/v0.9.6",
  "intent": "self-improve-kiste",
  "status": "ready-for-review",
  "mode": "pull_request",
  "repos": [],
  "patchsets": [],
  "branch_plan": {},
  "commit_plan": {},
  "pull_request_plan": {},
  "rollback_plan": {},
  "quality_gates": [
    "validate.git_update_plan",
    "qc.patchset_check",
    "qa.traceability_check",
    "review.risk_score",
    "readiness.git_ready"
  ],
  "requires_review": true,
  "requires_approved_plan": true
}
```

---

## 20. Traceability Matrix

A traceability matrix maps requirements or intents to files, validations, and evidence.

Example:

```json
{
  "schema": "kiste.traceability-matrix/v0.9.6",
  "intent": "self-improve-kiste",
  "plan": ".kiste/plans/git-update-plan.json",
  "items": [
    {
      "requirement": "REQ-001",
      "description": "Update Git update engine spec",
      "changed_files": [
        "Phase 9.6/git-update-engine.md"
      ],
      "validation": [
        "validate.patchset",
        "qc.diff_check",
        "qa.traceability_check"
      ],
      "evidence": [
        ".kiste/qc/patchset-qc-report.json"
      ],
      "status": "passed"
    }
  ]
}
```

---

## 21. Evidence Index

Evidence must be indexed.

Example:

```json
{
  "schema": "kiste.evidence-index/v0.9.6",
  "plan": ".kiste/plans/git-update-plan.json",
  "evidence": [
    {
      "type": "qc_report",
      "path": ".kiste/qc/patchset-qc-report.json",
      "status": "passed"
    },
    {
      "type": "qa_report",
      "path": ".kiste/qa/traceability-matrix.json",
      "status": "passed"
    },
    {
      "type": "review_report",
      "path": ".kiste/review/risk-report.json",
      "status": "passed_with_warnings"
    }
  ]
}
```

---

## 22. Review Decision as QA Sign-Off

Review decision becomes a QA sign-off object.

Example:

```json
{
  "schema": "kiste.review-decision/v0.9.6",
  "decision": "approved_for_git_update",
  "quality_gate": "passed",
  "qa_status": "compliant",
  "qc_status": "passed",
  "approved_plan": ".kiste/plans/git-update-plan.json",
  "evidence": [
    ".kiste/qc/patchset-qc-report.json",
    ".kiste/qa/traceability-matrix.json",
    ".kiste/review/risk-report.json"
  ],
  "blockers": [],
  "warnings": [],
  "approval": {
    "scope": "git_update",
    "target": "feature_branch_only",
    "reviewer": "human-or-policy",
    "requires_reapproval_if_plan_changes": true
  }
}
```

---

## 23. Reapproval Rule

```text
Any change to an approved GitUpdatePlan invalidates the approval.
```

Approval must be reissued if any of the following change:

```text
PatchSet content
changed file list
branch plan
commit plan
pull request plan
rollback plan
policy result
risk score
SelfTestPlan result
quality gate result
```

---

## 24. Output Layout

Phase 9.6 should write:

```text
.kiste/git/
  repo-state.json
  worktree-state.json
  provider-state.json
  patchset.json
  branch-plan.json
  commit-plan.json
  pull-request-plan.json
  rollback-plan.json

.kiste/plans/
  git-update-plan.json
  git-self-test-plan.json

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
  review-validation-report.json
  review-decision.json
  blocker-report.json
  warning-report.json
  risk-report.json
  readiness-report.json

.kiste/selftest/
  self-test-report.json
  scratch-worktree-report.json
  ci-probe-report.json
  cleanup-report.json

.kiste/monitor/
  pr-status.json
  ci-status.json
  branch-status.json
  conflict-report.json
  drift-report.json
```

---

## 25. Lifecycle Integration

### Read

Read collects:

```text
repo path
provider type
remote URL
default branch
current branch
HEAD
dirty/staged/untracked state
tags
submodules
open PR references
multi-repo graph
provider capabilities
```

Output:

```text
RepoState
WorktreeState
ProviderState
```

### Inspect

Inspect validates:

```text
clean worktree requirement
remote availability
branch protection information if available
file boundary policy
secret risk
large file risk
multi-repo update order
WorkspaceAccessPolicy
```

Output:

```text
InspectReport
ValidationReport
QC precheck report
```

### Plan

Plan creates:

```text
PatchSet
GitUpdatePlan
BranchPlan
CommitPlan
PullRequestPlan
RollbackPlan
QualityGate requirements
```

### Review

Review runs:

```text
non-mutating validation
QC checks
QA checks
risk scoring
traceability matrix generation
evidence indexing
readiness gate
```

Review may generate:

```text
GitSelfTestPlan
```

if mutating validation is needed.

### Deploy

Deploy executes only approved GitUpdatePlan.

Allowed operations after approval:

```text
create branch
apply patch
commit
push branch
open pull request
update pull request
```

Default forbidden operations:

```text
direct main push
force push
merge
unreviewed commit
unapproved branch mutation
```

### Monitor

Monitor tracks:

```text
branch status
PR status
CI status
review status
merge conflict status
post-merge drift
quality evidence state
```

---

## 26. Safety Rules

Required safety rules:

```text
No Git mutation during read.
No Git mutation during inspect.
No Git mutation during non-mutating review validation.
No branch creation without approved GitUpdatePlan or SelfTestPlan.
No push without approved plan.
No pull request creation without approved plan.
No direct main push.
No force push by default.
No commit without quality gate.
No mutation if QC fails.
No mutation if QA fails.
No approval reuse after plan changes.
No secret file commit.
No patch outside allowed paths.
No mutating self-test without approved SelfTestPlan.
```

---

## 27. Acceptance Criteria

Phase 9.6 is accepted only if:

```text
1. Every GitUpdatePlan has a QC report.
2. Every PatchSet has a diff check.
3. Every file change maps to an intent or requirement.
4. Every plan has a rollback section.
5. Every secret check produces evidence.
6. Every policy violation becomes a blocker or warning.
7. Every approval records scope and evidence.
8. Any plan change after approval invalidates the approval.
9. Mutating self-tests require their own approved SelfTestPlan.
10. Deploy refuses plans without passed QC/QA gate.
11. Non-mutating validation can run during review.
12. Mutating validation cannot run during review unless represented by approved SelfTestPlan.
13. Evidence index is generated.
14. Traceability matrix is generated.
15. ReviewDecision is generated before deploy.
16. Deploy executes only approved GitUpdatePlan.
17. Monitor records PR/CI/branch status after deploy.
```

---

## 28. Non-Goals

Phase 9.6 does not include:

```text
Kubernetes integration
Pulumi integration
Ansible integration
boto3 execution
GitHub Actions execution lane
AI capability taxonomy
central unit registry
full self-hosted manager implementation
full plugin marketplace
```

Those belong to later phases.

Phase 9.6 focuses on Git-native update safety.

---

## 29. Final Rule

```text
Phase 9.6 makes Git updates QC/QA controlled.

QA ensures the process is repeatable, traceable, policy-compliant, and auditable.

QC checks the actual PatchSet, GitUpdatePlan, BranchPlan, CommitPlan, PullRequestPlan, RollbackPlan, and self-test evidence.

Non-mutating checks can run directly during review.

Mutating checks must become approved SelfTestPlans.

No Git mutation is allowed unless the quality gate passes.
```
