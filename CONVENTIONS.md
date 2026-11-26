# Repository conventions

These conventions apply to every repository of the project: **chromosight, hicstuff, instaGRAAL, metaTOR**
(koszullab) and **metacooler, chromosomix, tinymapper** (js2264), and to any repository added later. Sections 1–22
are written for the Python packages; § 23 says what applies to chromosomix, a TypeScript application.
`MIGRATION.md` lists what each repository still has to change to follow them.

Where a repository already does something the way it should be done, it is named as the **reference**:
its file is the one to copy.

## How to apply them

- **New things follow the conventions in full; existing public names stay.** A convention never
  justifies renaming a command, an option, a module, an output file, a documentation URL, a test-data
  path, a branch or a published container tag: other people's scripts, pipelines and links use them
  (§ 18). Add an alias instead.
- **A deviation is allowed when its reason is written next to it**, as hicstuff's `pybwa<2.4` pin
  explains itself in a comment.
- **Whatever can be checked is checked in CI**, not left to review.

## 0. Decisions

| Topic | Decision |
|---|---|
| Build backend | hatchling ≥ 1.27 |
| Environment and lock | uv; `uv.lock` committed |
| Version | written once, as `version` in `pyproject.toml`; bumped by bump-my-version |
| Python versions | 3.10 and newer, for as long as the dependencies allow |
| Formatting and linting | ruff |
| Type checking | ty |
| Tests | pytest; Codecov: at least 80 % covered overall, and 80 % of the lines a pull request changes |
| Documentation | zensical with chromokit, on GitHub Pages |
| Changelog | `docs/changelog.md`: `## Unreleased`, then `## [X.Y.Z] - YYYY-MM-DD` |
| Commits | Conventional Commits, always; pull requests squash-merged |
| Releases | tag `vX.Y.Z`; PyPI trusted publishing; GitHub release from the changelog |
| Conda | bioconda |
| Containers | only where pip cannot install everything; tags `X.Y.Z` and `latest` |
| Logs | a pipeline writes `<sample>.<tool>_<YYYYMMDDHHMMSS>.log` beside its outputs, with hicstuff's header; a single command writes no log file, only stderr |
| Reports | a `report` command in every tool, on metaTOR's model: `report.txt`, a MultiQC row and a page, from chromokit's report kit |
| Agent instructions | `AGENTS.md`, and nothing else |
| Shared files | the community files in the `.github` repositories; everything else in chromokit: the docs' look, the report kit, the copier template |

To change a decision, change it here first, then in every repository.

## 1. Layout

```text
<repo>/
├── src/<package>/                 the import package (src layout)
│   ├── __init__.py                exposes __version__, read from the installed metadata
│   ├── cli.py or cli/             the command line
│   ├── _report/                   chromokit's report kit (§ 12, § 21), never edited
│   └── py.typed                   once the public API is annotated
├── tests/                         pytest
│   └── data/                      small test data
├── docs/                          the documentation site's pages
├── benchmarks/                    optional (§ 20)
├── .github/
│   ├── workflows/                 ci.yml, changelog.yml, docs.yml, release.yml,
│   │                              dependabot-automerge.yml,
│   │                              retry-jobs-without-runner.yml (macOS runners only)
│   ├── dependabot.yml
│   └── codecov.yml
├── pyproject.toml   uv.lock   .python-version
├── zensical.toml   chromokit.json   overrides/
├── .copier-answers.yml            the answers given to chromokit's template (§ 21)
├── .pre-commit-config.yaml   .editorconfig   .gitattributes   .gitignore   .git-blame-ignore-revs
├── Dockerfile   .dockerignore     only under § 15
├── README.md   LICENSE   CITATION.cff   CONTRIBUTING.md
└── AGENTS.md                      the only instructions for coding agents (§ 21)
```

