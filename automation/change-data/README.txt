Purpose:
  Stores structured change input data used by the automation templates.

What belongs here:
  YAML files that describe services, customers, devices, VRFs, route-targets,
  BGP neighbors, and other intent data.

How it is used:
  Scripts read these YAML files and render IOS XR configs through Jinja2
  templates.

What you can add next:
  Add one folder per service type, such as customers, core, multicast, or
  segment-routing.
