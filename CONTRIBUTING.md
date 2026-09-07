# Contributing

Contributions welcome! Here's how to add your bug fix story.

## What to Include

Each entry should contain:

1. **Problem** — What was the error? What were you trying to do?
2. **Environment** — OS, tool version, relevant configuration
3. **Diagnosis Process** — How did you find the root cause? What logs did you check?
4. **Root Cause** — What was actually wrong?
5. **Fix** — The solution, with code/steps
6. **Lessons Learned** — What would you do differently next time?

## Template

```markdown
# [Tool Name]: [Error Summary]

## Problem
[Description of the error]

## Environment
- **OS:** [e.g., Windows 11, macOS 14]
- **Tool:** [e.g., Claude Desktop 1.46388.4.0]

## Diagnosis Process
[Step-by-step how you found the root cause]

## Root Cause
[What was actually wrong]

## Fix
[The solution]

## Lessons Learned
[Key takeaways]
```

## Quality Guidelines

- **Be specific** — Include exact error messages, file paths, commands
- **Be honest** — Include dead ends and failed attempts
- **Be helpful** — Explain *why* the fix works, not just *what* to do
- **Be concise** — Cut anything that doesn't help someone else

## How to Submit

1. Fork this repository
2. Create a new directory: `./[tool-name]-[bug-slug]/`
3. Add your `README.md` using the template
4. Update the main `README.md` table
5. Submit a Pull Request

## Code of Conduct

- No blaming or shaming tool developers
- Focus on helping others, not showing off
- Respect privacy — anonymize sensitive details
