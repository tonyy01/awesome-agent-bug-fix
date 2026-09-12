# Contributing

Contributions are welcome. This document explains how to add a bug fix entry.

## Entry Structure

Each fix should be a folder containing a `README.md` with:

1. **Problem** — What tool, what error, what context
2. **Environment** — OS, tool version, relevant configuration
3. **Workaround** — Step-by-step solution
4. **Why It Works** — Brief explanation of the root cause (optional but encouraged)
5. **Notes** — caveats, limitations, or follow-up items (optional)

## Folder Naming

Use the format: `[tool-name]-[bug-slug]`

Examples:
- `claude-cowork-rpc-pipe-closed`
- `workbuddy-opencode-go-400`

## How to Submit

1. Fork this repository
2. Create your entry folder and `README.md`
3. Update the main `README.md` table to include your fix
4. Submit a Pull Request

## Style Guidelines

- Be concise. Focus on actionable information.
- Include exact error messages and version numbers.
- Avoid unnecessary prose — bullet points are preferred.
- If citing external issues or sources, provide links.