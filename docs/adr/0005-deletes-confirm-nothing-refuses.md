# Deletes confirm, nothing refuses

Deletes are guarded only by a confirmation bound to the target: without `--confirm ID` (MCP `confirm`) core exits 8 with the plan and the exact rerun, a wrong id is exit 2, and a target already gone is success. Unlike youtube-kit's ADR 0002 there are no damage limits: nothing is refused for its age, reach or views, and exit 9 stays unused, because the owner decides what to delete and a refusal only sends them to the app to do it there. Publishing, replies and DMs aren't guarded in code at all: the skill asks the user, and accepts standing permission for an automation the user wrote down, so scripts can call `skit` directly. After a burst of test posts got a Threads account suspended, a local write ledger was added, and it too only warns (a stderr warning and `details.pacing` in the result), never blocks.

## Considered options

- **youtube-kit's damage limits**: rejected by the owner; no flag would override them.
- **`--yes`**: agents learn to append it by reflex, and it fits any target, while `--confirm ID` names one.
- **Refusing writes past the pacing limits**: breaks the user's own automations; pacing belongs to the skill and to Meta's limits.
