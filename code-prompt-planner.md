Claude Code Prompt Generator

Generate 1 or more focused, stateless Claude Code prompts that implement a development task in sequential steps. Each session has no memory of the previous one — only git history and the codebase are shared state.

Step 1 — Understand the Task

Gather from the user's message:

What needs to be done (feature, bug fix, refactor, tests)
Where in the codebase (files, directories, modules — if mentioned)
Constraints (naming conventions, patterns to follow, things to avoid)

If critical information is missing (e.g. no idea where the code lives, or the scope is extremely vague), ask one focused clarifying question before proceeding. Do not ask more than one question.

Step 2 — Decide on Prompt Count and Phases

Break the task into phases. Use this as a guide:

Complexity	Prompts
Trivial fix, single-file change, or isolated addition	1
Single-concern fix or small addition	2
Feature with model + logic + tests	3
Feature touching multiple layers (API, service, UI)	4
Large refactor or multi-component feature	5
Very large or highly complex task	6+ (ask first)

If the task seems to require more than 5 prompts, stop and ask the user: "This task looks complex enough to need more than 5 prompts. Is that okay, or would you like me to consolidate?" Only proceed with 6+ prompts if they confirm.

Principles for splitting:

Each phase must be completable independently by Claude Code with no conversation history
Each phase should touch a small, coherent slice of the codebase (e.g. data model only, then service layer, then API endpoint, then UI)
Avoid phases that require too much reasoning or too many unknowns — keep them specific
The final phase is always a cleanup/test-completeness pass if tests haven't been fully covered
Step 3 — Write Each Prompt

For every phase, produce a prompt following this structure:

Prompt Structure
## Prompt [N] of [TOTAL] — [Short Phase Title]

**Context**
[1–3 sentences explaining what has already been done (reference git, existing files, or prior phases). For Prompt 1, describe the starting state of the repo.]

**Task**
[Precise, scoped instructions. Use bullet points. Be explicit about file names, function names, or module paths if known. Avoid open-ended language like "implement as you see fit".]

**Constraints**
- Do not modify files outside the scope of this prompt
- Follow existing patterns in the codebase (naming, error handling, structure)
- [Any other task-specific constraints]

**Unit Tests**
At the end of your implementation, write or update unit tests that cover:
- [Specific behaviour 1]
- [Specific behaviour 2]
- [Edge case or error case relevant to this phase]
Run the tests and confirm they pass before finishing.

**Verification** *(include only when the human needs to manually check something)*
After completing this prompt, the developer should:
- [Concrete manual check — e.g. "run the app and navigate to /login to confirm the form renders"]
- [Or: "run `git diff` and confirm only these files were modified: ..."]
Step 4 — Rules for Good Prompts

Apply these rules to every prompt you write:

Statelessness

Never say "as discussed" or "from before" — Claude Code has no prior context
Always state the starting condition explicitly (what files exist, what was done in a prior phase)
Reference git history with phrases like: "The model was added in the previous commit — check git log --oneline -5 to orient yourself"

Scope control

Name the exact files to create or modify; do not leave it open-ended
Say explicitly what is out of scope for this prompt if there's risk of overreach
If a phase only touches one file, say so

Unit tests (every prompt)

Every prompt must end with a unit test block
Tests should be runnable in isolation — do not depend on results from previous prompts beyond what's in git
Specify the test runner if the project's stack is known; otherwise say "using whatever test framework is already in the project"

Verification steps (selective)

Include a verification block only when:
The phase produces a visible/runnable result (UI, endpoint, CLI output)
Or the human needs to make a judgement call before proceeding
Or there's a meaningful risk the phase could silently produce wrong output
Do not add verification steps to purely internal/logic phases where tests are sufficient

Tone

Write prompts in second-person imperative: "Create a file...", "Add a function...", "Update the schema..."
Be direct and unambiguous — these prompts will be pasted directly into a Claude Code session
Step 5 — Output Format

This skill is used on Claude web. Each prompt must be output as its own artifact (a separate markdown code block rendered by Claude's artifact system) so the user gets a dedicated one-click copy button per prompt.

Output in this order:

A one-line task summary (what the full sequence achieves)
A phase overview table in chat with prompt numbers, titles, and whether verification is needed
Each prompt as a separate artifact, titled Prompt N of TOTAL — Phase Title. Output them one after another with no extra commentary between them.
After all artifacts, a single closing line: "Paste each prompt into a fresh Claude Code session in order."

Each artifact contains only the prompt content — no meta-commentary inside the artifact itself.

Example phase overview table:

| # | Phase | Verification? |
|---|---|---|
| 1 | Add DB schema + migration | No |
| 2 | Implement service layer | No |
| 3 | Add REST endpoint | Yes — test with curl |
| 4 | Write integration tests | No |
Common Pitfalls to Avoid
Too broad: "Implement the authentication system" → bad. Split into: schema, service, endpoint, tests
Too vague about files: "Update the relevant files" → always name them when possible
Missing context: Prompt 2 that doesn't explain what Prompt 1 did → always include a one-sentence "what's already done" in Context
Runaway tests: Don't ask for exhaustive test suites in one prompt — scope tests to the phase
Over-verification: Not every prompt needs a human check — trust the tests for internal logic phases
