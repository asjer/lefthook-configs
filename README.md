# lefthook-configs

Personal [Lefthook](https://github.com/evilmartians/lefthook) configuration files for git hook automation.

## Available Configurations

### lefthook-rails.yml

A comprehensive configuration for Ruby on Rails projects with frontend assets.

## Git Hook Environment

Git exports repository-local `GIT_*` variables while hooks run. Jobs that may
shell out to Git in another checkout or cache (for example `bundler-audit
--update`, or test suites that create temporary repositories) clear those
variables first with `unset $(git rev-parse --local-env-vars 2>/dev/null)`.
This prevents nested Git commands from accidentally operating on the hooked
repository.

### lefthook-rails-ci.yml

For Rails apps whose checks live in `config/ci.rb` (Rails 8.1+ `bin/ci`). `config/ci.rb` is the single list of checks; the hook and GitHub Actions both run `bin/ci`.

| Hook | Job | Description |
|------|-----|-------------|
| **pre-commit** | **erblint**, **rubocop** | Same staged-file checks as `lefthook-rails.yml` (in parallel) |
| **pre-push** | **ci** | Runs `bin/ci` with live output (`follow: true`); fails the push when any step fails |

It has no deploy hooks; use `lefthook-rails.yml` for the Hatchbox `deploy`/`deploy-commit` hooks.

## Hooks (lefthook-rails.yml)

### pre-commit

Runs automatically before each commit (in parallel):

| Job | Tags | Description |
|-----|------|-------------|
| **erblint** | `frontend` | Lints ERB templates using erb_lint |
| **rubocop** | `backend` | Ruby code style and quality checks |

### pre-push

Runs automatically before pushing to remote:

| Job | Tags | Description |
|-----|------|-------------|
| **packages audit** | `frontend`, `security` | Audits npm packages for vulnerabilities |
| **gems audit** | `backend`, `security` | Audits Ruby gems using bundle-audit |
| **brakeman** | `security` | Static security analysis for Rails apps |
| **playwright** | `backend` | Runs Playwright end-to-end tests |
| **rspec** | `backend` | Runs RSpec test suite |

### deploy

Custom hook for deployment workflow:

| Job | Description |
|-----|-------------|
| **trigger hatchbox deploy** | Merges main → production and triggers Hatchbox deployment via webhook |

### deploy-commit

Custom hook for deploying a specific commit:

| Job | Description |
|-----|-------------|
| **deploy specific commit to hatchbox** | Deploys a specific commit SHA directly to Hatchbox with validation and confirmation |

**Usage:**
```bash
lefthook run deploy-commit -- abc123def
```

This hook will:
1. Validate the commit SHA exists
2. Display commit details
3. Ask for confirmation
4. Trigger Hatchbox deployment for that specific commit

Useful for hotfixes, rollbacks, or deploying commits from feature branches.

## Usage

1. Install Lefthook in your project
2. Reference this config in your `lefthook.yml`:

```yaml
extends:
  - https://raw.githubusercontent.com/asjer/lefthook-configs/main/lefthook-rails.yml
```

Or copy the configuration file directly into your project.

## Running Specific Tags

```bash
# Run only security-related checks
lefthook run pre-push --tags security

# Run only backend checks
lefthook run pre-push --tags backend
```

## License

See [LICENSE](LICENSE) for details.
