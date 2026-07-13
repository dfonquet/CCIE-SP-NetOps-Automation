Purpose:
  Stores Ansible inventories for the lab and future environments.

What belongs here:
  Inventory folders such as local, cml, eve-ng, staging, or production.

How it is used:
  Ansible playbooks use these inventories to know which devices to connect to
  and which connection variables to use.

What you can add next:
  Split local lab and production inventories, add group_vars, or generate
  inventories from parsed lab facts.
