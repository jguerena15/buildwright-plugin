---
name: kickoff
description: "Start work from a ClickUp task URL. Reads the task, extracts requirements and links, then decides whether to generate a PRD + Ralph loop or handle it directly. Triggers on: clickup url, start this task, kickoff, CU task link."
user-invokable: true
---

# Kickoff from ClickUp

Takes a ClickUp task and automatically determines the right workflow.

---

## Step 1: Read the ClickUp Task

Use the ClickUp MCP to fetch the task details:

1. Extract the task ID from the URL
2. Get the task title, description, comments, and any attachments
3. Collect all links found in the task (Figma, docs, etc.)

If ClickUp MCP is not available, stop and let the user know:

> ClickUp MCP isn't set up yet. Run `/mcp` to authenticate with ClickUp, or paste the task details directly.

---

## Step 2: Assess the Task

Evaluate the task to determine the right workflow. Consider:

- **Scope**: Is this a single, focused change or a multi-step feature?
- **Clarity**: Are there clear requirements/acceptance criteria?
- **Dependencies**: Does it involve multiple files, schema changes, backend + frontend?

### Route A: Simple Task (single story)

Use this route when:
- The task is a single, focused change (bug fix, one component, one page)
- It can be completed in one session without breaking it down further
- Requirements are clear enough to start immediately

**Action**: Skip PRD generation. If the task has Figma links, load the `build-ui` skill and start building. Otherwise, just implement it directly.

### Route B: Complex Feature (multi-story, Ralph-worthy)

Use this route when:
- The task describes a feature with multiple parts
- It involves schema + backend + frontend work
- It would benefit from being broken into ordered user stories
- It would take more than one focused session to complete

**Action**:
1. Load the `prd` skill to generate a PRD from the task details
2. Save to `tasks/prd-[feature-name].md`
3. Load the `ralph` skill to convert the PRD to `scripts/ralph/prd.json`
4. Inform the user the Ralph loop is ready and how to run it:
   ```
   ./scripts/ralph/ralph.sh
   ```

---

## Step 3: Extract and Follow Links

Regardless of route, extract all useful links from the task:

- **Figma links**: Note them for the `build-ui` skill to use
- **Doc links**: Read them for additional context
- **API/endpoint references**: Note for implementation

For Route A with Figma links: use Figma MCP to pull the design before building.

For Route B: include Figma links in the relevant user stories' acceptance criteria (e.g., "Build component matching Figma link: [url]. Verify in browser using Playwright MCP").

---

## Step 4: Inform the User

After routing, clearly tell the user what you decided and why:

- What route was chosen (simple vs Ralph)
- What links were found and how they'll be used
- What happens next (building directly, or PRD → Ralph)
- If Ralph: remind them to review the PRD and prd.json before running the loop
