# CCIE SP NetOps Automation

Repositorio standalone para automatizacion de un lab CCIE Service Provider con Cisco IOS XR.

Este repo separa la automatizacion de la pagina web. La pagina puede seguir mostrando configs como libreria, pero el flujo CI/CD vive aqui.

## Estructura

```text
.github/workflows/       Pipelines GitHub Actions
automation/              Scripts, templates, Ansible, pyATS y evidencia
full-configs/            Configs base del lab
```

## Flujos incluidos

- Vision offline del lab desde `full-configs`.
- Generacion de facts, inventario Ansible, CSV de enlaces y diagrama Mermaid.
- Estandar para nueva VRF en varios PE.
- Templates Jinja2 para IOS XR: VRF, ISIS/SR e iBGP con RR.
- Playbooks Ansible para render, dry-run, CML/EVE-NG, produccion y post-validacion.
- pyATS skeleton para validar BGP, ISIS, VPNv4 y VPNv6.
- Pipeline GitHub Actions y ejemplo GitLab CI.

## Uso local

```powershell
python -m pip install -r .\automation\requirements.txt
python .\automation\scripts\build_lab_facts.py
python .\automation\scripts\validate_lab_facts.py
python .\automation\scripts\validate_change_data.py
python .\automation\scripts\render_change.py
python .\automation\scripts\validate_rendered_config.py
```

## Flujo de cambio

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

## Nota

Los archivos generados en `automation/generated`, `automation/rendered` y `automation/evidence` quedan ignorados por git salvo sus `.gitignore`.
