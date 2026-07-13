# CCIE SP Change Flow

```text
Engineer creates branch
        |
Modify YAML change data and Jinja2 template
        |
Commit
        |
Pull Request / Merge Request
        |
Pipeline
        |
Validate YAML
        |
Validate rendered IOS XR config
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

## Where to change things

- `change-data/customers/*.yml`: customer, VRF, PE, CE, RD/RT, RR data.
- `templates/iosxr/*.j2`: IOS XR standards.
- `inventories/local/hosts.yml`: CML/EVE-NG or production management IPs.
- `rendered/<CRQ>/`: generated configs, ignored by git.
- `evidence/<CRQ>/`: collected post-checks, ignored by git.

## Local commands

```powershell
python .\automation\scripts\validate_change_data.py
python .\automation\scripts\render_change.py
python .\automation\scripts\validate_rendered_config.py
```

## Ansible commands

```powershell
ansible-playbook -i .\automation\inventories\local\hosts.yml .\automation\playbooks\render-change.yml
ansible-playbook -i .\automation\inventories\local\hosts.yml .\automation\playbooks\dry-run-cml.yml --check --diff
ansible-playbook -i .\automation\inventories\local\hosts.yml .\automation\playbooks\deploy-cml.yml
ansible-playbook -i .\automation\inventories\local\hosts.yml .\automation\playbooks\post-validate.yml
```
