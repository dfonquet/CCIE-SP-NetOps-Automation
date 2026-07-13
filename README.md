# CCIE SP NetOps Automation

Standalone automation repository for a CCIE Service Provider lab running Cisco IOS XR.

This repository keeps NetOps automation separate from the website. The website can keep serving lab configs as a documentation library, while CI/CD, validation, templates, and deployment workflows live here.

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
python -m pip install -r .\automation\requirements.txt
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
