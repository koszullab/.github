# Contributing

Thank you for helping. Bug reports, fixes, features and corrections to the documentation are all
welcome. A repository's own contributing page (`docs/contributing.md`, on its documentation site) says
how to set it up, test it and document it; what follows holds for every repository.

## Reporting a problem

Open an issue in the repository concerned, with the *Bug report* form. Report a security problem
privately, as the [security policy](https://github.com/koszullab/.github/blob/main/SECURITY.md) says.

## Proposing a change

- **Open an issue first** for anything larger than a fix, so that the approach is agreed before you
  write it.
- **One pull request, one concern.** It is reviewed as a whole, tested at its head, and
  squash-merged.
- **Every commit is a [Conventional Commit](https://www.conventionalcommits.org/en/v1.0.0/):**
  `type(scope): summary`, and a body that says why. The types are `feat`, `fix`, `perf`, `docs`,
  `test`, `refactor`, `build`, `ci` and `chore`; a breaking change adds `!` and a `BREAKING CHANGE:`
  footer. A pull request that is one self-contained change has a Conventional Commit as its title too.
- **A change a user can notice** gets a line under *Unreleased* in `docs/changelog.md`, and changes
  the page of the documentation that tells a user about it (`docs/cli.md`, for the command line).
- **Tests** cover at least 80 % of the lines a pull request changes, and keep the package at 80 % or
  more. Codecov checks both on every pull request.
- **Nothing a user calls is renamed:** commands, options, output files, modules. Add an alias
  instead.

The conventions every repository follows, and why: [CONVENTIONS.md](https://github.com/koszullab/.github/blob/main/CONVENTIONS.md).

## Licence

Each repository has its own licence, in its `LICENSE` file. By contributing, you agree that your
contribution is released under it.

## Conduct

Everyone taking part follows the [code of conduct](https://github.com/koszullab/.github/blob/main/CODE_OF_CONDUCT.md).
