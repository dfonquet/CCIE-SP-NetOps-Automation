Purpose:
  Stores Cisco IOS XR Jinja2 templates.

What belongs here:
  Templates for PE VRFs, IS-IS/SR base config, route-reflector clients, SR-TE,
  multicast, and other IOS XR features.

How it is used:
  `render_change.py` reads YAML change data and renders these templates into
  per-device candidate configs.

What you can add next:
  Add TI-LFA, Flex-Algo, SR policy, PCE/PCC, multicast, QoS, or route-policy
  templates.
