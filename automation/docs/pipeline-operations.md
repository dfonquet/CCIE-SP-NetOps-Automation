# CI/CD Pipeline Operations

This guide explains how the pipeline is expected to behave when deploying ISIS, SR-MPLS, BGP, or VRF changes to the lab.

## Where the Pipeline Runs

The pipeline runs from an external automation node or runner, not from the routers themselves. In this lab, the recommended runner is a Linux VM connected to the EVE-NG or CML management network.

The runner needs:

- Git access to clone this repository.
- Internet access during setup for Python packages and Ansible collections.
- SSH reachability to IOS XR and CSR management interfaces.
- Access to EVE-NG or CML if lab lifecycle automation is added later.

## Required Secrets

| Secret | Purpose | Notes |
| --- | --- | --- |
| `LAB_USERNAME` | Device login username | Mark as masked/protected in CI/CD |
| `LAB_PASSWORD` | Device login password | Mark as masked/protected in CI/CD |
| `NETCONF_PRIVATE_KEY` | Private key for NETCONF/SSH workflows | Prefer file-type secrets when supported |

For production, use an external secret manager when possible. It centralizes rotation, auditing, and access control instead of spreading credentials across individual CI/CD projects.

## Branch and Merge Rules

| Event | Behavior |
| --- | --- |
| Pull request / merge request | Validate YAML, lint, validate topology, render configs, dry-run, and generate diff only |
| Push to `main` | Allow manual deployment jobs |
| Other branches | No deployment jobs |

## Production-Style Pipeline Flow

```text
Commit
  |
Lint
  |
Validate topology
  |
Dry-run
  |
Generate diff
  |
Manual approval
  |
Deploy PE-1
  |
Validate ISIS / BGP / SR
  |
Deploy next router
  |
Post-check
  |
Evidence attached to CRQ
```

## Job Behavior and Error Handling

| Condition | Expected behavior |
| --- | --- |
| NETCONF or SSH timeout | Retry, because this is usually transient |
| Invalid configuration | Stop the pipeline |
| Non-critical documentation failure | Allow failure without blocking network validation |
| Deployment failure | Collect diagnostics, stop for review, then run a guarded rollback only after confirming the target commit/checkpoint |
| Production deployment | Require manual approval and protected branch/environment rules |

## Rollback Policy

Rollback must be treated as a controlled manual action. The pipeline should collect diagnostics automatically when a deployment fails, but it should not blindly rollback without an operator confirming the rollback target.

Recommended rollback sequence:

```text
Deploy failure
  |
Collect diagnostics
  |
Review failed device, commit list, and rendered config
  |
Confirm rollback target
  |
Manual rollback approval
  |
Rollback selected device
  |
Post-check and evidence collection
```

For IOS XR, prefer rolling back to a known commit ID or checkpoint rather than assuming "last 1" is always the correct target. In a multi-router deployment, rollback should be done per device and followed by ISIS, BGP, SR, and VPN validation.

## AUTOCOR Quick Reference

| Concept | Function |
| --- | --- |
| Masked | Hides the value in logs |
| Hidden | Prevents the value from being displayed again |
| Protected | Makes the secret available only to protected branches/tags |
| External secret manager | Centralizes, rotates, and audits secrets |
| `rules` | Controls whether jobs are included and how they behave |
| First match wins | The first matching rule determines job behavior |
| `on_success` | Runs after previous jobs succeed |
| `on_failure` | Runs only after a failure |
| `always` | Runs regardless of success or failure |
| `manual` | Requires a human Play/approval |
| `delayed` | Waits before running |
| `never` | Excludes the job |
| `allow_failure` | Lets a job fail without blocking the pipeline |
| `retry` | Retries transient failures |

## What Each Pipeline Stage Does

| Stage | Purpose |
| --- | --- |
| `lint` | Runs low-risk checks and documentation validation |
| `validate_topology` | Builds offline topology facts from `full-configs` and validates the lab model |
| `dry_run` | Renders the intended config and runs Ansible in `--check --diff` mode |
| `generate_diff` | Publishes rendered config/diff artifacts for review |
| `deploy_lab` | Manually deploys to CML/EVE-NG only from `main` |
| `validate_services` | Uses pyATS/Genie to validate ISIS, BGP, SR, VPNv4, and VPNv6 |
| `post_check` | Collects post-change evidence |
| `diagnostics` | Collects logs and state when deployment fails; rollback remains a separate guarded manual action |
| `deploy_production` | Requires manual approval and protected branch/environment controls |
