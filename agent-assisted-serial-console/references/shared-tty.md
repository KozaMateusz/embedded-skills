# Shared TTY setup

## 1. Resolve the physical device

Use OpenBaud `list_ports`. Match USB VID, PID, serial number, manufacturer, and
product rather than assuming `/dev/ttyUSB0` is stable. Prefer a matching
`/dev/serial/by-id/...` link when one exists. If the intended port is not
unambiguous, ask the user for the exact `/dev/...` device. Ask the user for the
baud rate unless they already supplied or explicitly confirmed it. Never guess,
infer, or auto-detect either value, and do not use a profile or documentation as
silent confirmation. Do not open the device, start a capture, hand off
ownership, or launch `vsp-router` until both the exact physical path and the
user-supplied or user-confirmed baud rate are known. `list_ports` remains
permitted as read-only discovery to help the user choose.

Establish data bits, parity, and stop bits from user input, an existing device
profile, or device documentation.

Inspect `devices/` for a profile before probing. If the protocol is unknown,
do not transmit.

## 2. Perform the one-time handoff

If OpenBaud currently owns the physical port:

1. Stop its active capture and retain the returned capture path.
2. Close its session.
3. Ensure no terminal or other process owns the physical port.

Explain before doing this that inserting a broker requires a brief disconnect.

## 3. Start vsp-router

Unless the user explicitly requests other endpoint paths, always use
`/tmp/openbaud-agent` for OpenBaud and `/tmp/openbaud-user` for the human
operator. Keep these names unchanged across router restarts and physical-device
re-enumeration so clients have stable connection paths.

For an initial foreground setup, use explicit virtual paths and the confirmed
physical path and baud. Example:

```bash
vsp-router \
  --create agent:/tmp/openbaud-agent \
  --create user:/tmp/openbaud-user \
  --attach physical:/dev/ttyUSB0,921600 \
  --route agent:physical \
  --route user:physical \
  --route physical:agent \
  --route physical:user
```

The routing means both virtual clients can transmit to the physical device and
physical RX is copied to both clients. It intentionally does not copy one
client's TX directly to the other client. On an echoing shell console, the
device's echo makes typed commands visible to both; on a non-echoing protocol,
only responses are shared.

Keep the process alive. Record how it is supervised (foreground exec session,
terminal multiplexer, or user service). If a sandbox cannot see `/dev/ttyUSB0`
but OpenBaud `list_ports` can, request host-device access for the router rather
than claiming that the device disappeared.

Verify the endpoints:

```bash
ls -l /tmp/openbaud-agent /tmp/openbaud-user
readlink -f /tmp/openbaud-agent
readlink -f /tmp/openbaud-user
```

They must resolve to distinct `/dev/pts/*` devices.

## 4. Connect both clients

Open the agent endpoint with OpenBaud, using the confirmed transport settings:

```text
open(port=/tmp/openbaud-agent, baud=921600, data_bits=8,
     parity=none, stop_bits=1)
```

Start an OpenBaud capture on the new session before extended exploration.

After creating or recreating the endpoints, always print the exact installed
terminal command the user can run, including the confirmed baud rate. For
example:

```bash
tio -b 921600 /tmp/openbaud-user
```

Alternatives include `picocom --baud 921600 /tmp/openbaud-user` and
`screen /tmp/openbaud-user 921600`. Baud on a PTY is not electrically
significant, but passing the physical baud keeps the configuration legible.

## 5. Verify routing

Prefer passive device output. If the device is a confirmed shell console, a
harmless command such as `pwd` or `ls` is a reasonable bounded test after
explaining its effect. Confirm from router logs or both clients that physical
RX reached both PTYs. Do not infer full duplex from path creation alone.

## Optional user-service persistence

When requested, create a user systemd service with resolved device and baud
values. Use `%t/openbaud-vsp/agent` and `%t/openbaud-vsp/user` for runtime paths
and `RuntimeDirectory=openbaud-vsp` so stale links do not live indefinitely.
An example unit shape is:

```ini
[Unit]
Description=Shared serial router for OpenBaud

[Service]
Type=simple
RuntimeDirectory=openbaud-vsp
RuntimeDirectoryMode=0700
ExecStart=%h/.local/bin/vsp-router --create agent:%t/openbaud-vsp/agent --create user:%t/openbaud-vsp/user --attach physical:/dev/serial/by-id/DEVICE,921600 --route agent:physical --route user:physical --route physical:agent --route physical:user
Restart=on-failure
RestartSec=2

[Install]
WantedBy=default.target
```

Replace `DEVICE` and `921600` with verified values before writing the unit.
Request approval before creating or enabling it. After startup, OpenBaud uses
`/run/user/<uid>/openbaud-vsp/agent` and the user uses
`/run/user/<uid>/openbaud-vsp/user`.

## Troubleshooting and shutdown

- `No such file or directory` for a physical port while `list_ports` sees it:
  the launching sandbox may hide host device nodes.
- `Permission denied`: inspect ownership and group membership; do not broaden
  device permissions indiscriminately.
- Missing early output on the user side: virtual ports buffer unread data;
  attach and flush stale input if appropriate.
- Garbled physical data: stop transmitting and re-check baud and 8N1 settings.
- Mixed commands: coordinate writers; vsp-router does not provide transaction
  locking between clients.
- Router exit: OpenBaud's PTY session is no longer valid. Close it, restart the
  router, and open a new OpenBaud session.

For orderly shutdown, close the OpenBaud PTY session, exit the user terminal,
then stop the router. Preserve any capture before closing it.
