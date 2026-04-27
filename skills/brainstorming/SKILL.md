---
name: brainstorming
description: Use before creative work, new features, behavior changes, components, or ambiguous implementation requests to turn an idea into an approved design before coding.
---

# Brainstorming

## Overview

Turn an idea into a clear, approved design before implementation. Use the smallest process that resolves the important decisions: understand the context, ask focused questions, compare viable approaches, and get explicit approval before writing code.

## When To Use

- Creating features, components, flows, or user-facing behavior
- Modifying behavior where intent, constraints, or tradeoffs are unclear
- Turning a vague idea into a concrete design
- Deciding between multiple possible implementation approaches
- Planning work that touches several files, modules, or user interactions

## When Not To Use

- Pure bug fixes where the expected behavior is already clear
- Mechanical refactors with no product or design decision
- Code review requests
- Direct informational questions
- Requests where the user explicitly asks to skip planning

## Core Rules

1. Explore the project context first when the answer depends on existing code, docs, or conventions.
2. Ask one clarifying question at a time.
3. Provide your recommended answer with each question.
4. Prefer multiple choice questions when possible.
5. Do not implement until the user approves the design.
6. Keep the process as small as the task allows.
7. If a question can be answered by inspecting the codebase, inspect the codebase instead of asking the user.

## Workflow

1. Explore context: inspect relevant files, docs, and existing patterns when applicable.
2. Summarize what is already known in a few sentences.
3. Assess scope: if the request spans multiple independent systems, recommend decomposing it before continuing.
4. Ask clarifying questions one at a time until the key decisions are resolved.
5. Propose 2-3 approaches with tradeoffs.
6. Recommend one approach and explain why.
7. Present the design at the right level of detail for the task.
8. Ask the user to approve or revise the design.
9. After approval, ask whether to proceed with implementation.

## Question Strategy

Resolve decisions in dependency order:

1. Goal: what problem are we solving?
2. Users: who is this for?
3. Success criteria: what must be true when this is done?
4. Scope: what is included and excluded?
5. Constraints: technical, UX, performance, security, compatibility, or timeline limits.
6. Behavior: what should happen in the main path and edge cases?
7. Integration: where should this fit in the existing system?
8. Verification: how will we know it works?

Prefer questions that make progress:

- Ask about decisions that affect the design.
- Avoid asking for information already available in the codebase.
- Offer concrete choices instead of open-ended prompts when practical.
- Include a recommendation so the user can accept, reject, or adjust it quickly.

## Approach Comparison

Before presenting the final design, compare 2-3 viable approaches.

For each approach, cover:

- What it is
- Why it might be good
- Main tradeoff or risk
- When it would be the right choice

Lead with the recommended approach unless another ordering makes the tradeoff clearer.

## Design Approval Gate

Present the design before implementation. Scale the detail to the task.

For small changes, the design may be a short paragraph covering intent, affected area, and verification.

For larger changes, cover:

- Goal
- Non-goals
- User or system behavior
- Affected areas or architecture
- Data flow or state flow, if relevant
- Error handling, if relevant
- Testing or verification
- Open risks

Ask for explicit approval before implementation.

## Outputs

Default output is an approved design in chat.

Write a spec file only when the user explicitly requests it. Do not commit anything unless the user explicitly asks.

## Red Flags

- Starting implementation before the design is approved
- Asking broad questions that do not resolve a decision
- Skipping project exploration when existing code should answer the question
- Treating a large multi-system idea as one small feature
- Leaving success criteria vague
- Presenting only one approach when real alternatives exist
- Adding documentation or process that is heavier than the task needs
