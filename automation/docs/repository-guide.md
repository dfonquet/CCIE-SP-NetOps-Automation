# Repository Guide

This guide explains what each important file or folder does, why it exists, and what you can extend next.

## Root Files

| Path | What it does | Use it for | What you could add next |
| --- | --- | --- | --- |
| `README.md` | Main project landing page with lab context, screenshots, requirements, and workflow overview | First entry point for users visiting the repo | Add links to demo videos, lab build notes, or release versions |
| `.github/workflows/ccie-sp-lab-ci.yml` | GitHub Actions workflow for offline topology validation | Validate `full-configs` and generated lab facts | Add artifact publishing for topology diagrams |
| `.github/workflows/ccie-sp-change-pipeline.yml` | GitHub Actions workflow for change rendering, dry-run, CML/EVE-NG deploy, pyATS, and production gates | Run the NetOps change lifecycle from GitHub | Add environment-specific inventories and notification hooks |
| `.gitignore` | Keeps generated files, secrets, caches, and evidence out of git | Prevent accidental commits of local/runtime files | Add tool-specific cache paths if the automation node creates more |

## Lab Source Data

| Path | What it does | Use it for | What you could add next |
| --- | --- | --- | --- |
| `full-configs/` | Stores baseline router config snapshots from the CCIE SP lab | Offline parsing, topology discovery, facts generation, and documentation | Add new snapshots when the EVE-NG lab changes |
| `automation/generated/` | Stores generated facts, inventory, Mermaid topology, and CSV links | Local outputs from `build_lab_facts.py` | Publish as pipeline artifacts instead of committing generated files |
| `automation/rendered/` | Stores rendered IOS XR candidate configs by change ID | Review generated configs before deployment | Add per-device diffs or approvals per router |
| `automation/evidence/` | Stores validation and diagnostics evidence | Attach post-check output to a CRQ or change record | Add structured JSON reports for dashboards |

## Change Data

| Path | What it does | Use it for | What you could add next |
| --- | --- | --- | --- |
| `automation/change-data/customers/cust-new-vrf.yml` | Example customer service definition: VRF, PE devices, RD/RT, CE BGP, RR data | Source of truth for rendering a new L3VPN customer change | Add one YAML file per customer, service, or CRQ |

## Templates

| Path | What it does | Use it for | What you could add next |
| --- | --- | --- | --- |
| `automation/templates/iosxr/iosxr_pe_vrf.j2` | Renders IOS XR VRF, customer interface, route-policy, and CE BGP config | Build PE-side L3VPN service config from YAML | Add route-policy variants, prefix filters, QoS, or multicast options |
| `automation/templates/iosxr/iosxr_base_isis.j2` | Renders IOS XR IS-IS/SR base config using `pe_id` or `node_id` for NET and prefix-SID | Standardize new XR nodes or refresh base ISIS/SR config | Add TI-LFA, SRGB, Flex-Algo, or per-link metrics |
| `automation/templates/iosxr/iosxr_rr_add_clients.j2` | Renders route-reflector client config for PE loopbacks | Add PEs as VPNv4/VPNv6 RR clients | Add separate primary/shadow RR policies or peer-groups |

## Scripts

| Path | What it does | Use it for | What you could add next |
| --- | --- | --- | --- |
| `automation/scripts/build_lab_facts.py` | Parses `full-configs` and creates lab facts, topology edges, inventory, and Mermaid output | Offline topology visibility without logging into routers | Add JSON output, graph validation, or duplicate-link detection |
| `automation/scripts/validate_lab_facts.py` | Validates parsed lab facts and flags warnings such as missing peers or duplicate addresses | Catch topology/config inconsistencies before live automation | Add strict mode per lab domain or expected device count |
| `automation/scripts/validate_change_data.py` | Validates the customer/change YAML before rendering | Stop invalid service data early | Add schema validation with JSON Schema or Pydantic |
| `automation/scripts/render_change.py` | Renders IOS XR configs from YAML and Jinja2 templates | Generate candidate configs for review, dry-run, and deploy | Add per-device diff generation against running config |
| `automation/scripts/validate_rendered_config.py` | Checks rendered configs for unresolved variables and basic expected blocks | Catch template/rendering mistakes before Ansible runs | Add IOS XR parser checks or golden policy validation |

