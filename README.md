# CCIE SP NetOps Automation

[![Python](https://img.shields.io/badge/python-3.12+-3776AB?logo=python&logoColor=white)](#requirements)
[![Ansible](https://img.shields.io/badge/ansible-netops-EE0000?logo=ansible&logoColor=white)](#requirements)
[![pyATS](https://img.shields.io/badge/pyATS-validation-1f6feb)](#change-flow)
[![Lab](https://img.shields.io/badge/lab-IOS%20XR%206.3.1-0B7285)](#eve-ng-node-profiles)
[![Topology](https://img.shields.io/badge/topology-dual--stack%20SP-6f42c1)](#lab-at-a-glance)

Standalone automation repository for a CCIE Service Provider lab running Cisco IOS XR.

This repository keeps NetOps automation separate from the website. The website can keep serving lab configs as a documentation library, while CI/CD, validation, templates, and deployment workflows live here.

## Lab at a Glance

This lab models a dual-stack service provider backbone with IS-IS L2, Segment Routing, SR-TE with PCE/PCC, VPNv4/VPNv6 services, multicast, and inter-AS connectivity.

| Domain | Scope |
| --- | --- |
| ISP-1 | 27 devices in the EVE-NG design |
| ISP-2 | 19 devices in the EVE-NG design |
| Core protocols | IS-IS Level 2, BGP AS 500, MPLS Segment Routing |
| Services | L3VPN VRFs, VPNv4/VPNv6, multicast ASM/PIM-SM, Inter-AS |
| Automation scope | Offline config parsing, topology facts, Jinja2 rendering, Ansible deployment, pyATS validation |

## Lab Visuals

![CCIE SP full lab topology](docs/images/ccie-sp-full-topology.png)

| ISP and customer edges | Inter-AS and external connectivity |
| --- | --- |
| <img src="docs/images/isp2-pe-receivers.png" alt="ISP-2 PE receiver edge" width="430"> | <img src="docs/images/inter-as-edge.png" alt="Inter-AS edge" width="430"> |
| <img src="docs/images/isp1-pe-nat-edge.png" alt="ISP-1 PE NAT and edge services" width="430"> | <img src="docs/images/internet-edge-bgp65550.png" alt="Internet edge BGP 65550" width="430"> |

Current `full-configs` snapshots can be parsed offline to generate device facts, topology edges, an Ansible inventory, and a Mermaid topology view.

```mermaid
flowchart LR
  CUST_L["Customer / CE Nodes"] --> PE_L["PE Edge"]
  PE_L --> ISP1["ISP-1 Backbone\nIS-IS L2 / SR-MPLS / BGP 500"]
  ISP1 --> RR["Route Reflectors\nP11 / P12"]
  ISP1 --> PCE["PCE / SR-TE"]
  ISP1 --> INTERAS["Inter-AS Edge"]
  INTERAS --> ISP2["ISP-2 Backbone"]
  ISP2 --> PE_R["PE Edge"]
  PE_R --> CUST_R["Customer / Multicast Receivers"]
```

## Requirements

| Requirement | Purpose |
| --- | --- |
| Python 3.12+ | Local scripts, config parsing, validation, and rendering |
| Jinja2 + PyYAML | Template rendering and YAML change data validation |
| Ansible Core | Dry-run, deployment, and post-validation workflows |
| `cisco.iosxr` collection | IOS XR device automation |
| `ansible.netcommon` collection | Network connection plugins |
| pyATS + Genie | BGP, IS-IS, VPNv4/VPNv6 validation |
| EVE-NG or CML | Local lab execution environment |

## How to Use This in Your Lab

This automation is intended to run from an external automation node, not inside the IOS XR routers themselves. The recommended setup is a dedicated Linux workstation or Linux VM, preferably Ubuntu or a similar distribution, connected to the same management network as the EVE-NG or CML lab.

The automation node should have IP reachability and SSH access to the lab devices. It is responsible for offline config parsing, Jinja2 rendering, Ansible dry-runs, controlled deployment, pyATS validation, and evidence collection.

Recommended automation node options:

| Option | Recommendation |
| --- | --- |
| Ubuntu Server/Desktop 22.04 LTS or 24.04 LTS | Best default choice for Ansible, pyATS, Git, and Python tooling |
| Debian or another modern Linux distro | Fine if Python 3.12+, pip, SSH, and Ansible dependencies are available |
| Windows host with a Linux VM | Recommended if your main machine is Windows. Run the automation inside the Linux VM and use Windows/GitHub Desktop for editing if desired |
| Native Windows only | Usable for editing and Git operations, but not recommended as the main Ansible/pyATS execution node |

Network placement:

- Place the automation node on the EVE-NG or CML management network.
- It does not need to be in the service-provider data plane.
- It must reach the IOS XR and CSR management IPs over SSH.
- It should also reach the EVE-NG or CML web/API address if you plan to automate lab startup, snapshots, or device lifecycle later.
- Internet access is recommended for the initial setup: cloning this repository, installing Python packages, and downloading Ansible collections. After dependencies are cached, most validation and deployment workflows can run without Internet access.

Suggested Linux folder layout:

```bash
mkdir -p ~/netops
cd ~/netops
git clone <your-repo-url> CCIE-SP-NetOps-Automation
cd CCIE-SP-NetOps-Automation
```

Suggested management path:

```text
Linux automation VM
        |
EVE-NG/CML management bridge or cloud network
        |
IOS XR / CSR management interfaces
```

Recommended lab workflow:

1. Clone this repository on the Linux automation node.
2. Install Python dependencies and Ansible collections.
3. Update `automation/inventories/local/hosts.yml` with management IPs, usernames, and connection variables.
4. Run the offline flow first: parse `full-configs`, build topology facts, and validate lab data.
5. Render the intended change from YAML and Jinja2 templates.
6. Run Ansible in `--check --diff` mode before any live deployment.
7. Deploy first to EVE-NG or CML, validate with pyATS/Genie, then move to production only after manual approval.

Install the Python dependencies with:

```powershell
python -m pip install -r .\automation\requirements.txt
```

Install pyATS only on the runner or workstation that performs live validation:

```powershell
python -m pip install -r .\automation\requirements-pyats.txt
```

Install Ansible collections:

```powershell
ansible-galaxy collection install -r .\automation\requirements.yml
```

## EVE-NG Node Profiles

| Role | Image | RAM | Ethernet Interfaces | QEMU NIC |
| --- | --- | ---: | ---: | --- |
| SP routers / P / PE / RR / PCE | `xrv-K9-demo-6.3.1` | 3072 MB | 10 | `virtio-net-pci` |
| Customer routers / CE | `csr1000vng-universalk9.17.03.08.a-serial` | 4096 MB | 8 | `virtio-net-pci` |

## Structure

```text
.github/workflows/       GitHub Actions pipelines
automation/              Scripts, templates, Ansible, pyATS, and evidence
full-configs/            Baseline lab configs
```

## Included Workflows

- Offline lab visibility from `full-configs`.
- Generated facts, Ansible inventory, topology edge CSV, and Mermaid diagram.
- Standard workflow for adding a new VRF across multiple PEs.
- Jinja2 templates for IOS XR: VRF, ISIS/SR, and iBGP with route reflectors.
- Ansible playbooks for rendering, dry-run, CML/EVE-NG, production, and post-validation.
- pyATS skeleton for validating BGP, ISIS, VPNv4, and VPNv6.
- GitHub Actions pipeline and GitLab CI example.

## Local Usage

```powershell
python .\automation\scripts\build_lab_facts.py
python .\automation\scripts\validate_lab_facts.py
python .\automation\scripts\validate_change_data.py
python .\automation\scripts\render_change.py
python .\automation\scripts\validate_rendered_config.py
```

## Change Flow

```text
Engineer creates branch
        |
Modify YAML change data and Jinja2 template
        |
Commit / Pull Request
        |
Validate YAML
        |
Validate rendered IOS XR config
        |
Dry-run --check --diff
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

## Note

Generated files under `automation/generated`, `automation/rendered`, and `automation/evidence` are ignored by git except for their `.gitignore` placeholders.
