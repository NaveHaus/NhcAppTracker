# Context, Rules, and Guidelines for AI Agents
## Overview

## Important References

## Sections
- [Tech stack](#tech-stack)
- [Safety & Operational Rules (MANDATORY)](#safety--operational-rules-mandatory)
- [Terminology](#terminology)
- [Process](#process)
  - [Requirements (MANDATORY)](#requirements-mandatory)
- [Code Style](#code-style)
  - [Requirements (MANDATORY)](#requirements-mandatory-1)
- [Testing](#testing)
  - [Requirements (MANDATORY)](#requirements-mandatory-2)
  - [Edit-Build-Test Commands (MANDATORY)](#edit-build-test-commands-mandatory)
- [Example OpenSpec Workflows](#example-openspec-workflows)
  - [One-Shot Implementation](#one-shot-implementation)
  - [One-Shot Exploration to Implementation](#one-shot-exploration-to-implementation)
  - [Iterative Exploration to Implementation](#iterative-exploration-to-implementation)

## Tech stack
- C# 10
- GitHub Flavored Markdown

## Safety & Operational Rules (MANDATORY)
To maintain repository integrity, agents MUST follow these rules:
- **No Secrets**: NEVER commit secrets, API keys, or credentials.
- **Git Safety**:
  - DO NOT modify git configuration.
  - DO NOT use `--no-verify` or bypass hooks unless explicitly requested.
  - DO NOT force push to protected branches, e.g. `master` or `main`.
  - DO NOT automatically fix `git` errors---ALWAYS ask the user for confirmation first.
  - DO warn the user if attempting to commit to `master` or `main`.
  - DO warn the user if `git` returns an error or reports that the remote branch is missing.
  - DO offer to resolve `git` errors, presenting the user with 1-4 options for doing so.
- **Path Handling**: Always use absolute paths when interacting with file system tools.
- **Build Consistency**: DO NOT directly modify or remove any files or directories listed in `.gitignore`.
- **Verification**: Always run build and test commands after modifications.

## Terminology
- **OpenSpec**: An artifact-driven workflow for managing software changes (features, fixes, etc.) through structured specifications and tasks.
- **openspec-***: Agent skills for interacting with an OpenSpec change workflow.
- **convetional-commits**: An agent skill for generating a `git` commit message following the Conventional Commits v1.0 specification.
- **nhc-openspec-commit**: A skill that uses the `conventional-commits` skill to generate compliant `git` commit messages summarizing the changes made by an OpenSpec workflow.

## Process
### Requirements (MANDATORY)
- An OpenSpec workflow MUST be used to plan new features and modifications of existing features (e.g. bug fixes). Warn the user if the `openspec` directory is missing or inaccessible.
- A test-driven development (TDD) RED/GREEN/REFACTORING workflow using a `tdd` skill is required to test ALL code additions and modifications. Warn the user if the `tdd` skill is missing or inaccessible.
- The `nhc-openspec-commit` skill MUST be used to generate a `git` commit message and commit changes made WITH an OpenSpec workflow, but only AFTER the OpenSpec change has been archived. Warn the user if the `nhc-openspec-commit` skill is missing or inaccessible.
- The `conventional-commits` skill MUST be used to generate a `git` commit message for changes made OUTSiDE of an OpenSpec workflow. Warn the user if the `conventional-commits` skill is missing or inaccessible.
- NOTE: If unsure about which commit strategy applies, you MUST ask the user to avoid generating spurious or erroneous commits.
### Major Features
- Major features are comprised of two or more distinct OpenSpec changes, e.g. an MVP for a new front-end user interface or a new back-end service.
- You MUST use `openspec-explore` develop a high-level plan for implementing the individual changes comprising a major feature.
- You MUST use the `docs/plans/<major-feature-name>` directory as a workspace during `openspec-explore`:
  - You MUST ignore ALL files in `docs/plans/archive` unless the user explicitly requests to review them.
  - Before creating planning documents, you MUST **ask the user** to provide the name of the major feature if the user has not already provided one.
  - You MUST review existing planning documents for context while developing `<major-feature-name>`.
  - You MUST offer to record decisions, constraints, milestones, dependencies, or any other important information relevant to planning the major feature.
  - You MUST offer to create or update a current list of changes as they become clear.
  - You MUST include a dependency diagram when recording the current list of changes, if the dependencies are clear.
  - You MAY store high-level planning files directly under `docs/plans`, e.g. project-level guidance or decisions rather than feature-level information.
  - DO NOT store OpenSpec artifacts under `docs/plans/<major-feature-name>`---the directory MUST ONLY contain Markdown files required to plan the changes comprising `<major-feature-name>`.
  - You MAY offer to create the OpenSpec artifacts for a concrete, implementable change.
  - Prefer `openspec-ff-change` over `openspec-new`/`openspec-continue` when creating change artifacts for a major feature.


Most common sequence of `openspec` operations:
1. `openspec-new <name>`: Initialize the change.
2. `openspec-ff-change <name>`: Generate/update artifacts.
3. `nhc-openspec-refine <name>`: Make artifacts implementation-ready.
4. `openspec-apply-change <name>`: Implement the tasks (following TDD process - see [Testing](#testing)).
5. `dotnet tests`: Verify all tests before completing a code change.
6. `nhc-opsx-verify <name>`: Validate implementation against specs (see [Error Handling Guidance](#error-handling-guidance) if this fails).
7. `openspec-sync-specs <name>`: Make OpenSpec specs artifacts permanent.
8. `openspec-archive-change <name>`: Archive the OpenSpec change.
9. `git add ./openspec/ <paths-to-changed-files-and-directories>`: Stage the archived OpenSpec artifacts and associated changes.
10. `nhc-openspec-commit`: Generate a `conventional-commits` `git` commit message based on the staged changes and complete the commit.

## Testing
### Requirements (MANDATORY)
- Test-driven development (TDD) with a `tdd` skill MUST be used when changing or adding code.
- ALL changes to existing code MUST be tested by updating ALL relevant existing tests following TDD.
- Testing MAY be skipped for non-code changes, e.g. documentation-only changes.
- Tests MUST be stored under `tests/<category>`, where `<category>` MUST be the name of the library or tool containing the feature/class under test.

## Example OpenSpec Workflows
**MANDATORY** OpenSpec test tasks in `tasks.md` MUST follow test-driven development (TDD) using a `tdd` skill.

### One-Shot Implementation
Can be used for simple changes that require no investigation or decision making prior to implementation:
- Generate and review the OpenSpec change artifacts in one shot:
  - `openspec-new-change <change-name>`
  - `openspec-ff-change <change-name>`
  - `nhc-openspec-refine <change-name>`
- Implement and verify the change:
  - `openspec-apply-change <change-name>`
  - See [Edit-Build-Test Commands (MANDATORY)](#edit-build-test-commands-mandatory) for the required commands to use to implement the change.
  - `nhc-opsx-verify <change-name>`
- Archive the completed OpenSpec change:
  - `openspec-sync-specs <change-name>`
  - `openspec-archive-change <change-name>`
- Locally commit the working OpenSpec artifacts and associated project changes:
  - `git add ./openspec/ <changed-files-and-or-directories>`, e.g.:
    ```bash
    git add ${pwd}/openspec ${pwd}/tests
    ```
  - Use the `nhc-openspec-commit` skill to complete the `git` commit.

### One-Shot Exploration to Implementation
Can be used for straightforward changes that require some upfront investigation and/or decision making prior to implementing.
- Interactively research and investigate a change with the user:
  - `openspec-explore <topic>`
- Generate and review the OpenSpec change artifacts in one shot:
  - `openspec-new-change <change-name>`
  - `openspec-ff-change <change-name>`
  - `nhc-openspec-refine <change-name>`
- Implement and verify the change:
  - `openspec-apply-change <change-name>`
  - See [Edit-Build-Test Commands (MANDATORY)](#edit-build-test-commands-mandatory) for the required commands to use to implement the change.
  - `nhc-opsx-verify <change-name>`
- Archive the completed OpenSpec change:
  - `openspec-sync-specs <change-name>`
  - `openspec-archive-change <change-name>`
- Locally commit the working OpenSpec artifacts and associated project changes:
  - `git add ./openspec/ <changed-files-and-or-directories>`, e.g.:
    ```bash
    git add ${pwd}/openspec ${pwd}/tests
    ```
  - Use the `nhc-openspec-commit` skill to complete the `git` commit.

### Iterative Exploration to Implementation
Can be used for complex changes or changes with unclear requirements.
- Interactively research and investigate a change with the user:
  - `openspec-explore <topic>`
- Iteratively generate the OpenSpec change artifacts:
  - `openspec-new-change <change-name>`
  - `openspec-continue-change <change-name>` iteratively and interactively with the user until all artifacts have been accepted.
- Review the artifacts before implementation:
  - `nhc-openspec-refine <change-name>`
- Implement and verify the change:
  - `openspec-apply-change <change-name>`
  - See [Edit-Build-Test Commands (MANDATORY)](#edit-build-test-commands-mandatory) for the required commands to use to implement the change.
  - `nhc-opsx-verify <change-name>`
- Archive the completed OpenSpec change:
  - `openspec-sync-specs <change-name>`
  - `openspec-archive-change <change-name>`
- Locally commit the working OpenSpec artifacts and associated project changes:
  - `git add ./openspec/ <changed-files-and-or-directories>`, e.g.:
    ```bash
    git add ${pwd}/openspec ${pwd}/tests
    ```
  - Use the `nhc-openspec-commit` skill to complete the `git` commit.
