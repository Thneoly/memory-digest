# Privacy Policy — Memory Digest

`Memory Digest` is a **store app** — a digest reader that, on Refresh,
asks this app's own assistant to summarize the user's locally-stored
interests. The reader works without an assistant by quoting the local list.

## Data this app stores

- `prefs.json` in `.local-state/memory-digest/` — the interest chips the
  user typed on this device. Storage cap: 16 MiB.

## Data this app requests

- `storage` capability — used only for the on-device jail above.
- `octos.turn.start` host service — invoked **only when the user taps the
  Refresh button**. Each invocation sends exactly one short prompt to
  this app's own host-side agent:
  `In one short paragraph, summarize what you remember about my interests and preferences.`
  No individual chip text is forwarded; the digest is rebuilt server-side
  from the host agent's own memory of the app's prior turns.

## What this app does NOT do

- It does not request `network.hosts` and does not open any HTTP/HTTPS
  connection.
- It does not collect telemetry, analytics, or device identifiers.
- It does not auto-refresh on a timer; the prompt is gated by the Refresh
  button (startup invokes it once for first-render ergonomics — same
  one-line prompt).
- It does not collect passwords, PINs, codes, or any account field.

## Agent profile

`read-only` — the app never asks the host to write, sign, or publish.

## Honest degradation

If the host has no assistant, the app falls back to a local digest built
from `prefs.json`; the source line states this explicitly.

## Contact

https://github.com/Thneoly/memory-digest/issues

## Source

https://github.com/Thneoly/memory-digest