## Ansible

| Path | What it does | Use it for | What you could add next |
| --- | --- | --- | --- |
| `automation/ansible.cfg` | Sets local Ansible defaults for this repo | Consistent Ansible behavior across commands | Add callback/logging settings for CI evidence |
| `automation/inventories/local/hosts.yml` | Local inventory placeholder for IOS XR management IPs and connection variables | Tell Ansible how to reach the lab devices | Split into `cml`, `eve-ng`, and `production` inventories |
| `automation/playbooks/render-change.yml` | Runs validation, rendering, and rendered-config checks from Ansible | One-command render workflow | Add CRQ variable input and artifact directory selection |
| `automation/playbooks/dry-run-cml.yml` | Runs `iosxr_config` in check mode with diff | Preview changes against the lab before deploy | Add per-router serial execution and diff archiving |
| `automation/playbooks/deploy-cml.yml` | Deploys rendered configs to CML/EVE-NG devices | Controlled lab deployment | Add pre/post task hooks per service type |
| `automation/playbooks/deploy-production.yml` | Deploys rendered configs after explicit approval variable | Guarded production deployment pattern | Add environment protection and device batches |
| `automation/playbooks/post-validate.yml` | Collects BGP, ISIS, and route evidence after changes | Attach post-check output to a CRQ | Add structured pass/fail assertions |
| `automation/playbooks/collect-diagnostics.yml` | Collects non-destructive diagnostics after failed deploys | Failure evidence and troubleshooting | Add guarded rollback playbooks after commit/checkpoint confirmation |
| `automation/requirements.yml` | Lists Ansible collections required by this project | Install `ansible.netcommon` and `cisco.iosxr` | Pin tested collection versions |

## pyATS

| Path | What it does | Use it for | What you could add next |
| --- | --- | --- | --- |
| `automation/pyats/ccie_sp_testbed.yml` | Example pyATS testbed with TODO management IPs | Live BGP/ISIS/VPN validation | Generate it from inventory automatically |
| `automation/pyats/validate_sp_services.py` | pyATS validation skeleton for BGP VPNv4/VPNv6 and ISIS | Service validation after deploy | Add Genie parsers, SR policy checks, and per-VRF assertions |
| `automation/requirements-pyats.txt` | Python dependencies for pyATS/Genie live validation | Install validation tooling only where needed | Pin versions tested in your automation VM |

## CI/CD

| Path | What it does | Use it for | What you could add next |
| --- | --- | --- | --- |
| `automation/ci/gitlab-ci.example.yml` | GitLab CI example with lint, topology validation, dry-run, manual deploy, pyATS, diagnostics, and production gates | Reference for GitLab-based NetOps workflow | Add real runner tags, protected environments, and notification jobs |
| `automation/docs/pipeline-operations.md` | Explains secrets, rules, retry, allow_failure, diagnostics, rollback policy, and pipeline stages | Understand the operational model behind the CI/CD design | Add organization-specific CRQ and approval rules |
| `automation/docs/change-flow.md` | High-level change flow from branch to evidence | Quick workflow explanation for engineers | Add screenshots or example CRQ output |

## Dependency Files

| Path | What it does | Use it for | What you could add next |
| --- | --- | --- | --- |
| `automation/requirements.txt` | Core Python dependencies for rendering, validation, and Ansible | Automation node setup | Pin exact versions after testing |
| `automation/requirements-pyats.txt` | Heavy live-validation dependencies | Install only on runners that execute pyATS | Keep separate to avoid slowing basic lint jobs |

## Suggested Extension Order

1. Fill `automation/inventories/local/hosts.yml` with real management IPs.
2. Create one change YAML per customer or CRQ.
3. Render configs locally and review `automation/rendered/`.
4. Run Ansible dry-run against EVE-NG/CML.
5. Expand pyATS assertions for your exact ISIS, BGP, SR, VPNv4, and VPNv6 expectations.
6. Add guarded rollback playbooks only after defining your IOS XR commit/checkpoint policy.
