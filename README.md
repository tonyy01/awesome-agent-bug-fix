# awesome-agent-bug-fix

A curated collection of bug fixes for AI agents and tools. Each entry documents the problem, diagnosis process, root cause, and fix.

## Why This Exists

When AI tools break, the fix is often simple — but finding it isn't. This repository collects debugging stories that save others time and teach systematic problem-solving.

## How to Use

- **Found a bug in an AI tool?** Check if someone has already fixed it here.
- **Fixed a bug yourself?** Add your story to help others.
- **Learning to debug?** Read through the cases to build your intuition.

## Entries

| Tool | Bug | Root Cause | Fix |
|------|-----|------------|-----|
| [Claude Desktop](./claude-desktop-cowork/README.md) | Cowork workspace "RPC pipe closed" | MSIX packaging corrupts Authenticode signature | Manual service start with `xcopy /G` |

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

## License

MIT
