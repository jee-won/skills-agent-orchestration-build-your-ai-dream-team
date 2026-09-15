# Agent team

I will use GitHub Copilot CLI in a Codespace to orchestrate the custom agent team for Mona's Project Pulse dashboard.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| **Orchestrator** | Claude Opus 4.7 (copilot) | Coordinates the project, delegates work to the specialist agents, assigns file scopes and dependencies, runs parallel work safely, and verifies that the integrated result is coherent. | `.github/agents/orchestrator.agent.md` |
| **Planner** | Claude Opus 4.7 (copilot) | Researches the repository, documentation, dependencies, risks, edge cases, and validation needs, then produces an implementation plan without writing code. | `.github/agents/planner.agent.md` |
| **Coder** | GPT-5.5 (copilot) | Implements dashboard logic and other assigned code, follows repository patterns, adds required runnable-app support such as `.vscode/launch.json`, and validates the implementation. | `.github/agents/coder.agent.md` |
| **Designer** | Gemini 3.1 Pro (copilot) | Defines and implements the dashboard's UI/UX, accessibility, information hierarchy, interaction flow, responsive behavior, and visual styling. | `.github/agents/designer.agent.md` |

The Orchestrator will use the Planner's research to divide the work, then coordinate the Coder and Designer while keeping their file scopes explicit and resolving dependencies before integrating the final Project Pulse dashboard.
