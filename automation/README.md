# CCIE SP Lab Automation

Automatizacion local para aprovechar los `full-configs` del lab CCIE SP.

Este modulo no necesita acceso SSH para empezar. Primero construye una vista offline de la red desde los archivos `.txt`:

- hostnames y roles
- Loopback600 IPv4/IPv6
- interfaces activas y apagadas
- vecinos detectados por `description ... -> PEER INTERFACE`
- protocolos presentes: ISIS, BGP, Segment Routing, PCE/PCC, PIM, VRF
- inventario base para Ansible
- diagrama Mermaid y CSV de enlaces

## Flujo recomendado

```text
Configs en full-configs
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
CI / validacion local
   |
Ansible live checks / deploy controlado
```

## Flujo de cambios con templates

Para desplegar una VRF nueva o estandarizar un XR nuevo:

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

Archivos principales:

- `change-data/customers/cust-new-vrf.yml`: datos del cliente, PE, RD/RT, CE-BGP y RRs.
- `templates/iosxr/iosxr_pe_vrf.j2`: VRF, interfaz cliente y BGP CE.
- `templates/iosxr/iosxr_base_isis.j2`: estandar ISIS/SR para XR. Cambia el NET y prefix-sid desde `pe_id` o `node_id`.
- `templates/iosxr/iosxr_rr_add_clients.j2`: agrega PEs como clientes iBGP en los RRs.
- `docs/change-flow.md`: flujo operativo completo.

## Uso local

Desde la raiz de este repo:

```powershell
python -m pip install -r .\automation\requirements.txt
python .\automation\scripts\build_lab_facts.py
python .\automation\scripts\validate_lab_facts.py
python .\automation\scripts\validate_change_data.py
python .\automation\scripts\render_change.py
python .\automation\scripts\validate_rendered_config.py
```

Los resultados quedan en:

```text
automation/generated/
```

## Uso con Ansible

El inventario generado usa `ansible_host` como TODO cuando no hay management IP clara. Edita:

```text
automation/inventories/local/hosts.yml
```

Despues puedes correr:

```powershell
ansible-galaxy collection install -r .\automation\requirements.yml
ansible-playbook -i .\automation\inventories\local\hosts.yml .\automation\playbooks\validate-live.yml
```

## Nota

Para cambios reales usa playbooks separados y siempre primero `--check --diff`. La primera etapa de este modulo es observabilidad y validacion offline, no deploy automatico agresivo.
