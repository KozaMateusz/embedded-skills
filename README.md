# Agent-Assisted Serial Console

A Codex skill that gives both an agent and a human operator safe, simultaneous
access to one physical serial console through separate virtual TTY endpoints.

The skill requires the user to provide or confirm the exact physical
`/dev/...` device and baud rate before anything opens or takes ownership of
the serial port. It does not guess or auto-detect either value.

## Install

Ask Codex:

```text
Use $skill-installer to install the agent-assisted-serial-console skill from
https://github.com/KozaMateusz/agent-assisted-serial-console/tree/main/skills/agent-assisted-serial-console
```

Or run the bundled installer:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo KozaMateusz/agent-assisted-serial-console \
  --path skills/agent-assisted-serial-console
```

Start a new Codex task after installation.

## Runtime dependencies

The skill checks for and, with approval, installs:

- [OpenBaud](https://github.com/Leonezz/openbaud)
- [vsp-router](https://github.com/rfdonnelly/vsp-router)

See the skill's installation reference for the exact audited procedure.
