# MCP tools grouped by platform, noun and safety class

The MCP server exposes one tool per platform, noun and safety class (`ig_post_read`, `ig_post_write`, each with a `method` named after the CLI verb), one guarded `<platform>_delete(kind, id, confirm)` per platform, plus `setup_guide` and `auth_status`: 37 tools instead of about 110, a departure from youtube-kit's one tool per operation. VS Code caps tools across all servers at 128 and Windsurf at 100, and clients approve per tool, never per argument, so the grouping follows the safety class: a client can allow every `_read`, prompt on `_write`, and put exactly one tool per platform behind its delete prompt. `_read` carries `readOnlyHint`; `_write` and `_delete` carry `destructiveHint: true` and `idempotentHint: false`, since `_write` holds overwrites too.

Each tool's input is the flat union of its methods' parameters, all defaulting to `null` and documented with the methods they apply to; the adapter drops the nulls, rejects a parameter the method doesn't take with that method's parameter list, and lets the operation's own defaults apply. The output is `{method, result}`. The adapter publishes its schema itself as an object root with a `oneOf` per method, because FastMCP otherwise wraps a union under a second `result` and strips the discriminator. That roughly triples each tool's `tools/list` entry (about 12k tokens across the server), accepted for a typed result. The server never validates its output, so a test checks every method's fixture output against its tool's schema.

## Considered options

- **One tool per operation**: breaks the client caps alongside any other server.
- **Dynamic toolsets** (`enable_toolset`, `tools/list_changed`): the 2026-07-28 spec forbids tool lists that change as a side effect of a call. `skit mcp --platform` and `--read-only` fix the list per connection instead.
- **Server-side code mode**: one `execute` call can fan out into a burst of live writes.
- **No output schema**: a third of the tokens, but clients and models lose the typed result.
