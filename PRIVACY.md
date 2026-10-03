# Privacy Policy — Memory Digest

`Memory Digest` is the **reader** in a Notes + Digest pair. The
interests it shows are read straight from your device's assistant via
`octos.session.history`; the same screen's **Refresh** button asks the
assistant to summarise them in its own words. **Add** sends
`I'm interested in <text>` to the assistant, so the new chip appears
the next time Notes or Digest reads back. The device itself keeps no
copy of the interests.

## Data this app stores

None. No file is written under `.local-state/memory-digest/` for the
interests themselves; the app does not call the `storage` capability.

## Data this app requests

- `octos.session.history` host service — invoked when the screen opens
  and after every Add / Forget. The response is filtered locally for
  messages whose role is `user` and whose text starts with
  `I'm interested in ` or `I like `. The remaining text becomes one
  chip. Nothing else from the conversation is read, rendered, stored,
  or forwarded.
- `octos.turn.start` host service — invoked on three explicit user
  actions, each sending exactly one short string to the assistant:
  - **Add**: `I'm interested in <text>`
  - **Refresh**:
    `In one short paragraph, summarize what you remember about my interests and preferences.`
  - **Forget** (per chip): `Forget this interest: <text>`

## What this app does NOT do

- It does not request `network.hosts` and does not open any HTTP/HTTPS
  connection.
- It does not request the `storage` capability and does not write any
  local file.
- It does not collect telemetry, analytics, or device identifiers.
- It does not auto-refresh on a timer; the prompt is gated by the
  Refresh button (and the startup one-shot, with the same one-line
  prompt).
- It does not collect passwords, PINs, codes, or any account field.

## Agent profile

`read-only` — the app never asks the host to write, sign, or publish.

## On devices without an assistant

If the host has no assistant service (today: every OctoSense shell,
every `card-host`), the digest card and the chip list show
**Assistant unavailable** and the buttons do nothing. There is no
local cache to fall back to.

## Contact

https://github.com/Thneoly/memory-digest/issues

## Source

https://github.com/Thneoly/memory-digest
