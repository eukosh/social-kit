# One core, two adapters, the CLI built from the registry

Every operation is a plain, synchronous, typed function in its platform package, registered by `@operation` with its platform, noun, verb and safety class. `cli.py` (cyclopts) builds the `skit <platform> <noun> <verb>` tree from that registry, and `mcp_server.py` (FastMCP) groups the same functions into tools (ADR 0004). This is youtube-kit's split: a CLI costs an agent no context until it is called and composes with `jq`, MCP serves clients without a shell, and one signature and docstring are the single source of truth for flags, help and schemas. Core never prints, exits or imports either library; it raises errors carrying an exit code, type and reason.

## Considered options

- **Generating the CLI from the MCP server** (`fastmcp generate-cli`): every call spawns the server and runs a handshake, the generated file goes stale, and it drops empty-list arguments.
- **Naming each command by hand in `cli.py`**, as youtube-kit does: the registry already holds platform, noun and verb, so a hand list could only drift from it.
