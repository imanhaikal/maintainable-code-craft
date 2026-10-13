# maintainable-code-craft

A Claude skill that helps Claude write, change, refactor and review code so it stays easy to understand and easy to change.

It combines the main ideas from three books into one working method:

- **A Philosophy of Software Design** by John Ousterhout
- **The Pragmatic Programmer** (20th Anniversary Edition) by David Thomas and Andrew Hunt
- **Refactoring** (2nd Edition) by Martin Fowler

The skill is written in my own words as a practical guide. It does not reproduce the books, and it is not affiliated with or endorsed by the authors or publishers. If you find it useful, read the books. They go much deeper than any summary can.

## What's inside

The skill lives in [`maintainable-code-craft/SKILL.md`](maintainable-code-craft/SKILL.md) and covers:

1. Core principles: what complexity is, where it comes from, and strategic vs tactical programming
2. A workflow for writing new code
3. A workflow for changing existing code safely
4. Module and API design: deep modules, information hiding, decoupling, inheritance
5. Functions and control flow, including when to split or merge functions
6. Data and state
7. Error handling, contracts and resource management
8. Naming
9. Comments and documentation
10. Testing
11. Performance
12. Concurrency
13. Security basics
14. Consistency and day-to-day habits
15. A table of 33 red flags and code smells, each with its fix
16. A quick reference for the refactoring catalog
17. A review checklist to run before calling code done
18. How to resolve the places where the three books disagree

It works with any programming language.

## Installation

### Claude Code

Copy the skill folder into your personal skills directory:

```bash
git clone https://github.com/imanhaikal/maintainable-code-craft.git
cp -r maintainable-code-craft/maintainable-code-craft ~/.claude/skills/
```

Or put it in a project's `.claude/skills/` folder to share it with your team through that repo.

### Claude apps (claude.ai, desktop)

Zip the `maintainable-code-craft` folder (the one containing `SKILL.md`) and upload it in Claude's skills settings.

## Usage

There's nothing to call. Claude loads the skill automatically when you ask it to write, fix, refactor or review code. You can also ask for it by name, for example: "Use maintainable-code-craft to review this module."

## License

[MIT](LICENSE)
