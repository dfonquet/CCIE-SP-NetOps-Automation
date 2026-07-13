Purpose:
  Stores the local lab inventory used by Ansible.

What belongs here:
  Device hostnames, management IPs, platform variables, credentials references,
  and group variables for the EVE-NG or CML lab.

How it is used:
  Live validation, dry-run, deployment, diagnostics, and post-check playbooks
  read this inventory.

What you can add next:
  Replace TODO management IPs with real addresses and move credentials to
  environment variables or CI/CD secrets.
