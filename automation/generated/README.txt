Purpose:
  Stores files generated from offline parsing of the lab configs.

What belongs here:
  Generated lab facts, Ansible inventory, topology edges, and Mermaid
  diagrams.

How it is used:
  The offline scripts write here after reading `full-configs/`.

What you can add next:
  Add JSON exports, graph files, or CI artifacts for topology review.

Note:
  Generated outputs are usually ignored by git. Recreate them by running
  `automation/scripts/build_lab_facts.py`.