- No `setup.py`, `setup.cfg`, `MANIFEST.in` or `requirements*.txt`: `pyproject.toml` and `uv.lock` say
  everything. A conda environment file exists only if the Dockerfile uses it (metaTOR's `metator.yaml`), or if
  the runs need programs pip cannot install, as tinymapper's do: `env/tinymapper.yaml`, locked in
  `env/conda-lock.yml` and locked again at every release.
- The code of conduct, security policy, support page, issue forms and pull-request template come from the
  owner's `.github` repository (§ 21). A repository keeps its own only when it needs different ones.
- Existing repositories keep the names of their test-data directories (`test_data/`, `tests_data/`,
  `data_test/`): CI configurations in other projects may fetch from those paths.

## 2. Packaging

References: metacooler's `pyproject.toml` for the build, the sdist and the dependency groups;
instaGRAAL's for the version and the `gpu` extra; metaTOR's for the pytest block (§ 13).

```toml
[build-system]
requires = ["hatchling>=1.27"]        # 1.27 writes PEP 639 license metadata
build-backend = "hatchling.build"

[project]
name = "<distribution>"
version = "X.Y.Z"                     # the single source of the version (§ 14)
description = "<one sentence>"
readme = "README.md"
requires-python = ">=3.10"            # § 3
license = "<SPDX expression>"         # and no "License ::" classifier beside it
license-files = ["LICENSE"]
authors = [{ name = "…", email = "…" }]
maintainers = [{ name = "…", email = "…" }]
keywords = ["hi-c", "…"]
classifiers = [
    "Development Status :: …",
    "Intended Audience :: Science/Research",
    "Programming Language :: Python :: 3 :: Only",
    "Programming Language :: Python :: 3.10",      # one line per supported version
    "Topic :: Scientific/Engineering :: Bio-Informatics",
    "Operating System :: POSIX :: Linux",
    "Operating System :: MacOS",
]
dependencies = [
    "numpy>=1.24",                    # every floor: § 3
]

[project.optional-dependencies]       # optional runtime features only, never tools
# gpu = ["pycuda"]                    # as instaGRAAL's: only the MCMC scaffolder needs it

[dependency-groups]                   # never published
dev = ["bump-my-version", "diff-cover", "pre-commit", "pytest", "pytest-cov", "ruff", "ty"]
docs = ["zensical", "mkdocstrings[python]"]

[project.urls]
Homepage = "https://github.com/<owner>/<repo>"
Documentation = "https://<owner>.github.io/<repo>/"
Repository = "https://github.com/<owner>/<repo>"
Changelog = "https://<owner>.github.io/<repo>/changelog/"
Issues = "https://github.com/<owner>/<repo>/issues"

[project.scripts]
<command> = "<package>.cli:cli"

[tool.hatch.build.targets.wheel]
packages = ["src/<package>"]

[tool.hatch.build.targets.sdist]
exclude = ["/.github", "/uv.lock", "/.pre-commit-config.yaml", "/Makefile",
           "/AGENTS.md", "/benchmarks", "/docs/images"]
```

- **The version is written once**, as `version` in `[project]`, as hicstuff and instaGRAAL do.
  `__init__.py` exposes it: `__version__ = importlib.metadata.version("<distribution>")`.
  - metaTOR and metacooler write it in a module instead (`version.py`, `_version.py`), which hatch reads
    (`dynamic = ["version"]`, `[tool.hatch.version]`). That works as well, and they keep it. Their
    `[tool.uv] cache-keys` names the module, so that uv notices a bump.
  - Version modules that exist stay importable: `metator.version`, `instagraal.version`,
    `chromosight.version`.
- **Extras are for optional features users install**, never for development tools. An example is
  instaGRAAL's `gpu`, which adds pycuda for the MCMC scaffolder. Tools go in dependency groups, which uv
  installs and PyPI never shows.
- **Every runtime dependency has a lower bound, and every bound has a reason** (§ 3).
- **The sdist carries what it takes to build and test the package:** the sources, the tests and their
  data, README, LICENSE, CITATION.cff and the changelog. Packagers run the tests from it, and
  chromosomix's CI builds its fixtures from metacooler's. Repository tooling and media stay out. A
  repository whose test data is too large to ship says so in a comment (metaTOR leaves out its 28 MB
  `tests_data/`).
- **The wheel carries the package and nothing else.** Data files the package reads at run time sit
  inside `src/<package>/`.
- **The command is named after the distribution.** Further commands share its prefix
  (`instagraal-pre`).

## 3. Supported versions

- **Python: 3.10 and newer, for as long as the dependencies allow.** Python 3.10 reached its end of life
  in October 2026, and stays supported regardless.
  - A version is dropped only once a dependency the package needs has dropped it, and then in a
    **minor** release. pip and uv go on installing the last release that supported it, so nothing
    breaks; that Python just stops receiving new releases.
  - On 3.10, users already get numpy 2.2, scipy 1.15 and pandas 2 at the newest, since numpy 2.3,
    scipy 1.16 and pandas 3 require 3.11. The `bounds` job tests the oldest versions on 3.10 (§ 6).
  - A package that raises its floor on another package of the project to a release that has already
    dropped a Python version drops that version in the same release.
- **The next Python is tested as soon as it can be installed**, in a job allowed to fail (§ 6).
- **No upper bound on Python** unless a dependency forces one. The bound carries a comment naming the
  dependency and the issue that tracks lifting it.
- **Dependencies:**
  - each lower bound is the oldest version the `bounds` job (§ 6) passes with;
  - a bound is raised only with its reason, in the `pyproject.toml` comment and in the changelog;
  - upper bounds only for a known incompatibility, with a comment and an issue;
  - no `==` pins.
- **Platforms:** Linux (x86-64 and arm64) and macOS (arm64 and x86-64). Windows is not supported unless
  a repository says otherwise.

## 4. Development environment

- **uv.** `uv sync` makes the environment: the package in editable mode, and the `dev` group. The
  `docs` group is used on its own: `uv run --only-group docs …` installs neither the package nor its
  dependencies, since mkdocstrings reads the sources without importing them.
- **`uv.lock` is committed and checked:** `uv lock --check` and `--locked` in CI. Dependabot refreshes
  it (§ 7).
- **`.python-version` is committed.** It names the Python used for development, normally the newest one
  every dependency publishes wheels for.
- **The same commands everywhere**, listed in `AGENTS.md` and in `docs/contributing.md`:

  ```bash
  uv sync                                         # the environment
  uv run pytest                                   # the tests, with coverage
  uv run ruff check . && uv run ruff format .     # lint and format
  uv run ty check src                             # types
  uv run pre-commit install                       # the hooks
  uv run --only-group docs zensical serve         # the documentation, at http://localhost:8000
  uv run bump-my-version bump minor               # a release (§ 14)
  ```

- **A Makefile is optional**, for the repository's own chores (fixtures, a rendered README), never for
  the commands above.

## 5. Code quality

- **Format with ruff.**
  - The line length is each repository's choice; new repositories use ruff's default of 88. Changing an
    existing repository's line length reformats everything for no gain.
  - A mass reformatting commit goes in `.git-blame-ignore-revs`.
- **Lint with ruff**, from an explicit rule list so that upgrading ruff never changes the rules:

  ```toml
  [tool.ruff.lint]
  select = ["E", "W", "F", "I", "B", "UP", "SIM", "RUF"]
  ignore = ["E501"]   # the formatter wraps code; long strings and docstrings are allowed
  ```

  Every other entry in `ignore` has a comment saying why.
- **Type-check with ty**, in CI. ty is advisory (`continue-on-error`) until a repository's existing
  diagnostics are fixed, then required.
- **Docstrings:** numpy style in new repositories. Each repository states its style once, in
  mkdocstrings' `docstring_style` and in ruff's `[tool.ruff.lint.pydocstyle] convention`, and new code
  follows it.
- **Logging:** `logging.getLogger("<package>")`, never `print` for diagnostics. Only the command line
  configures handlers.
- **pre-commit.** Reference: metacooler's list.

  ```yaml
  repos:
    - repo: https://github.com/pre-commit/pre-commit-hooks
      rev: v6.0.0
      hooks:
        - id: trailing-whitespace
        - id: end-of-file-fixer
        - id: check-yaml
        - id: check-toml
        - id: check-json
        - id: check-merge-conflict
        - id: debug-statements
        - id: check-added-large-files
          args: [--maxkb=5000]
    - repo: https://github.com/astral-sh/ruff-pre-commit
      rev: vX.Y.Z          # the ruff of uv.lock: CI fails when they differ
      hooks:
        - id: ruff-check
          args: [--fix]
        - id: ruff-format
  ```

- **`.editorconfig`:**
  - UTF-8, LF line endings, a final newline, no trailing whitespace;
  - indentation of 4 spaces for Python, 2 for YAML, TOML and JSON.
- **`.gitattributes`:**
  - `* text=auto eol=lf`;
  - binary test data (`*.cool`, `*.mcool`, `*.metacool`, `*.h5`, `*.gz`, `*.bt2`) marked `binary`;
  - generated files marked `linguist-generated`.

## 6. Continuous integration

References: metaTOR's `ci.yml` (lint, test), metacooler's `dependency-bounds` job, and hicstuff's
`pip-only` and `wheels` jobs and `retry-jobs-without-runner.yml`.

`ci.yml` runs on a push to the default branch, on every pull request, by hand, every week, and from
`release.yml` (`workflow_call`).

| Job | What it does |
|---|---|
| `lint` | `uv lock --check`; `uv run --locked ruff check .`; `uv run --locked ruff format --check .`; ty (§ 5); checks that pre-commit's ruff `rev` is the locked ruff; zizmor on the workflows |
| `test` | `uv sync --locked`; `<command> --version`; pytest. Every supported Python on `ubuntu-latest`, and the newest on `macos-latest`. The next Python is allowed to fail (`continue-on-error`), building from source what has no wheel yet. Coverage is uploaded from one leg only: the newest Python, on Linux |
| `bounds` | Ignores `uv.lock` and re-resolves twice: `UV_RESOLUTION=lowest-direct` on the oldest supported Python, then `highest` on the newest. Then `uv sync` and `pytest --no-cov`. Every uv command of the job sees `UV_RESOLUTION` (metacooler's comment explains why) |
| `install` | Builds the wheel and installs it in a fresh environment, as a user would, then runs `<command> --version`, `--help` and a short real run. A package that needs nothing outside PyPI runs it with an empty `PATH`. Runs on Linux (x86-64, arm64) and macOS (arm64, x86-64), on the oldest and newest Python |
| `wheels` | Only for packages with compiled dependencies. On each platform and Python, checks that every version pip would pick has a wheel |

`changelog.yml` (§ 9) runs on pull requests, including when a label changes. `docs.yml` is described in
§ 8, `release.yml` in § 14.

**Tests that need special hardware** sit in their own module, skip themselves without it, and run only on
matching runners. Reference: instaGRAAL's GPU tests, `tests/test_instagraal_gpu.py`, which
`test-gpu.yml` runs on RunPod pods, with four safeguards against a pod outliving its run.

Every workflow follows these rules:
- **Permissions:** `permissions: contents: read` (or `{}`) at the top; a job asks for more only when it
  needs it.
- **Concurrency:** `${{ github.workflow }}-${{ github.ref }}`, cancelling superseded CI and docs builds,
  and never a deployment or a release.
- **Timeouts:** `timeout-minutes` on every job.
- **Triggers:** pushes filtered to the default branch. Pull requests test their own branches, and an
  unfiltered `push` runs everything twice.
- **Default branch:** read from `github.event.repository.default_branch`, never written out. It is
  `master`, `devel` or `main`, depending on the repository.
- **Untrusted input:**
  - no `pull_request_target`;
  - values from a pull request (its title, its branch name) reach a `run:` step through `env:`, never
    through `${{ }}` inside the script.
- **Pinning:** `actions/*` pinned to a major version, third-party actions to a commit SHA with the
  version in a comment. Dependabot updates both.
- **macOS:** a repository that uses macOS runners also has `retry-jobs-without-runner.yml`.

## 7. Dependency automation

References: hicstuff's and metaTOR's `.github/dependabot.yml` and `dependabot-automerge.yml`, which are
identical.

- **`uv`, weekly.** Minor and patch updates are grouped into one pull request, which merges itself once
  the required checks pass. Major updates come one per pull request, for review.
- **`github-actions`, weekly**, grouped into one pull request.
- **`docker`, weekly**, where a Dockerfile exists.
- **Dependabot alerts and security updates** are on.
- **The weekly `bounds` run is the early warning** for upstream releases that land between two lock
  updates.

## 8. Documentation

References: metaTOR's and instaGRAAL's `docs/` and `zensical.toml`; instaGRAAL's `docs.yml`.

- **Engine: zensical, with chromokit's look.**
  - The kit's files are listed in `chromokit.json` and never edited.
  - A repository's own styles go in `docs/stylesheets/site.css`.
  - `npx github:js2264/chromokit sync` updates the kit.
- **Pages,** in this order (`nav` in `zensical.toml`):

  | Section | Pages |
  |---|---|
  | Home | `index.md` |
  | Getting started | `why.md`, `installation.md`, `quickstart.md`, `concepts.md` |
  | Guides | `guides/*.md`, one per task |
  | Reference | `cli.md`, `formats.md`, `api/*.md` |
  | The project | `contributing.md`, `citation.md`, a page on upgrading (§ 9), `changelog.md`, and optionally `benchmarks/` |

  - A repository adds the pages it needs: metaTOR's and instaGRAAL's `end-to-end.md`, instaGRAAL's
    `showcase.md` and `troubleshooting.md`.
  - An existing page keeps its file name, and so its URL: metacooler's `format.md` stays as it is.
- **The Python API pages** are mkdocstrings directives (`::: package.module`), with the options
  metaTOR's `zensical.toml` sets.
- **The CLI reference, `docs/cli.md`, is written by hand.**
  - It has one `` ### `command` { #command } `` section per command, with a table whose first column
    lists each option.
  - `tests/test_docs.py` fails when the page misses a command or an option, or documents one that does
    not exist. hicstuff's and metaTOR's copies of that test are the same code.
- **The report (§ 12) is documented on `formats.md`:** every number it holds, its unit and its MultiQC
  column, as metaTOR's section "The report of a run" does.
- **Diagrams** are Mermaid code blocks; **images** go in `docs/images/`.
- **Build:** `uv run --locked --only-group docs zensical build --clean --strict` on every pull request.
  - It installs the docs tools alone, as instaGRAAL's `docs.yml` does: mkdocstrings reads `src/` without
    importing it, so neither the package nor its dependencies are needed.
  - The same job runs `npx github:js2264/chromokit check`.
- **Deploy:** to GitHub Pages, from the default branch only. The `github-pages` environment refuses tags,
  so a release reaches the site through its release commit.
- **Version shown:** the site shows `extra.release` from `zensical.toml`, which bump-my-version updates.
- **The README is short:**
  - what the package does;
  - badges, in this order: PyPI, Python versions, bioconda, CI, Docs, Codecov, DOI, License, Ruff;
  - one install line per channel;
  - a quick start;
  - links to the site and to the citation.

## 9. Changelog

- **`docs/changelog.md`**, so that the site carries it. GitHub release notes are copied from it.
- **Headings:** `## Unreleased`, as hicstuff, metaTOR, metacooler and instaGRAAL write it. Then one
  `## [X.Y.Z] - YYYY-MM-DD` per release, in Keep a Changelog 1.1's format. chromosomix keeps Keep a
  Changelog's `## [Unreleased]`.
- **Sections**, in this order, keeping only those that apply:
  - Breaking changes
  - Added
  - Changed
  - Deprecated
  - Removed
  - Fixed
  - Performance
  - Security
- **Every change a user can notice gets an entry, in the pull request that makes it.** `changelog.yml`
  fails a pull request that changes `src/` without changing the changelog, unless it carries the
  `no-changelog` label.
- **Each entry ends with a link to its pull request**, as instaGRAAL's recent entries do.
- **Every release with breaking changes comes with a page on upgrading.** metaTOR's is `migration.md`,
  for 2.0; instaGRAAL's is `upgrading.md`.

## 10. Command lines

These apply to new command lines, commands and options. Existing ones keep their names, and may gain
aliases. Existing parsers are not rewritten, since a new parser changes which command lines are
accepted: chromosight keeps docopt, instaGRAAL keeps click.

- **Framework:** rich-click.
  - Every command takes `-h/--help`; the root command takes `--version`.
  - Each command's `help=` is one line, and the details go in its options' help.
  - Commands are grouped into panels.
- **Option names:**

  | Purpose | Option | Existing exceptions, kept |
  |---|---|---|
  | One output file | `-o/--output` | |
  | A directory of outputs | `-o/--outdir` | instaGRAAL's `--output-dir`; tinymapper's `-o/--output` |
  | Temporary files | `--tmpdir` | metacooler's `--temp-dir`, cooler's name |
  | Parallelism | `-t/--threads` | metacooler's `--nproc`, cooler's name |
  | Overwriting outputs | `--force` | |
  | The run's name: the sample | `-p/--prefix` | chromosight's positional `<prefix>`; tinymapper's `-s/--sample`, a path prefix whose base name is the sample |
  | Debug logging | `-v/--verbose` | tinymapper's `-v`, which is `--version` |
  | Warnings only | `-q/--quiet` | |

- **Outputs:** an existing output is never overwritten without `--force`.
- **Option names are lowercase, with words joined by hyphens.** tinymapper's `--keepIntermediate` and
  `--MNaseSizes` stay, and may gain hyphenated aliases.
- **Several values:** an option that takes several values accepts both repetition and a comma-separated
  list.
- **Exit codes:** 0 for success, 1 for an error, 2 for a usage error (click's).
- **The sample's name** is the run's prefix, as hicstuff and metaTOR take it. A command without one takes
  it from its main output's file name, without the extensions: metaTOR's report takes the map's. The
  sample's name names the log (§ 11) and the report (§ 12).
- **Logs** go to stderr. A pipeline also writes the log file of § 11; a single command does not.
- **A `report` command** in every tool writes the report of a run (§ 12).

## 11. Logs

References: hicstuff's and metaTOR's logs; see `generate_log_header`, `log_end` and `log_file_name` in
metaTOR's `cli.py`.

- **Only a pipeline writes a log file.** A pipeline is a command that runs several steps, start to end, as
  one run: a workflow, an end-to-end run. They are `hicstuff pipeline`, `metator pipeline`,
  `instagraal-endtoend` and a `tinymapper` run, the commands that write their report at their end
  (§ 12). It writes its log beside its outputs, as `<sample>.<tool>_<YYYYMMDDHHMMSS>.log`.
  - `<sample>` is the run's name (§ 10), and the time is when the pipeline started, in local time.
  - Without a sample name, the log is `<tool>_<YYYYMMDDHHMMSS>.log`.
  - hicstuff and metaTOR already name their logs this way.
  - A tool that keeps its logs in a `logs/` folder of its output directory keeps them there, as
    tinymapper does.
  - The pipeline's steps log into its file. A step writes no file of its own.
- **A single command writes no log file,** whatever it writes (`digest`, `filter`, `rebin`, `detect`,
  `annotate`, …), and a command that only reads (`info`, `ls`, a dump to stdout) neither. They log to
  stderr only. A log file is the record of a whole run. One beside every output that users chain by hand
  would scatter files through their directories, and every log file's name and header is interface that
  then has to be kept (§ 18).
  - **A tool with no pipeline writes no log file at all:** metacooler and chromosight. That is the
    rule, not an exception to it.
  - **A tool that already writes a log from a single command keeps it, and adds none:** metaTOR's
    `network`, `partition`, … Dropping one is a breaking change (§ 18).
- **The command line attaches the file handler** when the pipeline starts, in its output directory, never
  when a module is imported. Today, instaGRAAL opens `instagraal-<date>.log` in the working directory
  on import, even for `--help`, so every one of its commands writes it.
- **The header is hicstuff's**, which metaTOR copies: the version, the date, the command line, one line
  per parameter (as given, or by default), then `---`:

  ```text
  ## hicstuff: v4.0.0 log file
  ## date: 2026-11-26 10:15:30
  ## cmd: hicstuff pipeline -g genome.fa -o out -p S1 R1.fq.gz R2.fq.gz
  ## threads: 8
  ## prefix: S1
  ---
  ```

  Then one line per record, `%(asctime)s :: %(levelname)s :: %(message)s` with dates as
  `%Y-%m-%d,%H:%M:%S`. The last line says what the run wrote and how long it took
  (`… after 0h 17m 31s`).
- **Levels:**
  - the file always gets INFO and above;
  - `-v` adds DEBUG, to the file and to stderr;
  - `-q` limits stderr to warnings, never the file.
- **A log is part of the interface** (§ 18). `hicstuff stats` reads it, and a pipeline's report (§ 12)
  takes its *Run* tab from the header. Its file name, the header's keys, and the lines a parser reads
  change only with a changelog entry.

## 12. Reports

Reference: metaTOR's `metator report`, in `src/metator/report.py` and `src/metator/templates/report.html.j2`.
Every tool's report follows it.

Every tool has **one command that writes the report of one run, for one sample**:
- **Name:**
  - `<tool> report` where the command line is a group: `hicstuff report`, `metator report`,
    `metacooler report`, `chromosight report`;
  - `<tool>-report` beside a command line that is a single command, or a set of commands:
    `tinymapper-report`, `instagraal-report`;
  - chromosomix's `mapreport` is its report, and keeps its name.
- **Input:** it reads what the run left, meaning its output directory, or its main output (for metaTOR,
  the map). It never runs the analysis again.
- **Pipelines write it at their end:** `hicstuff pipeline`, `metator pipeline` (as it already does),
  `instagraal-endtoend`, and every `tinymapper` run.
- **Files,** in the output directory, named after the sample, as metaTOR's `report_files` names them:

  | File | What it holds |
  |---|---|
  | `<sample>.report.txt` | every number of the report, for scripts: `[section]` then a key and its value per line; `[table: name]` then a header and a row per line; `NA` when missing. Its `format_version` is in its header |
  | `<sample>.report_mqc.yaml` | MultiQC's custom content: the sample's row of the General Statistics table |
  | `<sample>.report.html` | the page (below) |

- **How it is made,** as in metaTOR:
  - `collect` gathers the run into a report of sections (a key and its value each) and tables;
  - `write_txt` writes `report.txt`, and `read_txt` reads it back;
  - `write_mqc` writes `report_mqc.yaml`;
  - `write_html` writes the page from the Jinja2 template;
  - `write_report` writes the three, the page last, since it links to the other two.
- **The values in `report.txt`** keep metaTOR's rules:
  - an integer as an integer;
  - a `_pct` with 2 decimals, a `_ratio` or `_fraction` with 4;
  - lengths in `_bp`;
  - text on one line.
- **The page** is rendered by Jinja2 from the kit's base template (§ 21). That template is metaTOR's,
  restyled with chromokit's look. Each tool's template extends it with its own sections. The page has:
  - a header with the tool, its version, the sample, the date, and the main input;
  - a navigation bar over the sections: an overview of the key numbers as cards, one section per subject,
    a *Run* section with the command and its parameters, and a *Files* section linking the run's outputs;
  - tables that sort on a click and filter as you type, each with a *Download CSV* button;
  - interactive plots, drawn by plotly.js. The page loads it from its CDN, at a pinned version
    (metaTOR's `PLOTLY_JS`). Offline, each plot says it needs a connection, while the numbers and tables
    stay readable;
  - a footer naming the tool's version and the report's format.
- **Dependencies:** `jinja2>=3.1` for the page, and `pyyaml>=6.0` for the MultiQC file, as metaTOR
  declares them. A package that lacks them adds them to its runtime dependencies and to its bioconda
  recipe.
- **MultiQC,** checked against MultiQC 1.35:
  - **`<sample>.report_mqc.yaml` is the report's only MultiQC file.** It holds one section:
    `plot_type: generalstats`, `id: <tool>` and `section_name`, with one column per number that sums up a
    sample. Each column has its own `pconfig` (title, description, format, minimum and maximum), as
    metaTOR's `MQC_COLUMNS` does. A file that lists several sections stops MultiQC's custom content
    altogether.
  - **`data` is keyed by the sample.** Every sample's file carries the same `id`, and MultiQC merges them
    into one table.
  - **The tools a run calls,** where MultiQC has a module for them (samtools, bowtie2, STAR, MACS,
    deepTools), keep their own logs or statistics under the sample's name, beside the report. They are
    the run's outputs, not the report's files: MultiQC reads them with its own modules. tinymapper's
    runs call these tools.
  - **Ids and columns belong to the report's format.** Changing or removing one raises `format_version`,
    as metaTOR's rule says, and is a breaking change (§ 18).
- **One name for one sample.** The sample's name is the same in the log, the report's files and its
  MultiQC rows. A pipeline that runs hicstuff and then metaTOR on a sample, under one name, gets one row
  in MultiQC's General Statistics, with both tools' columns.
- **Tests:**
  - the test data's report is checked: `report.txt` compared, and `read_txt` reading back what
    `write_txt` wrote;
  - CI runs MultiQC on it (`uvx multiqc`), and fails when its section is missing or a module breaks.
- **Documentation:** `formats.md` documents every key of `report.txt`, its unit and its MultiQC column,
  as metaTOR's section "The report of a run" does.

## 13. Tests and coverage

Reference: metaTOR's pytest configuration.

```toml
[tool.pytest.ini_options]
minversion = "8"
testpaths = ["tests"]
xfail_strict = true
addopts = ["-ra", "--strict-config", "--strict-markers",
           "--cov=<package>", "--cov-report=term-missing", "--cov-report=xml"]
```

- **The command line is tested in-process,** with click's `CliRunner`.
  - A subprocess is only for third-party programs.
  - A test that needs a program that is not installed is skipped.
- **A known bug is an `xfail` test** naming its issue in `reason=`. `xfail_strict` fails the suite once
  the bug is fixed, so the marker leaves with the fix.
- **Test data:**
  - no file over 5 MB (the pre-commit hook enforces it);
  - larger data in a release asset named after its generator's version (metacooler's
    `benchmark-data-v<N>`), fetched and cached by the tests;
  - generated data sits beside the script that generates it.
- **Codecov** (`.github/codecov.yml`), with two required checks, both at 80 %:
  - `project`: the package as a whole is at least 80 % covered (`target: 80%`);
  - `patch`: the lines a pull request changes are at least 80 % covered (`target: 80%`).

  Contributors check both before pushing: `pytest`'s coverage report for the first, metacooler's
  `diff-cover` command for the second.

## 14. Releases

Reference: metaTOR's `release.yml`.

- **Semantic versioning,** over the public interface § 18 defines.
- **Cutting a release,** from an up-to-date default branch with green CI:

  ```bash
  uv run bump-my-version bump minor     # or patch, or major
  git push --follow-tags
  ```

- **bump-my-version**, configured identically everywhere:

  ```toml
  [tool.bumpversion]
  current_version = "X.Y.Z"
  commit = true
  tag = true
  tag_name = "v{new_version}"
  message = "chore(release): v{new_version}"
  allow_dirty = false

  # The package's own entries: the name anchors the search, keeping it off
  # current_version above and off the other packages of uv.lock (hicstuff's).
  [[tool.bumpversion.files]]
  filename = "pyproject.toml"
  search = "name = \"<distribution>\"\nversion = \"{current_version}\""
  replace = "name = \"<distribution>\"\nversion = \"{new_version}\""

  [[tool.bumpversion.files]]
  filename = "uv.lock"
  search = "name = \"<distribution>\"\nversion = \"{current_version}\""
  replace = "name = \"<distribution>\"\nversion = \"{new_version}\""

  [[tool.bumpversion.files]]
  filename = "zensical.toml"
  search = 'release = "{current_version}"'
  replace = 'release = "{new_version}"'

  [[tool.bumpversion.files]]
  filename = "docs/changelog.md"
  search = "## Unreleased"
  replace = "## Unreleased\n\n## [{new_version}] - {now:%Y-%m-%d}"

  [[tool.bumpversion.files]]
  filename = "CITATION.cff"
  search = 'version: "{current_version}"'
  replace = 'version: "{new_version}"'
  ```

  - `date-released` in `CITATION.cff` gets one more entry, with `regex = true`.
  - metaTOR and metacooler bump their version module instead of the first two entries.
- **`release.yml`** runs on a `v*` tag. It can also be run by hand on an existing tag, to rebuild without
  publishing again.
  1. **Test:** run `ci.yml` on the tagged commit.
  2. **Build and verify:**
     - `uv build` and `twine check`;
     - the tag is `v` followed by the built wheel's version, the `Version:` of its `METADATA`, which holds
       wherever a repository writes the version;
     - the changelog has a non-empty section for the version;
     - install the wheel in a clean environment, then run `<command> --version` and `--help`.
  3. **Publish** to PyPI by trusted publishing (`pypa/gh-action-pypi-publish`, environment `pypi`).
     - No token is stored, and the action uploads attestations.
     - This job stays in each repository's own workflow: PyPI's trusted publishing has not accepted a
       reusable workflow as the publisher.
  4. **GitHub release:** named `vX.Y.Z`, its body the changelog section, with the sdist and the wheel
     attached.
  5. **Container** (§ 15): built from the published wheel, tested, then pushed.
- **Zenodo:** its GitHub integration archives each release and gives it a DOI, reading `CITATION.cff`.
  The README badge shows the concept DOI.
- **Bioconda:** its bot picks up each PyPI release. When runtime dependencies changed, the recipe's are
  updated in bioconda-recipes the same day, by hand (the pull-request template asks).

## 15. Containers

References: metaTOR's `Dockerfile`, `.dockerignore`, and the `container` job of its `release.yml`;
instaGRAAL's `docker.yml` for images in several variants.

- **An image is published only when pip cannot install everything the package runs:** hicstuff's
  aligners, instaGRAAL's CUDA, tinymapper's aligners and peak callers. Every other package's container
  is the BioContainers image that bioconda builds.
- **The base image is pinned to a version** (`ghcr.io/mamba-org/micromamba:X.Y.Z`,
  `nvidia/cuda:X.Y.Z-…`), and Dependabot updates it.
- **The image is built from the released wheel**, not from the checkout. `.dockerignore` is an
  allowlist: `*`, then `!dist/*.whl` and the environment file.
- **Tags come from `docker/metadata-action`:**
  - `X.Y.Z` and `latest`;
  - variants by suffix (`X.Y.Z-cuda11.8.0`, `latest-cuda11.8.0`), with `latest` given to the variant
    that runs most widely (instaGRAAL: CUDA 11.8.0);
  - a tag scheme that is already published goes on being published beside these (metaTOR's `vX.Y.Z`);
  - the action's OCI labels, never a hand-written `LABEL Version=`.
- **Tested before pushing:** `docker run --rm <image> <command> --version`.
- **Platforms:** linux/amd64, and linux/arm64 wherever the base image has it.
- **Build cache:** `type=gha`, never a tag in the image's own repository.
- **Registry:** `ghcr.io/<owner>/<distribution>`.

## 16. Pull requests, branches and commits

- **Branches:**
  - one long-lived branch, the default, which keeps its current name;
  - work happens on short-lived branches;
  - releases are tags.
- **Every change goes through a pull request and is squash-merged.** A pull request is reviewed as a
  whole, and tested at its head (metaTOR's rule).
- **Every commit follows Conventional Commits:** `type(scope): summary`, with a body that says why.
  - The types are `feat`, `fix`, `perf`, `docs`, `test`, `refactor`, `build`, `ci` and `chore`. A breaking
    change adds `!` (`feat(cli)!: …`) and a `BREAKING CHANGE:` footer.
  - This includes the squash commit: its message is written as one when the pull request is merged.
  - A release commit is `chore(release): vX.Y.Z`.
- **Pull-request titles** need not follow it, unless the pull request is one self-contained change. Its
  title is then a Conventional Commit, and becomes its squash commit as it is.
- **The pull-request template's checklist:**
  - a changelog entry;
  - the documentation page a user reads about the change;
  - tests, covering at least 80 % of the changed lines;
  - `docs/cli.md`, for any command-line change;
  - the bioconda recipe, for any change to runtime dependencies.
- **A ruleset protects the default branch:**
  - a pull request is required;
  - `lint`, the stable legs of `test` and the docs build are required checks;
  - force pushes are blocked;
  - auto-merge is allowed, and head branches are deleted once merged.

## 17. Repository files

- **README.md:** as § 8 says.
- **LICENSE:** each repository keeps its own. `pyproject.toml` declares it as an SPDX expression.
- **CITATION.cff:**
  - a software entry with the version, the release date and the Zenodo concept DOI;
  - a `preferred-citation` for the paper, where there is one;
  - GitHub shows it as "Cite this repository", and Zenodo reads it.
- **CONTRIBUTING.md:** three lines pointing to `docs/contributing.md`. That page covers setting up,
  testing, documenting and releasing, as hicstuff's and metaTOR's pages do.
- **Community files** (code of conduct, security policy, support, issue forms, pull-request template):
  inherited from `koszullab/.github` or `js2264/.github`.
- **`.gitignore`:** the standard Python list, plus the repository's own outputs. It never lists
  `uv.lock` or `.python-version`.

## 18. Compatibility, versions and deprecation

- **The public interface:**
  - **the command line:** commands, option names and short flags, defaults, exit codes, and the names and
    formats of the files commands write;
  - **the Python API:** everything the API reference documents, meaning names without a leading
    underscore in the documented modules;
  - **file formats:** `.metacool` and `.mmetacool` (metacooler's `docs/format.md`), and the URIs that
    address them;
  - **logs and reports:** the log's file name and its header's keys (§ 11); the report's file names, the
    keys of its `report.txt`, and its MultiQC ids and columns, under its `format_version` (§ 12);
  - **distribution names:** PyPI and bioconda package names, container image names, and published tags.
- **A major release** is an incompatible change to the public interface. Before 1.0 (metacooler,
  instaGRAAL, chromosomix), a minor release may break it; a patch release never does.
- **A minor release** brings new features, raises dependency floors, or drops a Python version that a
  dependency no longer supports (§ 3).
- **A patch release** only fixes.
- **Deprecation comes before removal**, for at least one minor release:
  - the command line warns on stderr, naming the replacement;
  - the Python API raises `FutureWarning`;
  - the changelog says so under *Deprecated*.

  The removal comes in the next major release, with its migration page.
- **Nothing is renamed just to follow a convention.** Old names stay as aliases.
- **URLs do not move:** documentation pages, paths in the repository that others fetch (test data,
  example files), branch names.

## 19. Security and settings

- **CodeQL default setup**, for Python, for JavaScript/TypeScript where the repository has any, and for
  GitHub Actions.
- **`actions/dependency-review-action`** on pull requests.
- **Private vulnerability reporting** is on, and the security policy (§ 17) points to it.
- **Secrets:**
  - no PyPI token anywhere (trusted publishing);
  - a workflow names the secrets it needs;
  - `CODECOV_TOKEN` is an organisation secret on koszullab;
  - a GitHub App token, rather than a personal access token, wherever a workflow needs more than
    `GITHUB_TOKEN`.
- **Environments:** `pypi`, used by `release.yml` only, and `github-pages`.
- **Repository settings:**
  - squash merging only;
  - auto-merge allowed;
  - head branches deleted automatically;
  - Pages built from GitHub Actions.

## 20. Data, benchmarks and validation

- **Large data lives outside git:** in release assets versioned by the generator that made them
  (metacooler's `benchmark-data-v<N>`), or in a public bucket (chromosomix's demo data).
- **Data a paper or a page cites is archived on Zenodo**, as instaGRAAL's test data is
  (`10.5281/zenodo.19813387`).
- **History is never rewritten to remove a large file.** Doing so would break every clone and fork. The
  file stays in history, and new ones go elsewhere.
- **Benchmarks are optional.** Reference: metacooler's `benchmarks/`:
  - a Quarto notebook and a timing script;
  - deterministic datasets from a versioned generator;
  - a workflow that renders the notebook on every release and on demand;
  - the page published at `/benchmarks/` on the documentation site.
- **Validation against reference tools** follows chromosomix's `notebooks/validation.qmd`: each
  comparison names the reference's version.

## 21. Sharing, and instructions for agents

- **The `.github` repositories hold the default community files:** the code of conduct, the security
  policy, the support page, a generic `CONTRIBUTING.md`, the issue forms and the pull-request template.
  - `koszullab/.github` serves koszullab's repositories, and also holds this file. `js2264/.github`
    serves js2264's.
  - GitHub takes them only from a public `.github` repository, and the issue forms and their
    `config.yml` only from its `.github/ISSUE_TEMPLATE/` folder. It uses them in every repository of the
    owner, public or private, that has no file of its own.
  - A repository's own file replaces the default, and a repository with its own
    `.github/ISSUE_TEMPLATE/` folder gets none of the default forms. So a repository keeps files of its
    own only when it needs different ones, as chromosomix does.
- **chromokit** (`js2264/chromokit`) holds everything else the repositories share, under one version:
  - **the documentation's look,** checked by hash (`chromokit check`);
  - **the report kit** (§ 12), which is metaTOR's report machinery, generalised:
    - the page's base Jinja2 template, restyled with chromokit's look, which each tool's template
      extends;
    - the writers of metaTOR's `report.py`: `report_files`, `write_txt` and `read_txt`, `write_mqc`,
      `write_html` and `write_report`.

    `chromokit sync` puts these in `src/<package>/_report/`, and `chromokit check` refuses local edits to
    them, as it does for the docs;
  - **the copier template** of what each repository fills in with its own names:
    - the `pyproject.toml` blocks;
    - pre-commit, `.editorconfig` and `.gitattributes`;
    - the workflows, `dependabot.yml` and `codecov.yml`;
    - the docs skeleton and `tests/test_docs.py`;
    - `CITATION.cff` and `AGENTS.md`.

    It lives in chromokit's `template/` folder, which chromokit's `copier.yml` names as its
    `_subdirectory`, and its versions are chromokit's tags. A repository runs
    `copier copy gh:js2264/chromokit .` once, commits its `.copier-answers.yml`, and takes changes with
    `copier update`.
- **The steps several workflows share,** extracting the changelog section and smoke-testing the wheel, are
  written once in the template's workflows: `copier update` brings a fix to every repository, so no
  composite action is needed. Publishing to PyPI stays in each repository's `release.yml` (§ 14).
- **Instructions for coding agents: `AGENTS.md`, and nothing else.**
  - **One file at the root** holds everything an agent needs:
    - the commands (§ 4) and the layout;
    - the repository's own rules;
    - its procedures: reviewing a pull request, adding a command, the test data;
    - a link to this file.

    Rules for one part of the repository are a section of that file, not a nested `AGENTS.md`: VS Code's
    Copilot reads nested files only behind an experimental setting.
  - **Copilot** reads it natively, in its coding agent, in VS Code and in its CLI.
  - **Claude Code** reads it natively, from v2.1.277 on. It does so only when no `CLAUDE.md`,
    `.claude/CLAUDE.md` or `CLAUDE.local.md` exists in or above the working directory: one stray file
    switches it off.
    - So no `CLAUDE.md` is committed, and nobody runs `/init`, which writes one.
    - A contributor on an older Claude Code keeps a personal `CLAUDE.md` holding the one line
      `@AGENTS.md`. `.gitignore` lists `CLAUDE.md`, so that it is never committed by accident.
  - **No other agent files:** no `CLAUDE.md`, `.github/copilot-instructions.md`, `.github/instructions/`,
    `.github/agents/`, `.github/prompts/`, `.github/skills/`, `.claude/skills/` or `GEMINI.md`.
    - Their content moves into `AGENTS.md`.
    - A long procedure becomes ordinary documentation, such as the contributing page or the format
      specification, which `AGENTS.md` links to. Claude Code's docs advise keeping an instruction file
      under about 200 lines.
    - What is lost is the shortcut, not the procedure: a custom agent, a prompt or a skill becomes a
      section, or a page, that any agent follows. Claude Code loads skills only from `.claude/skills/`, so
      none is kept.
  - **Rules every repository's `AGENTS.md` states:**
    - a pull request is reviewed as a whole, and tested at its head;
    - a change a user can notice carries its changelog entry;
    - every commit is a Conventional Commit;
    - coverage stays at 80 %, overall and on the lines a pull request changes;
    - the command line is tested in-process;
    - diagnostics go through logging.

## 22. Across repositories

- **metacooler owns the `.metacool` format.** hicstuff, metaTOR and chromosomix read and write it only
  through metacooler, or through its specification.
- **Downstream testing:**
  - metacooler's CI runs hicstuff's and metaTOR's test suites against each of its pull requests (a
    `downstream` job);
  - hicstuff and metaTOR test against metacooler's default branch every week, in a job allowed to fail.
- **Contracts that are tested:**
  - **metacooler's sdist ships `tests/data/`.** chromosomix's CI builds its fixtures from it, and
    metacooler's release job checks that it is there.
  - **Every `metacooler>=` floor** is a version the downstream repository's `bounds` job passes with.
  - **A new chromosight release** updates the version chromosomix's validation notebook pins.
- **cooler** is upgraded in metacooler first. hicstuff and metaTOR follow through their locks.
- **One sample, one name, across tools:** a run's prefix carries the sample from hicstuff to metaTOR, so
  that MultiQC lines up their reports' rows (§ 12).
- **tinymapper runs hicstuff's command line** in its Hi-C mode, passing `<sample>^<hash>` as hicstuff's
  prefix. Its conda lock follows hicstuff's releases, and its Hi-C integration test runs every week
  against hicstuff's default branch.

## 23. chromosomix and other non-Python repositories

These sections apply:
- automation (§ 7), for `npm` and `github-actions`;
- releases (§ 14): the tag must be `package.json`'s version, npm trusted publishing with provenance, and
  a draft release with its assets, as chromosomix does today;
- the changelog's headings and sections (§ 9), in its root `CHANGELOG.md`;
- pull requests and commits (§ 16): its commits become Conventional Commits, keeping the SPECS section
  they touch in the summary, as in `fix(notebooks): ask again when a download's server fails (§18.10)`;
- compatibility (§ 18), through the contract stated at the top of its CHANGELOG: file formats and links;
- security (§ 19), data (§ 20), sharing and agents (§ 21), and the cross-repository contracts (§ 22).

Further rules:
- **Node:** `engines` names the oldest Node it supports, and CI builds with that Node as well as the
  newest.
- **Documentation:** its own Starlight site.
- **Report:** `chromosomix mapreport` is its report (§ 12). Its page, written in TypeScript, stays its
  own. Its MultiQC files keep their names (`_mqc.tsv`, `_mqc.json`) and their
  `chromosomix_` ids. They may add `parent_id: chromosomix`.
- **Logs:** a command line run that chains steps (a pipeline) writes its log beside its outputs, named as
  in § 11. A single command writes none.
- **Sub-packages:** `widgets/python` and `widgets/r` follow the Python conventions where they apply, and
  carry the application's version.
