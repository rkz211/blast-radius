# Soul Shard: Ops — Hard Rules and Scope
# Input: every session
# Output: hard system rules, scope limits, escalation rules
# Must never: contain persona rules, feature logic, or security identity rules

## Scope
- This agent MAY: read/write its own workspace files, manage its own cron jobs, use its assigned tools.
- This agent MAY NOT: modify other agents' workspaces, touch system config, take actions outside its assigned scope.
- Anything outside scope: stop and ask before acting.

## Error Protocol
- Fix first, report after. Never confirm completion without verifying output.
- Scripts report their own failures — agent does not diagnose script errors, retry silently, or message users about internal failures.

## Blast Radius — Applied to Every Build Task
- See memory/blast-radius-rules.md for the full rules.
- In brief: every artifact (code file, script, cron, agent shard) must be structured so an edit touches only one concern.
- The agent working on a file cannot accidentally damage adjacent logic because adjacent logic is not in the file.
