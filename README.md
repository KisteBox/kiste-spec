# Kiste Main Specification Archive

This repository is the main specification source of truth that all Kiste projects must follow. It contains the versioned product and architecture specifications for the Kiste ecosystem. Kiste is a deployment-oriented developer tool that starts with Docker repository inspection and grows into Kubernetes generation, GitOps, library/CLI workflows, self-hosting, modular cloud architecture, static plugins, and expanded diagnostics.

## How to use this repository

- Use this repository as the canonical reference before implementing or changing any Kiste project.
- Start with [Phase 1](Phase%201/spec.md) for the initial repo scanner and token inspector scope.
- Use the phase table below to find the specification that matches the capability or release version you need.
- Treat the PDF files as the canonical specs for later phases unless a Markdown source is present beside them.
- Keep future specifications in phase-numbered folders using descriptive filenames.

## Specification index

| Phase | Focus | Spec files |
| --- | --- | --- |
| Phase 1 | Repo scanner, token inspector, readiness checks, and command reference. | [`spec.md`](Phase%201/spec.md), [`cmd.md`](Phase%201/cmd.md) |
| Phase 2 | Next-phase Kiste specification. | [`kiste_phase2_spec.pdf`](Phase%202/kiste_phase2_spec.pdf) |
| Phase 3 | Phase 3 Kiste specification. | [`kiste_phase3_spec.pdf`](Phase%203/kiste_phase3_spec.pdf) |
| Phase 4 | GitOps and SOPS specification. | [`kiste_phase4_gitops_sops_spec.pdf`](Phase%204/kiste_phase4_gitops_sops_spec.pdf) |
| Phase 4.5 | Review specification. | [`kiste_phase4_5_review_spec.pdf`](Phase%204.5/kiste_phase4_5_review_spec.pdf) |
| Phase 5 | Library and CLI specification. | [`kiste_phase5_library_cli_spec.pdf`](Phase%205/kiste_phase5_library_cli_spec.pdf) |
| Phase 6 | Repository, serverless, and Docker Hub specification. | [`kiste_phase6_repo_serverless_dockerhub_spec.pdf`](Phase%206/kiste_phase6_repo_serverless_dockerhub_spec.pdf) |
| Phase 7 | Data repository specification. | [`kiste_phase7_data_repo_spec.pdf`](Phase%207/kiste_phase7_data_repo_spec.pdf) |
| Phase 8 | Self-hosting, modular cloud, and Argo CD specification. | [`kiste_phase8_self_hosting_modular_cloud_argocd_spec.pdf`](Phase%208/kiste_phase8_self_hosting_modular_cloud_argocd_spec.pdf) |
| Phase 8.5 | Release stabilization and data/blob storage revision. | [`kiste_phase8_5_release_stabilization_spec.pdf`](Phase%208.5/kiste_phase8_5_release_stabilization_spec.pdf), [`kiste_phase8_5_1_data_blob_storage_revision_spec.pdf`](Phase%208.5/kiste_phase8_5_1_data_blob_storage_revision_spec.pdf) |
| Phase 9 / v0.9.0 | Static plugins and split-ready architecture. | [`kiste_phase9_v0_9_0_static_plugins_split_ready_architecture_spec.pdf`](Phase%209/kiste_phase9_v0_9_0_static_plugins_split_ready_architecture_spec.pdf) |
| Phase 9.1 / v0.9.1 | Standard config and contract updates. | [`kiste_v0_9_1_standard_config_contract_spec.pdf`](Phase%209.1/kiste_v0_9_1_standard_config_contract_spec.pdf), [`kiste_v0_9_1_standard_contract_updated_spec.pdf`](Phase%209.1/kiste_v0_9_1_standard_contract_updated_spec.pdf) |
| Phase 9.2 / v0.9.2 | Expanded doctor diagnostics. | [`kiste_v0_9_2_expanded_doctor_spec.pdf`](Phase%209.2/kiste_v0_9_2_expanded_doctor_spec.pdf) |

## Phase progression

1. **Inspect** a Docker-based repository and report deployment readiness.
2. **Generate and manage** deployment artifacts and supporting configuration.
3. **Operationalize** Kubernetes, GitOps, secrets, data, and hosting workflows.
4. **Stabilize and extend** the architecture through release contracts, plugins, and doctor diagnostics.

## Repository maintenance

When adding or updating specs:

1. Keep this repository aligned as the source of truth for every Kiste implementation repository.
2. Add new files to the appropriate phase folder.
3. Use filenames that include the Kiste phase or release version.
4. Update the specification index in this README.
5. Prefer Markdown for editable source specs and PDF for published artifacts.
