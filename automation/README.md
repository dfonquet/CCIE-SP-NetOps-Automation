# CCIE SP Lab Automation

Local automation for extracting value from the CCIE SP lab `full-configs`.

This module does not require SSH access to get started. It first builds an offline network view from the `.txt` config files:

- hostnames and roles
- Loopback600 IPv4/IPv6
- active and shutdown interfaces
- neighbors detected through `description ... -> PEER INTERFACE`
- detected protocols: ISIS, BGP, Segment Routing, PCE/PCC, PIM, VRF
- baseline Ansible inventory
- Mermaid diagram and topology edge CSV

## Recommended Flow

```text
Configs in full-configs
   |
scripts/build_lab_facts.py
   |
generated/lab_facts.yml
generated/ansible_inventory.yml
generated/topology_edges.csv
generated/topology.mmd
   |
scripts/validate_lab_facts.py
   |
CI / local validation
   |
Ansible live checks / controlled deployment
```

## Execution Model

This automation stack is intended to run from a Linux automation host or VM, not from the routers themselves.

The offline phase uses only the configuration files stored under `full-configs/`. It does not require SSH, NETCONF, or IP reachability to the lab devices. Use this phase first to build topology facts, validate the lab model, and understand what the repository sees.

The live phase starts only after the Ansible inventory is completed. Live validation, dry-runs, deployment, diagnostics, and pyATS checks require management IP reachability, working credentials, and an updated inventory under `automation/inventories/local/hosts.yml`.

## Before You Start

- Use a Linux automation host or VM, preferably Ubuntu 22.04 LTS or 24.04 LTS.
- Install Python dependencies from `automation/requirements.txt`.
- Install Ansible collections from `automation/requirements.yml`.
- Install pyATS/Genie from `automation/requirements-pyats.txt` only where live validation will run.
- Update `automation/inventories/local/hosts.yml` with real management IPs and connection variables.
- Run the offline checks before any live command.
- Use `--check --diff` before any deployment.

## Quick Start

```powershell
python -m pip install -r .\automation\requirements.txt
ansible-galaxy collection install -r .\automation\requirements.yml
python .\automation\scripts\build_lab_facts.py
python .\automation\scripts\validate_lab_facts.py
python .\automation\scripts\validate_change_data.py
python .\automation\scripts\render_change.py
python .\automation\scripts\validate_rendered_config.py
```

## Lab Assumptions

- `full-configs/` contains IOS XR and CSR config snapshots from the EVE-NG lab.
- The first workflow is offline and safe to run without device access.
- Live workflows assume SSH reachability to IOS XR/CSR management interfaces.
- The automation host is connected to the EVE-NG or CML management network.
- Production-style deployment should be manual, reviewed, and protected by branch/environment rules.

## Template-Based Change Flow

For deploying a new VRF or standardizing a new XR node:

```text
Engineer creates branch
        |
Modify YAML change data and Jinja2 template
        |
Commit / Pull Request
        |
Validate YAML
        |
Render IOS XR config
        |
Validate rendered config
        |
Ansible dry-run --check --diff
        |
Deploy to CML/EVE-NG
        |
pyATS validates BGP, ISIS, VPNv4, VPNv6
        |
Manual approval
        |
Deploy to production
        |
Post-validation
        |
Evidence attached to CRQ
```

Key files:

- `change-data/customers/cust-new-vrf.yml`: customer, PE, RD/RT, CE-BGP, and RR data.
- `templates/iosxr/iosxr_pe_vrf.j2`: VRF, customer interface, and CE BGP.
- `templates/iosxr/iosxr_base_isis.j2`: IOS XR ISIS/SR standard. The NET and prefix-SID are derived from `pe_id` or `node_id`.
- `templates/iosxr/iosxr_rr_add_clients.j2`: adds PEs as iBGP clients on the route reflectors.
- `docs/change-flow.md`: full operational flow.
- `docs/repository-guide.md`: file-by-file explanation of what each part does and what can be extended next.

## Local Usage

From the repository root:

```powershell
python -m pip install -r .\automation\requirements.txt
python .\automation\scripts\build_lab_facts.py
python .\automation\scripts\validate_lab_facts.py
python .\automation\scripts\validate_change_data.py
python .\automation\scripts\render_change.py
python .\automation\scripts\validate_rendered_config.py
```

Outputs are written to:

```text
automation/generated/
```

## Ansible Usage

The generated inventory uses `ansible_host` as `TODO` when there is no clear management IP. Edit:

```text
automation/inventories/local/hosts.yml
```

Then run:

```powershell
ansible-galaxy collection install -r .\automation\requirements.yml
ansible-playbook -i .\automation\inventories\local\hosts.yml .\automation\playbooks\validate-live.yml
```

## Note

For real changes, use separate playbooks and always start with `--check --diff`. The first stage of this module is observability and offline validation, not aggressive automatic deployment.
