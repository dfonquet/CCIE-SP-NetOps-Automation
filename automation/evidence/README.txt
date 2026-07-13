Purpose:
  Stores runtime evidence collected after validation, deployment, or failure.

What belongs here:
  Post-check outputs, diagnostics, pyATS logs, command outputs, and CRQ
  evidence artifacts.

How it is used:
  Playbooks write evidence here so it can be reviewed or attached to change
  records.

What you can add next:
  Add structured JSON summaries, per-change folders, or archive/export jobs.

Note:
  Runtime evidence is usually ignored by git. Keep only this README and the
  placeholder .gitignore under version control.
