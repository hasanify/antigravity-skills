---
name: get-shit-done
description: A context engineering and spec-driven development framework for structured project planning and execution. Use this skill to systematically build, plan, execute, and verify complex projects without context degradation.
---

# Get Shit Done (GSD) System

## Goal
To implement the GSD meta-prompting methodology, acting as a structured orchestrator for software development. This skill ensures reliable code generation by relying on persistent files for long-term memory, decomposing work into atomic tasks, executing in isolated waves, and enforcing strict requirement verification to prevent context rot.

## When to use this skill
- Whenever user invokes or runs the `/get-shit-done` command.
- When starting a new feature or project to ensure a robust architectural foundation.
- When an existing project needs a formalized roadmap and rigorous execution planning.
- When you need to avoid "yolo" coding and want a methodical, step-by-step development process.

## Core Principles
1. **Context Engineering:** Rely exclusively on standardized markdown files (`PROJECT.md`, `ROADMAP.md`, `STATE.md`) rather than accumulating conversational chat history. 
2. **Fresh Context Execution:** Treat each implementation task as an isolated unit to avoid token bloat.
3. **Atomic Everything:** Each task requires a narrowly defined scope and its own atomic Git commit.
4. **Goal-Backward Verification:** Always verify work by working backward from the necessary artifacts to ensure requirements are met, rather than just checking if code superficially runs.

## Instructions: The GSD Workflow

You are expected to act as the GSD Orchestrator and guide the user through these sequential phases:

### Phase 1: Initialization
1. Ask targeted questions until you deeply understand the user's vision, constraints, stack preferences, and edge cases.
2. Filter the scope into v1, v2, and out-of-scope.
3. Generate the core memory files in a `.planning/` directory:
   - `PROJECT.md`: The core vision and technical specifications.
   - `REQUIREMENTS.md`: Scoped requirements with clear traceability.
   - `ROADMAP.md`: Sequential phases mapped to the extracted requirements.
   - `STATE.md`: Tracker for progress, locked decisions, and blockers.

### Phase 2: Discuss & Context
1. Analyze the upcoming phase from `ROADMAP.md`.
2. Identify gray areas based on the domain (e.g., visual density, API response formats, error flows).
3. Present these questions to the user to eliminate ambiguity before writing code.
4. Record the final, locked decisions into `CONTEXT.md`. 

### Phase 3: Research & Plan
1. **Research:** Investigate the stack and patterns using the locked decisions in `CONTEXT.md`.
2. **Decompose:** Create 2-3 atomic task plans using structured XML formats (e.g., `<task type="auto">`).
3. **Budget:** Ensure each plan modifies a maximum of 3-5 files to stay within a small, safe execution context limit.
4. **Draft:** Save the output to `.planning/{phase}-{N}-PLAN.md`.
5. **Verify:** Rigorously cross-check the drafted plan against `REQUIREMENTS.md`. If it drops any user requirements, revise the plan immediately.

### Phase 4: Execution
1. Build dependency graphs for the planned tasks and group them into Execution Waves.
2. **Independent Plans:** Execute in parallel (where the environment allows).
3. **Dependent Plans:** Execute sequentially to respect dependencies.
4. **Commit:** Generate an atomic Git commit strictly tied to each executed task's boundaries. Do not bundle disparate changes into a single commit.

### Phase 5: Verification & Review
1. Perform a multi-source coverage audit. 
2. Verify artifacts exist, are wired correctly, and are substantive (no dummy returns, `TODO` blocks, or empty components).
3. Prompt the user for real-world verification (e.g., "Does the login button work?").
4. If tests or human verification fail, automatically propose a fix plan, and loop back to execution.
5. Update `STATE.md` with completion metrics and clear the active phase.

## Constraints
- **Strictly enforce locked decisions.** If `CONTEXT.md` specifies a library or pattern, you must not hallucinate alternatives.
- **Never silently drop scope.** If a requirement cannot fit the context budget, suggest splitting the phase into smaller sub-phases. 
- **Do not combine tasks.** Each task must remain atomic to allow isolated git commits and easy rollbacks.
- **Prevent schema drift.** If ORM schema files change, ensure migrations are included in the task plan.
- **Never proceed to execution without an approved plan.** Planning must always precede coding.
- **Always verify against requirements.** If the plan does not meet all requirements, it must be revised before execution.
