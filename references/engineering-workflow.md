# Engineering Workflow

## Project Rules

- Before modifying code, check applicable project-level rule files such as `AGENTS.md`, `CLAUDE.md`, `SKILL.md`, `CONTRIBUTING.md`, `README.md`, `.github/copilot-instructions.md`, and `.github/instructions/*`.
- If multiple project rules conflict, prefer the more specific rule located closer to the files being changed.
- Read applicable project rules and relevant code before proposing or making changes.
- Do not invent project rules or conventions that do not exist.

## Permission Boundaries

### May Perform Without Additional Permission

- Read, search, and analyze project files.
- Perform task-relevant read-only web searches and access public documentation.
- Perform read-only access to public repository hosting services such as GitHub and GitLab.
- Run read-only Git network operations such as `git fetch` and `git remote show`.
- Modify ordinary source-code files within the explicit task scope.
- Run existing tests, build, lint, typecheck, and formatter commands.
- Create a temporary or feature branch from the repository's detected primary development branch.
- Prepare commit contents, show the proposed diff, and suggest a commit message.
- Perform small, clearly scoped refactors required by the task.

### Must Obtain Permission First

- Install, remove, or update dependencies.
- Modify a lockfile.
- Delete files, delete data, clear caches, overwrite many files, or move files in bulk.
- Format the entire repository.
- Modify CI/CD, GitHub Actions, deployment configuration, production systems, or release configuration.
- Modify a database, run a migration, or perform other persistent data changes.
- Execute `sudo`.
- Use authentication, tokens, API keys, secrets, SSH private keys, or cloud credentials.
- Call third-party APIs that produce side effects.
- Download and execute external files.
- Upload data or modify remote state.
- Execute `git commit`, `git push`, create a Pull Request, publish a package, create a Release, or deploy.
- Modify `main` or `master`.
- Perform any force push. Refuse force pushes to `main` or `master` even if requested.
- Modify `LICENSE`, CLA, security policy, copyright statements, or similar governance files.

Treat read-only operations as allowed by default. Decide whether other writes need permission based on their risk, permanence, and blast radius.

## Default Workflow

For software engineering tasks:

1. Read project-level instructions and conventions.
2. Read the relevant code and tests.
3. Build an evidence-based understanding of the problem.
4. Briefly state the plan when the task is non-trivial.
5. Make the smallest coherent change that solves the task.
6. Run focused validation.
7. Report changes, validation results, potential risks, and unfinished items.

For tasks exceeding roughly 10 minutes or containing clearly distinct stages, provide periodic, meaningful progress updates. Do not report every minor action.

## Implementation Preferences

Prioritize:

1. Correctness
2. Security
3. Readability
4. Maintainability
5. Backward compatibility
6. Minimal dependencies
7. Performance

Prefer:

- Small, focused patches.
- Clear naming and readable control flow.
- Few dependencies and no unnecessary frameworks.
- Existing project patterns, abstractions, and conventions.
- Changes limited to the current issue or request.

Avoid:

- Unrelated or architecture-wide refactors.
- Over-abstraction and magic code.
- Sacrificing readability for elegance.
- Introducing C++ when it is not necessary.
- Large formatting-only diffs or modifications to unrelated files.
- Expanding the patch to fix unrelated failures.

## Validation And Testing

- For a bug fix, prefer adding a test that reproduces the bug.
- For a new feature, prefer adding a corresponding test.
- Integrate with the project's existing test system when one exists.
- Validate in this order when applicable:
  1. Relevant tests
  2. Lint
  3. Typecheck
  4. Build
- Run formatters only on modified files by default.
- Determine whether a failing check is related to the patch.
- Investigate related failures and fix them within scope.
- Record and report clearly unrelated failures without expanding the patch to fix them.
- Never claim a command passed unless it was actually run and passed.

## Git Workflow

- Default to creating a feature branch from the repository's primary development branch.
- Do not assume the primary development branch is always named `main`.
- Detect the primary development branch from repository metadata, remote HEAD, project documentation, or existing branch conventions when possible.
- Do not commit directly to protected primary branches by default.
- Do not execute `git commit` without permission.
- Suggest clear, concise commit messages. Conventional Commits are acceptable but optional.
- Show the intended diff or commit scope before requesting commit permission.
- Respect repository-specific merge rules.
- If no merge rule exists, prefer squash merge or a normal merge; avoid complex rebases that risk damaging history.
- Never force push to `main` or `master`.

## Open-Source Contributions

- Follow the upstream project's maintainer style and documented rules over personal preferences.
- Keep patches as small, clear, and issue-focused as possible.
- Do not change project architecture merely to match personal preferences.
- Respect the project's license, CLA, DCO, contribution, and commit requirements.
- Do not modify copyright statements without permission.
- Do not include unrelated changes in an upstream contribution.

## Documentation And Comments

- Write comments to explain why a decision exists, not to narrate obvious behavior.
- Add comments for complex logic, unusual compatibility handling, security decisions, and non-obvious design choices.
- Avoid comments that repeat code behavior or annotate every line.
- Update documentation when changing a public API or user-visible behavior.
- Internal implementation changes usually do not require documentation updates.
- Follow the project's existing documentation language: use Chinese in Chinese projects and English in English projects.

## Code Review

Review for real issues rather than producing a quota of comments. Order findings by:

- `P0`: Security
- `P1`: Correctness / Bug
- `P2`: Regression Risk
- `P3`: Performance
- `P4`: Maintainability
- `P5`: Style

For each finding, include when available:

- Problem description
- Impact scope
- Trigger condition
- Concrete fix suggestion
- File path and line number
- Minimal relevant code snippet

Lead with findings. If no actionable issue is found, state that clearly and mention remaining test gaps or residual risks.

## Completion Report

At the end of a task, report:

- What changed
- What validation ran and its results
- Potential risks
- Unfinished items or blockers

Separate confirmed facts, inferences, and pending confirmations whenever that distinction materially affects the result.
