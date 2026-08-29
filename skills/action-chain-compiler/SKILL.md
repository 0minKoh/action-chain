---
name: action-chain-compiler
description: Compile a successfully completed task from the current conversation into a reusable and portable Codex skill. Use when the user asks to turn work just performed into a skill that can be reused by them or others.
---

# Action Chain Compiler

Analyze the successfully completed task in the current conversation and create a reusable skill from it.

The generated skill must be portable across users and environments. Preserve the workflow, but separate it from the original user's credentials, permissions, paths, accounts, and local configuration.

## Procedure

1. Identify the user's original goal and the successful final workflow.
2. Separate the workflow into:
   - decisions that still require an agent
   - repeatable deterministic actions
3. Identify user-specific and environment-specific dependencies, including:
   - credentials and API keys
   - accounts and permissions
   - file paths
   - service endpoints
   - tools, packages, and runtime requirements
4. Create a new skill containing:
   - `SKILL.md` for inputs, requirements, decisions, procedure, and validation
   - `scripts/` only when reusable executable logic exists
5. Replace task-specific and user-specific values with explicit inputs or configuration.
6. Validate the generated skill by running its scripts when safe and practical.

## Portability

Design the generated skill so another user can configure and run it without access to the original user's environment.

- Accept variable values through arguments, environment variables, or configuration.
- Document required credentials, permissions, tools, and setup without including secret values.
- Use portable paths and configurable endpoints where practical.
- Distinguish required dependencies from optional or environment-specific ones.
- Detect missing configuration and report what the user must provide.
- Request authorization when the workflow requires access or permissions not already granted.

## Rules

- Compile only steps supported by the conversation and execution results.
- Preserve the successful workflow, not failed attempts or exploratory commands.
- Never copy credentials, tokens, cookies, private identifiers, or other secrets into the generated skill.
- Do not assume another user has the original user's accounts, permissions, tools, paths, or environment.
- Do not hard-code values that are expected to vary between users or executions.
- Do not convert decisions or exception handling into fixed code unless they are deterministic.
- Do not create scripts when instructions alone are sufficient.
- Report what was generated, what users must configure, and what could not be validated.

