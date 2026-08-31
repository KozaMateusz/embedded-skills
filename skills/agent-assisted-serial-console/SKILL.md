---
name: agent-assisted-serial-console
description: Give both an agent and a human operator simultaneous, safe access to one physical serial console through separate virtual TTYs. Use for collaborative serial-console setup, shared receive streams, routing-component installation, and agent-plus-terminal access.
---

# Agent-assisted serial console

Set up a serial topology in which `vsp-router` is the sole owner of the physical
TTY. Give OpenBaud and the human operator separate virtual TTYs:

```text
physical TTY -> vsp-router -> OpenBaud PTY
                         `-> user PTY
```

Read [references/install.md](references/install.md) when either dependency must
be installed or updated. Read
[references/shared-tty.md](references/shared-tty.md) when creating, operating,
persisting, or troubleshooting the shared topology.

## Invariants

- Before opening a serial session, starting a capture, changing TTY ownership,
  or launching `vsp-router`, establish both the exact physical `/dev/...` path
  and the baud rate. If the intended device is not unambiguous, ask the user
  which exact `/dev` device to use. Always ask the user for the baud rate unless
  they already supplied or explicitly confirmed it. Never guess, infer,
  auto-detect, or silently take either value from a likely-looking port or
  profile. Without both values, do not start anything that opens or owns the
  device; read-only discovery with `list_ports` is allowed only to help the user
  identify the path.
- Discover and identify the physical device with OpenBaud `list_ports` before
  opening it or changing ownership.
- Never let OpenBaud, a terminal emulator, and `vsp-router` independently open
  the same physical TTY. Competing readers split bytes instead of receiving a
  broadcast.
- Before inserting the router, stop any active capture and close the OpenBaud
  physical-port session. Explain that this causes one brief disconnect.
- After routing, use OpenBaud only through its dedicated PTY. Do not replace its
  audited device operations with shell writes.
- Explain the expected effect before sending bytes. Do not use an unknown frame
  as a connectivity test; prefer passive reads or a harmless documented command.
- Two virtual clients can transmit concurrently, so coordinate writers. The
  router prevents competing reads but does not make interleaved commands safe.
- Request approval immediately before downloads, installation outside the
  workspace, service changes, or access to a host-hidden `/dev` node.

## Completion criteria

Confirm all of the following:

1. `vsp-router --version` succeeds.
2. The router owns the intended physical device at the intended baud rate.
3. Both virtual TTY paths exist and resolve to distinct PTYs.
4. OpenBaud opens only its PTY and starts a capture for continued exploration.
5. The user can open the other PTY with `tio`, `picocom`, or an equivalent
   terminal.
6. A passive device message or harmless documented command reaches both sides.

Report the physical path, both virtual paths, baud rate, OpenBaud session ID,
router lifecycle, and the exact user terminal command.
