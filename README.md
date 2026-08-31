# Embedded Device Skills

A collection of Codex skills for working safely and repeatably with embedded
devices, serial interfaces, hardware tools, device protocols, and related
development workflows.

Each skill is self-contained under `skills/<skill-name>/` so skills can be
installed independently.

## Available skills

### Agent-Assisted Serial Console

Gives both an agent and a human operator safe, simultaneous access to one
physical serial console through separate virtual TTY endpoints.

The skill requires the user to provide or confirm the exact physical
`/dev/...` device and baud rate before anything opens or takes ownership of
the serial port. It does not guess or auto-detect either value.

[View the skill](skills/agent-assisted-serial-console)

## Install a skill

Ask Codex:

```text
Use $skill-installer to install the agent-assisted-serial-console skill from
https://github.com/KozaMateusz/embedded-device-skills/tree/main/skills/agent-assisted-serial-console
```

Or run the bundled installer:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo KozaMateusz/embedded-device-skills \
  --path skills/agent-assisted-serial-console
```

Start a new Codex task after installation.

## Repository layout

```text
skills/
  agent-assisted-serial-console/
    SKILL.md
    agents/
    references/
```

Additional embedded-device skills can be added as sibling directories under
`skills/`.

## Skill-specific dependencies

Each skill documents its own runtime dependencies and installation procedure.
The agent-assisted serial console currently uses:

- [OpenBaud](https://github.com/Leonezz/openbaud)
- [vsp-router](https://github.com/rfdonnelly/vsp-router)
