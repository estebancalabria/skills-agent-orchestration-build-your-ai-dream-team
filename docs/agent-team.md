# Agent team

Mona's Project Pulse dashboard is built with GitHub Copilot CLI in a
Codespace, using an Orchestrator to coordinate specialized custom agents.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 | Turns the requested dashboard outcome into dependency-aware phases, delegates explicit file scopes to specialists, coordinates parallel and sequential work, and verifies the integrated result. | [.github/agents/orchestrator.agent.md](../.github/agents/orchestrator.agent.md) |
| Planner | Claude Opus 5.5 | Researches the repository and relevant documentation, then produces implementation steps, file assignments, dependencies, risks, edge cases, validation expectations, and open questions. | [.github/agents/planner.agent.md](../.github/agents/planner.agent.md) |
| Designer | Gemini 3.7 Flash | Defines and implements accessible, responsive UI/UX styling: project cards, status badges, priority hierarchy, clear visual affordances, and stable CSS hooks such as `.dashboard` and `.project-card`. | [.github/agents/designer.agent.md](../.github/agents/designer.agent.md) |
| Coder | GPT-5.5 | Implements the assigned application logic and fixes with explicit errors and testable behavior. For runnable Project Pulse work, it can configure `.vscode/launch.json` to launch `app/index.html`. | [.github/agents/coder.agent.md](../.github/agents/coder.agent.md) |

The workflow begins with planning, then the Orchestrator assigns
non-overlapping implementation and design tasks in parallel when possible.
Tasks with shared files or dependencies run sequentially. Every agent remains
within its assigned scope, and the learner retains control of staging,
committing, and pushing changes.
