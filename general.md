# General

- Keep the plan current. When a plan exists (plan mode or a plan file in the repository), apply the outcome of every design decision (an answered question, an accepted recommendation) to the plan in the same turn.
- Don't guess on decisions that matter. When a choice is ambiguous and costly to reverse, ask before writing anything that depends on it. Ask with the `AskUserQuestion` tool when it is available, not as a list of questions at the end of a chat reply, and include a recommended option. For minor choices, pick a sensible default and state the assumption.
- Never commit or push without asking. When work is ready, ask via `AskUserQuestion` and proceed only on a yes. Leave changes in the working tree and offer to commit/push at natural checkpoints. Approval for one commit doesn't cover the next.
