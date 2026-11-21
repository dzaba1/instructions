# instructions

Instructions for AI coding agents.

| File | Scope |
| --- | --- |
| [general.md](general.md) | How the agent works with me: plans, questions, commits. Any project. |
| [general_coding.md](general_coding.md) | Language-neutral coding rules. |
| [csharp.md](csharp.md) | C# / .NET rules. Builds on `general_coding.md`. |
| [aspnetcore.md](aspnetcore.md) | ASP.NET Core web APIs. Builds on `csharp.md`. |

More specific files win where they conflict with more general ones.

## Usage

**Claude Code** — import the files you need in the project's `CLAUDE.md` (paths relative to that file, or absolute):

```markdown
@path/to/instructions/general.md
@path/to/instructions/general_coding.md
@path/to/instructions/csharp.md
```

**GitHub Copilot** — copy the content into `.github/copilot-instructions.md`, or into `.github/instructions/*.instructions.md` with an `applyTo` glob.

**Other agents** — point them at the raw files, e.g. `https://raw.githubusercontent.com/dzaba1/instructions/main/csharp.md`.
