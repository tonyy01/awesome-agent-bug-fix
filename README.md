# awesome-agent-bug-fix

A collection of real-world bug fixes for AI agents and tools. Each entry documents the problem, root cause, and workaround.

## Fixes

| Tool | Issue | Workaround |
|------|-------|------------|
| Claude Desktop | Cowork workspace fails to start: "RPC pipe closed" | [Manual service start](./claude-cowork-rpc-pipe-closed/README.md) |
| WorkBuddy + OpenCode Go | 400 error: Missing x-opencode-session header | [Local proxy injection](./workbuddy-opencode-go-400/README.md) |

## Contributing

To add a fix:

1. Create a folder named `[tool]-[bug-slug]/`
2. Add a `README.md` with:
   - Problem description
   - Environment (OS, tool version)
   - Workaround steps
3. Submit a Pull Request

See [CONTRIBUTING.md](./CONTRIBUTING.md) for details.