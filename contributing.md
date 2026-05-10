# Contributing

Contributions to Sloth are welcome. This page covers the essentials — setup, workflow and conventions.

## Setup

```bash
git clone https://github.com/folivoro/sloth.git
cd sloth
composer install
```

Work on the `develop` branch. Create feature branches from there:

```bash
git checkout develop
git checkout -b feature/my-feature
```

## Workflow

Before committing, run:

```bash
composer cs-fix    # fix code style
composer analyse   # PHPStan static analysis
composer test      # Pest test suite
```

All three must pass. PRs with failing tests or PHPStan errors won't be merged.

## Commit messages

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(routing): add support for route groups
fix(model): resolve issue with Post::find
docs(readme): add installation instructions
test(cache): add tests for wp-transients driver
refactor(console): simplify command discovery
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

## Adding features

- New classes go in the appropriate `src/` directory
- Every class needs PHPDoc with `@since 1.0.0`
- New features need tests in `tests/Unit/`
- Auto-discovered classes (Models, Modules, Providers etc.) need no registration — drop them in the right directory
- If adding a config option, add `mergeConfigFrom` and `publishes` to the ServiceProvider

## Pull requests

- Target `develop`, not `main`
- Include a clear description of what the PR does and why
- Link to related issues
- Keep PRs focused — one thing per PR

## Reporting issues

Open an issue on GitHub with:
- PHP and WordPress version
- Steps to reproduce
- Expected vs actual behavior
- Relevant error output

## License

By contributing, you agree your contributions will be licensed under the MIT License.
