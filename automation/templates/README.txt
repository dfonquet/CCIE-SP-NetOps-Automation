Purpose:
  Stores Jinja2 templates grouped by platform or vendor.

What belongs here:
  Platform-specific template folders such as iosxr, iosxe, junos, or eos.

How it is used:
  The render workflow combines YAML change data with these templates to
  produce candidate configs.

What you can add next:
  Add templates for IOS XE customer routers, multicast services, SR policies,
  or rollback snippets.
