# Repository Guidelines

## Project Structure & Module Organization
Each role lives at the repository root (`caddy/`, `docker_compose/`) and follows the usual layout: `tasks/main.yml` for orchestration, `defaults/main.yml` for overridable variables, `templates/` for Jinja sources, `meta/argument_specs.yml` for input contracts, and `meta/main.yml` with the Galaxy metadata required for installs. Mirror that layout when adding new roles and document required vars plus usage in the role `README.md`.

## Build, Test, and Development Commands
The bundled `ansible.cfg` points at the INI inventory `inventory/default`, so run commands from the repo root without extra flags:
- `ansible-playbook tests/caddy_role.yml --syntax-check` sanity-checks the Caddy role.
- `ansible-playbook tests/docker_compose_role.yml --syntax-check` does the same for the Docker Compose role.
- `ansible-lint caddy docker_compose` enforces Ansible best practices.
- `yamllint .` catches structural YAML issues early.

## Coding Style & Naming Conventions
Use two-space indentation in YAML and avoid tabs. Variables stay lowercase snake_case, prefixed with the role name when shared (`caddy_image`). Task names are imperative (“Deploy docker compose bundle”). Templates should rely on explicit filters for clarity. For Ansible changes, run `ansible-lint` before opening a pull request to confirm metadata and style remain consistent.

## Testing Guidelines
Sample inventory lives in `inventory/default` and targets `localhost`. Role-specific playbooks in `tests/caddy_role.yml` and `tests/docker_compose_role.yml` hardcode the vars they need and use fixtures under `tests/fixtures/`. Copy that pattern when adding roles. Run syntax checks and lint for affected Ansible files. `ansible-playbook … --check --diff` previews supported changes; it does not prove idempotence. Verify idempotence with two real runs in an isolated test environment and confirm the second reports no unexpected changes. Add Molecule scenarios under `molecule/<role>/default/` when broader role coverage is needed, and report what was exercised. Instruction-only edits need generated-file and diff checks, without running playbooks.

## Commit & Pull Request Guidelines
Keep commit subjects short and imperative (e.g. `Add docker compose fixture`). PRs should identify affected roles, validation results, fixture changes, and follow-up work. Follow the shared delivery workflow below.

## Security & Configuration Tips
Do not commit secrets; depend on vaulted files or environment overrides instead. Review exposed ports and volume mounts in `templates/` when touching Docker assets. Document required environment variables or external services in each role’s README so operators can reproduce the configuration safely.

## Pull Request Delivery Workflow

Deliver repository changes through pull requests by default, regardless of
size. Do not make changes directly in the main checkout unless the user
explicitly approves an exception. Direct commits to `main` or the default
branch should be limited to explicit user-approved exceptions.

Work on a dedicated topic branch, using a separate worktree when required or
useful. Make the requested change, run relevant validation, and pass the review
gate below before committing or creating/updating a PR. Keep saved-plan
progress current and close the plan when its objective is complete. PRs should
describe the final scope and validation results.

When asked to prepare changes as PRs for review, finish with validated,
reviewed PRs and report remaining limitations. A read-only review ends with
findings and coverage limits; it does not authorize changes or PR creation.
For authorized delivery, continue through green checks,
merge, any explicitly authorized deployment, and verified cleanup. Use a
Conventional Commit PR title and squash subject when the repository uses them
to determine release versions.

Treat a request to `deploy`, `ship`, `publish`, or `deliver` the current
requested repository change set as authorization to complete this normal
topic-branch workflow: commit reviewed in-scope changes, push the topic branch,
create or update its pull request, monitor required checks, make narrowly scoped
fixes for failures caused by the change, merge when all gates pass, and remove
the clean merged worktree and merged topic branches under the cleanup checks
below. Apply required validation and review to every fix. Do not ask for
separate approval for each ordinary step.

This authorization applies only to the current requested repository change
set. It does not authorize force pushes; bypassing reviews, checks, or branch
protections; direct-default-branch commits; manual releases or package
publication outside the repository's existing merge-triggered automation;
access to or disclosure of secrets; destructive repository operations;
unrelated pull requests; or material scope expansion. Authorization to merge a
pull request includes any package version and publication performed
automatically by the repository's existing merge workflow. In this section,
`deploy` authorizes repository delivery; it authorizes a service or
infrastructure deployment only when the current request specifically identifies
that deployment. More-specific repository approval rules, including final
content or product publication, still apply. Cleanup is limited to the verified merged worktree and topic
branches described below; it never includes dirty worktrees or forced remote operations.

When requesting platform approval for an authorized step, quote the user's
delivery request and this shared instruction in the justification. If a
platform reviewer rejects the action, ask the user once and wait. Do not retry
an equivalent escalation or repeat the prompt during automatic continuations
unless the user provides new authorization or relevant context.

Before committing, run `git status --short`, stage intended files by exact
path, and verify the staged scope. For an explicitly approved default-branch
exception, state that the normal PR workflow is being bypassed and still check
scope. Include screenshots only for changes to rendered UI, generated visual
output, or external presentation.

## Merged-Branch Cleanup

After confirming the exact PR is merged, remove only its clean worktree.
Ordinary remote branch deletion requires the remote ref to match the PR's
recorded head. A local topic branch may be deleted with `git branch -D` only
when its tip matches that recorded head and either:

- Its tree matches the squash commit's tree; or
- When the base advanced, both the `git patch-id --verbatim` of the aggregate
  diff from the merge base matches the squash commit's first-parent diff and
  applying that exact aggregate diff to the first-parent tree produces the
  squash commit's tree.

The second proof handles intervening base changes without ignoring whitespace
or patch locations. Retain the branch if neither proof succeeds. This is not
authorization for `git branch -D` on any other local branch or for other
destructive operations.

## Review Gate

Before committing, use the installed `$branch-review-subagent-loop` skill to
review the complete branch diff. Follow the skill through any required fixes,
validation, and re-review. If the skill is unavailable, ask the user to install
it before continuing.

Create, update, or merge the pull request only after the review gate passes.
Merging also requires green checks unless the user explicitly accepts the
remaining risk.
