---
description: Convert a PRD to Ralph's prd.json format for autonomous execution
argument-hint: "[path to PRD file]"
---

Load the `ralph` skill and convert the specified PRD into `prd.json` format.

If a file path is provided via $ARGUMENTS, read that PRD. Otherwise look for PRDs in `tasks/` and ask the user which one to convert.

Save the output to `scripts/ralph/prd.json`.
