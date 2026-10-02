# Memory Digest (memory-digest) — brief

The reader demo of the shared-memory system: opens with a digest built
from what the app's own assistant remembers about the person, showing how
any store app personalizes from the memory layer. Base works without an
AI (a local interest list); the assistant digest degrades honestly.

## Screens

1. **Main** — title "Memory Digest"; a large digest card: heading
   "Your digest", the digest body text, and a source note ("From your
   assistant" vs "From your local interests — no assistant on this
   device"); below it, the local interests editor (entry + Add, tap a
   chip to remove) that the base digest is built from; hint line.

## Actions

- **Ask the assistant** — a "Refresh" button: sends
  `host.request("octos.turn.start", {text: "In one short paragraph,
  summarize what you remember about my interests and preferences."})`;
  on success the card shows the reply (`r.data.text`) attributed to the
  assistant; on failure it falls back to the local digest with the
  honest note.
- **Manage local interests** — add an interest (entry + Add, empty
  ignored); tap an interest chip to remove it; stored in `prefs.json`;
  the base digest renders them as a sentence list.

## Data

- `prefs.json` in the jail: array of strings (interests).

## States

- No interests, no assistant: card explains both and how to start.
- Interests, no assistant (card-host): "You like: a, b, c." with the
  local note.
- Assistant present: its paragraph replaces the local text.
- Restart persistence for interests.

## Hosts

None.

## Capabilities and why

- `storage` — `prefs.json` in the jail.
- `octos.turn.start` — the Refresh action, this app's own agent only.
- `agent` block (`profile: read-only`) — declares the agent (what the
  writer demo and os.memory sync into; this app reads it back).

## Not in this app

- Reading any other app's data directly (impossible by isolation; the
  sync happens through the system assistant, user-approved).
