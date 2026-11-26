# Migration plan

What each repository has to change to follow `CONVENTIONS.md`, in which order, and what users may
notice. It was written from these commits, on 26 November 2026:

| Repository | Commit |
|---|---|
| chromosight | `2e6bf59` |
| hicstuff | `3cf59f9` |
| instaGRAAL | `e4906fc`, with #53 and #54 taken as merged (`42d23ac`); they merged as `07b36f0` and `6031add` |
| metaTOR | `af54142` |
| metacooler | `8c650e7` |
| chromosomix | `60d32e6` |
| tinymapper | `cf09301` |

For instaGRAAL, two open pull requests were taken as merged. Both have merged since (26 November), squashed
into `master`, whose head is `6031add`:
- **#54** moves the documentation to Zensical on GitHub Pages;
- **#53**, which #54 is stacked on, adds `instagraal-refine` and makes pycuda a `gpu` extra.

The checkouts don't show repository settings, secrets or bioconda recipes. Where an item depends on one,
it says to check.

Each item has an **effort** and a **break** level:
- **Effort:** S is up to half a day, M up to two days, L longer.
- **Break,** what users may notice:
  - **no:** nothing a user could notice;
  - **soft:** a warning, or a change no working setup depends on;
  - **yes:** an incompatible change;
  - **conda** or **docker:** affects only that channel.

**Each repository gets one pull request**, holding all of its items. That covers its section of phases 2
to 4, its log and report items (R-1 to R-8), and its cross-repository jobs (X-1, X-2). chromokit gets one
too, before them, holding the report kit (R-0) and the template (2.T). The phases list what goes in each
pull request; "Order" says how the pull requests land.

## Status, 26 November 2026

Checked against the repositories' heads. The checkboxes below follow it. The settings items (0.2, 0.3) and
what only the maintainer can do (releases, PyPI, bioconda, Zenodo) don't show in a checkout, and are left
as they were.

| Step | State |
|---|---|
| `.github` repositories (0.1) | Done |
| chromokit: the report kit (R-0) and the template (2.T) | Done: v0.2.0, then v0.2.1 with the fixes that metacooler's pull request found (#2) |
| metacooler (MC-1 to MC-6, R-1, R-4, X-1) | Pull request #35 merged into `devel` (`79f75e4`), then chromokit 0.2.1 taken (`f50bd8b`). **0.5.0 is not released**, and MC-5's Zenodo step is open |
| instaGRAAL | #53 and #54 merged. Its pull request has not started |
| hicstuff, metaTOR, chromosight, tinymapper, chromosomix | Not started: each default branch is still at the commit this plan was written from |
| 1.1 metacooler on bioconda | A recipe exists in js2264's fork (0.2.0 added on 17 November, 0.4.0 on 26 November); not confirmed in bioconda itself |

**Next:** sessions 3 to 8 of `SESSION_PROMPTS.md` can all start, hicstuff and metaTOR (3 and 4) first.
Release metacooler 0.5.0 before hicstuff and metaTOR.

## Ground rules

1. **Rename nothing that users or other projects call.** That covers commands, options, modules, output
   files, documentation URLs, test-data paths, branches and published container tags. Add an alias
   instead.
2. **Change no behaviour.** No item here changes what a command computes or writes. Ruff fixes are applied
   only when ruff calls them safe.
3. **Verify every packaging change by diffing,** with the snapshot script (appendix A), run at the base
   commit and at the pull request's head. The sdist and wheel file lists, the metadata, `__version__` and
   every command's `--help` must match, apart from the intended change.
4. **User-visible changes ship in a minor (or major) release, never a patch,** with a changelog entry.

## Order

The shared work, which lives outside the seven repositories, comes first. That way no repository needs a
second pull request.

```text
1      Outside the seven repositories, in this order:
       the .github repositories (0.1); one chromokit pull request with the report kit (R-0)
       and the copier template (2.T), and its release; metacooler on bioconda (1.1)
2      One pull request per repository:
       metacooler first; then hicstuff and metaTOR; then instaGRAAL, chromosight, tinymapper and chromosomix
3      Around each pull request: the settings it needs (0.2, 0.3, instaGRAAL's Pages source, the
       trusted publishers, deleting Read the Docs: 1.4), then its release (bump-my-version),
       then its bioconda recipe
Later  the end-to-end dataset (X-4), versioned documentation (X-5), benchmarks (X-6): work of their own
```

| Repository | Its pull request holds | Lands after | Release |
|---|---|---|---|
| metacooler | MC-1 to MC-6, R-1, R-4, X-1 | the kit and the template | 0.4.0 → **0.5.0** |
| hicstuff | HS-1 to HS-6, R-1, R-3, X-2 | metacooler's pull request | 4.0.0 → **4.1.0** |
| metaTOR | MT-1 to MT-5, R-1, R-2, X-2 | metacooler's pull request | 1.4.0 → **2.0.0**, with its own 2.0 changes |
| instaGRAAL | IG-1 to IG-5, R-1, R-5 | #53 and #54 | 0.2.0 → **0.3.0**, with #49's and #53's changes |
| chromosight | CS-1 to CS-7, R-1, R-6 | the kit and the template | 1.6.3 → **1.7.0** |
| chromosomix | CX-1 (1.3) to CX-5, R-1, R-7 | the kit | 0.21.0 → **0.22.0** |
| tinymapper | TM-1 to TM-6, R-1, R-8, X-2 | the kit and the template | 0.14.25 → **0.15.0** |

Dependencies:
- **The template (2.T) is built first, in chromokit,** from the reference files the conventions name.
  metacooler's pull request, the first, validates it, and the later ones apply it. Building it after the
  repositories would cost each of them a second pull request.
- **The report kit (R-0) is built first too, in the same chromokit pull request,** tested with metaTOR's
  current report as its sample. Its page layout is settled before any repository uses it. A chromokit
  release then carries both, and every repository's pull request starts from that tag.
- **metacooler's pull request goes first:** it is upstream of hicstuff and metaTOR, and validates the
  template.
- **instaGRAAL's pull request lands after #53 and #54.**
- **hicstuff's and metaTOR's bioconda recipes wait for item 1.1.**
- **Phase 1's item 1.3 joins chromosomix's pull request.** Land it on its own only if it cannot wait.
- **Each pull request is large,** the report commands especially. It is reviewed as a whole, and
  squash-merged (metaTOR's rule).

## Making each pull request

Each pull request is made in a session of its own, in the same steps. chromosomix's skips steps 3 and
4, and the Python checks (conventions § 23).

1. **Read first:** `CONVENTIONS.md`; this plan's ground rules, "Order" and the repository's section;
   its rows in "Releases" and "What users may notice"; the appendices. Then the repository's own
   instructions for agents (`CLAUDE.md`, `AGENTS.md`, `.github/copilot-instructions.md`): they hold until
   the pull request replaces them.
2. **Check the plan against the head of the default branch.** The plan was written from the commits at
   its top. An item already done is skipped, an item whose ground has moved is checked again, and
   anything that contradicts the plan is reported rather than forced.
3. **Snapshot the base** (appendix A) before changing anything.
4. **Apply the template** from chromokit's latest tag (2.T): `uvx copier copy --vcs-ref <tag>
   --data-file <answers> --overwrite gh:js2264/chromokit .`, then review `git diff` file by file.
   - The repository's own version stays where the conventions name it as the reference, or where the
     difference is a deviation, written down where it is.
   - A difference that the template should absorb goes to chromokit's next patch release. metacooler's
     session, the first, fixes chromokit itself, in one pull request there; the later ones list theirs in
     their pull request's description, so that no two sessions edit chromokit at once.
5. **Then the items,** in the plan's order, one or a few Conventional Commits each.
6. **Verify at the head,** and say in the pull request how:
   - the test suite;
   - coverage: 80 % of the changed lines (`diff-cover` against the default branch), and the overall
     figure, with how far it is from 80 % where it is below;
   - the snapshot (appendix A): only the intended differences, each one listed;
   - `uv lock --check`, ruff, ty, pre-commit, actionlint and zizmor;
   - the documentation: `uv run --locked --only-group docs zensical build --clean --strict`, then
     `npx github:js2264/chromokit check`;
   - the report: written from the test data and read back by `read_txt`; two samples' files through
     `uvx multiqc`, with their General Statistics rows merged; screenshots of the page;
   - the log: a command run, its log's name and header checked.
7. **Open one pull request** into the default branch, as a draft, with a plain title: it is not one
   self-contained change. Its description lists:
   - the items done, and any left out, with the reason;
   - the snapshot's differences, and what users may notice;
   - what only the maintainer can do: the settings (0.2), the secrets (0.3), the Pages source, the
     trusted publisher, Read the Docs (1.4), the bioconda recipe (drafted in the description), and the
     release (`bump-my-version`, then the tag).
8. **Never** push to the default branch, tag, publish, or change a repository's settings. Ask before
   anything that costs money (instaGRAAL's GPU runs), and before deleting anything the plan does not
   name.

## Phase 0: shared scaffolding

- [x] **0.1 Fill `koszullab/.github` and `js2264/.github`.** S, no.
  - Both exist and are public; koszullab's holds `CONVENTIONS.md` and this plan, which stays there until
    the last pull request merges.
  - Both get the same community files: `CODE_OF_CONDUCT.md`, `SECURITY.md`, `SUPPORT.md`, a generic
    `CONTRIBUTING.md`, `.github/ISSUE_TEMPLATE/` (bug, feature, `config.yml`) and
    `.github/pull_request_template.md` (the checklist of conventions § 16). Each owner's files name its
    own issues and contact.
  - Start from chromosomix's files, made generic. Its bug form asks app-specific questions; the generic
    one asks for the version, the install channel (pip, conda, Docker), the command and the log.
  - **Pushed by hand.** A cloud session cannot attach a repository whose name starts with a dot: the
    files are prepared in a session and pushed by the maintainer. Sessions read them through an
    anonymous clone (`git clone https://github.com/koszullab/.github`).
  - These defaults apply to every repository of the owner, public or private, without a file of its
    own. Today chromosomix alone has its own code of conduct, security policy, issue forms and
    pull-request template, and keeps them (CX-6). Every repository keeps, or gets, its own
    `CONTRIBUTING.md` (conventions § 17), so the default one serves only the others.
  - **Done 26 November:** both repositories have the files on their default branch. Each also holds its
    hand-push bundle at its root (`koszullab-dotgithub.zip`, `js2264-dotgithub.zip`), committed by
    mistake: delete them.
- [ ] **0.2 Audit each repository's settings** against conventions § 19, which the checkouts don't show.
  S per repository. Check:
  - merge methods, auto-merge, deleting head branches;
  - the ruleset on the default branch, and its required checks;
  - private vulnerability reporting, Dependabot alerts and security updates;
  - CodeQL default setup;
  - the Pages source;
  - the `pypi` environment, and the trusted publishers registered on PyPI.
- [ ] **0.3 Make `CODECOV_TOKEN` an organisation secret on koszullab**, shared with its four
  repositories, and delete the repository copies. S, no.

## Phase 1: fixes that do not wait for the conventions

- [ ] **1.1 Get metacooler onto bioconda.** M, conda.
  - Check first: anaconda.org could not be reached from where this plan was written. metacooler's docs
    give no conda install, and metaTOR's `pyproject.toml` says to "submit its recipe if it is not" there.
  - **26 November:** js2264's fork of bioconda-recipes has a `recipes/metacooler` (`noarch: python`): 0.2.0
    on the branch `js/update-metacooler` (17 November), 0.4.0 on `js/metacooler-0.4.0`. Whether either
    merged into bioconda is not known: anaconda.org is still unreachable from a session. On that branch,
    hicstuff's recipe is 3.3.0 and metaTOR's 1.3.10.
  - Until it is there, the conda packages of hicstuff 4.0.0, already on PyPI, and of metaTOR 2.0 cannot
    be built: both require `metacooler>=0.4.0`.
  - Recipe: start from `grayskull pypi metacooler`, `noarch: python`. The run dependencies are those of
    `pyproject.toml`: cooler, h5py, numpy, pandas, rich-click, rich, pyfaidx, matplotlib, scipy, click,
    pyyaml, pyarrow. Tests: `metacooler --help` and the import.
  - Then hicstuff's recipe for 4.0.0: add metacooler, pybwa (check that conda has it) and `pysam>=0.23`.
  - Then metaTOR's for 2.0 (MT-7).
- [x] **1.2 instaGRAAL's image tags: done in #54.**
  - A release now pushes `X.Y.Z-cuda<cuda>` and `latest-cuda<cuda>`, and `latest` from the CUDA 11.8.0
    build.
  - The tags 0.2.0 left behind, `v0.2.0` and `latest`, hold whichever CUDA build finished last. They stay
    until 0.3.0; the docs point to `master-cuda<cuda>` in the meantime.
- [ ] **1.3 Turn on Dependabot in chromosomix.** S, no.
  - `npm` for `/`, `/documentation` and `/notebooks/viewers` (three lockfiles), with minor and patch
    updates grouped;
  - `github-actions`;
  - `pip` for `/widgets/python`.

  Merge by hand at first: its test job rebuilds fixtures from PyPI packages and runs long.
- [ ] **1.4 Delete Read the Docs.** S, yes (links).
  - **instaGRAAL:** delete the `instagraal` project on Read the Docs once #54 is merged and its site is on
    GitHub Pages. #54 merged on 26 November: check that the Pages site is live, then delete.
  - **chromosight:** its site serves its last build, and cannot rebuild: there is no `.readthedocs.yaml`,
    and its configuration relies on recommonmark, which is deprecated. Delete the project once CS-4 has
    put the site on GitHub Pages.
  - Links to `*.readthedocs.io` then stop working ("What users may notice").

## Phase 2: metacooler, hicstuff and metaTOR

### metacooler, next release 0.5.0

Pull request #35 merged into `devel` on 26 November (`79f75e4`), followed by two commits taking chromokit
0.2.1 (`9408641`, `f50bd8b`). Checked against `devel`: every item below is in it, except what MC-5 says.
The repository is `js2264/metacooler`. **0.5.0 is not released:** `current_version` is still 0.4.0, and
the latest tag is `v0.4.0`.

- [x] **MC-1 CI.** M, no.
  - **`lint`** job, as metaTOR's:
    - `uv lock --check`;
    - the locked ruff (`check`, `format --check`);
    - `ty check src tests`, required from the start if it passes (pre-commit already runs it);
    - the ruff-pin check (B.1);
    - zizmor.
  - **`test`:**
    - drop the `setup-python` step (setup-uv installs Python);
    - `uv sync --locked` rather than `--all-extras`;
    - a 3.15 leg allowed to fail, with `libhdf5-dev pkg-config` so that h5py builds (hicstuff's entry);
    - keep the macOS leg and the CLI smoke tests.
  - **`dependency-bounds`** becomes `bounds`. Add a weekly `schedule` to ci.yml's triggers.
  - **`install`** job: the wheel in a clean environment, with an empty `PATH`, since metacooler is pure
    Python.
  - **`permissions: contents: read`** and `timeout-minutes` everywhere.
  - **`retry-jobs-without-runner.yml`**, from hicstuff, for the macOS leg.
  - **Push triggers:** `devel` only. ci.yml and docs.yml also name a `main` branch, which does not exist.
  - **`changelog.yml`** (B.3).
- [x] **MC-2 Dependabot**, from hicstuff (`dependabot.yml` and `dependabot-automerge.yml`). S, no. Its
  first pull request brings the actions up to date: checkout 4 and 5, setup-uv 5, codecov 6, Pages 4,
  action-gh-release 2.
- [x] **MC-3 Packaging and tooling.** S, soft.
  - The `docs` extra becomes a dependency group.
    - docs.yml builds from the group alone:
      `uv run --locked --only-group docs zensical build --clean`, as instaGRAAL's does.
    - `docs/installation.md` loses `pip install "metacooler[docs]"`; pip ≥ 25.1 has `--group`.
    - `pip install metacooler[docs]` then warns, and installs metacooler.
  - pytest: `xfail_strict`, `--strict-markers`, `--strict-config`, `-ra`.
  - `[tool.uv] cache-keys` with `_version.py`.
  - ruff: add `W` and `RUF` to `select`.
  - pre-commit: ruff v0.15.14 → v0.17.4, the version in the lock.
  - Add:
    - `.editorconfig`;
    - `.gitattributes`, with `tests/data/*.metacool`, `*.mmetacool` and `*.gz` marked binary;
    - `.python-version`.
  - Codecov: `project` at 80 %, where it is informational today, and `patch` at 80 % (conventions § 13).
  - Run the snapshot (A) before and after.
- [x] **MC-4 Documentation.** M, no.
  - **chromokit:**
    - run `npx github:js2264/chromokit sync`;
    - in `docs/index.md`, rename the 14 `mc-` classes to `cx-`: the kit's stylesheet has every one;
    - move whatever is metacooler's own from `metacooler.css` to `docs/stylesheets/site.css`;
    - delete `metacooler.css`, `metacooler.js` and `overrides/partials/mc-page-footer.html`;
    - set `extra_css`, `extra_javascript` and `kicker` in `zensical.toml` as metaTOR does;
    - compare screenshots of the pages before and after.
  - **docs.yml:** a `build` job on every pull request (`--strict`, `chromokit check`), and the `deploy`
    job on the default branch, keeping its benchmark-page step. Fix whatever `--strict` reports.
  - **`tests/test_docs.py`**, from metaTOR, adapted to `docs/cli.md`'s headings. metacooler's CLAUDE.md
    calls that page the single source of truth; this test is what keeps it true.
  - **New pages:**
    - `contributing.md`, from the README's *Releasing* section and the commands in CLAUDE.md;
    - `citation.md`, citing the software until there is a paper.
- [ ] **MC-5 Changelog, citation and release.** S, no. Merged in #35, except Zenodo's integration (a
  setting, not checked) and the release itself.
  - **Changelog headings:**
    - each `## X.Y.Z - date` → `## [X.Y.Z] - date`; `## Unreleased` stays;
    - `## 0.1.0` → `## [0.1.0]`, deleting the `---` line above it;
    - bump-my-version's `replace` writes `## [{new_version}] - …` (conventions § 14).
  - **release.yml:**
    - run CI on the tag (`uses: ./.github/workflows/ci.yml`);
    - extract the notes with the shared extractor (B.2);
    - smoke-test the wheel;
    - check the sdist contract: `tar -tzf dist/*.tar.gz | grep -q '/tests/data/'`.
  - **`CITATION.cff`**, with its bump-my-version entry. Turn on Zenodo's GitHub integration.
- [x] **MC-6 Agent instructions.** S, no.
  - **One `AGENTS.md`** takes the content of CLAUDE.md (122 lines) and of the Copilot files (979 lines),
    which largely repeat it:
    - `.github/copilot-instructions.md`, which still says "Alpha (v0.1.0)";
    - `.github/instructions/` (CLI commands, HDF5 I/O, tests), which become sections;
    - `.github/agents/` (reviewer, HDF5 reviewer, tester, documenter), which become a section on reviewing
      and one on testing;
    - `.github/prompts/` and `.github/skills/`: their procedures become sections, or pages of the docs
      that `AGENTS.md` links to (adding a command, on the contributing page; the format, in
      `docs/format.md`).
  - **Then delete** CLAUDE.md and every file above, and add `CLAUDE.md` to `.gitignore`.
  - **Deduplicate as you merge:** Claude Code's docs advise keeping an instruction file under about 200
    lines.

### hicstuff, next release 4.1.0

- [ ] **HS-1 CI.** S, no.
  - **`lint`:**
    - `uv run --locked ruff` instead of `uvx ruff`, which takes the newest ruff;
    - `uv lock --check`;
    - ty, advisory;
    - the ruff-pin check;
    - zizmor.
  - **`test`:** `uv sync --locked`. CI does not check the lock today.
  - **`bounds`:** metacooler's job, with a weekly schedule. It shows which of the unbounded
    dependencies (numpy, scipy, biopython, cooler, python-dateutil) need a floor.
  - **`concurrency`, `permissions`, `timeout-minutes`**, and `changelog.yml`.
- [ ] **HS-2 Packaging.** M, soft.
  - **Build backend:** setuptools → hatchling ≥ 1.27. The version stays in `pyproject.toml`, as the
    conventions say.
  - **bump-my-version:** the Dockerfile entry goes with HS-5; the other entries stay.
  - **Groups:** the `dev` and `docs` extras become dependency groups, and the contributing page says
    `uv sync` and `uv run --only-group docs zensical serve`. `pip install hicstuff[dev]` then warns.
  - **Floors** from the `bounds` job, each with its comment.
  - **pytest:** the strict configuration. `fail_under = 85` may stay.
  - **sdist:** ship `test_data/` (19 MB, so packagers can run the tests), or leave it out with a
    comment.
  - **Snapshot (A):** the wheel should not change. Only `.py` files sit under `src/hicstuff`.
- [ ] **HS-3 Tooling.** S, no.
  - pre-commit-hooks v5 → v6, with `check-json` and `check-added-large-files`;
  - ruff v0.11.8 → v0.17.5, the version in the lock;
  - `SIM` and `RUF` added to ruff's `select`;
  - `.editorconfig`, and `.gitattributes` (`test_data/**` marked binary);
  - `.python-version`, taken out of `.gitignore`;
  - optionally, `.codecov.yml` moved to `.github/codecov.yml`;
  - Codecov's `project` at 80 %, rather than `auto` with a 1 % threshold.
- [ ] **HS-4 Release workflow.** M, no.
  - `publish.yml` becomes `release.yml`, as metaTOR's.
  - It gains a CI run on the tag, `twine check`, a wheel smoke test, and a GitHub release from the
    changelog. v3.3.0 and v4.0.0 were released by hand.
  - Publish with `pypa/gh-action-pypi-publish`, which uploads attestations. If `uv publish` now uploads
    them too, keep it.
  - PyPI's trusted publisher names the workflow file: register `release.yml` before deleting
    `publish.yml`.
- [ ] **HS-5 Image.** S, docker.
  - Build it from the released wheel.
  - Pin `mambaorg/micromamba` to a version.
  - Tags and labels from metadata-action: `X.Y.Z` and `latest`, the scheme hicstuff already publishes.
    Remove the hand-written `LABEL` and its bump-my-version entry.
  - Cache with `type=gha`, and delete the public `:buildcache` tag.
  - Smoke-test before pushing.
  - Build linux/arm64 too, if bioconda has the aligners for linux-aarch64 (check).
- [ ] **HS-6 Documentation and metadata.** S, no.
  - Changelog headings: `## 4.0.0 - 2026-11-23` becomes `## [4.0.0] - 2026-11-23`, with bump-my-version's
    `replace` to match; `## Unreleased` stays. *New* becomes *Added*.
  - `CITATION.cff`. First check which DOI is the concept DOI: the README badge says
    `10.5281/zenodo.2620608`, the citation page `10.5281/zenodo.4066363`, its BibTeX
    `10.5281/zenodo.4066351`.
  - A `CONTRIBUTING.md` pointer, and a contributing page that asks for Conventional Commits: hicstuff's
    history uses plain sentences.
  - `AGENTS.md`, and `CLAUDE.md` in `.gitignore`.
  - docs.yml: `chromokit check`, and a build from the docs group alone
    (`uv run --locked --only-group docs zensical build --clean --strict`, as instaGRAAL's).

### metaTOR, next release 2.0.0

- [ ] **MT-1 Packaging.** S, soft.
  - `license = "GPL-3.0-or-later"`, without the classifier.
  - `hatchling>=1.27`.
  - The `docs` extra becomes a dependency group, from which docs.yml builds alone (`--only-group docs`).
  - `[tool.uv] cache-keys` with `src/metator/version.py` (the module keeps its name).
  - `maintainers`.
  - The snapshot (A).
- [ ] **MT-2 CI.** M, no.
  - `latest-dependencies` becomes `bounds`, with a `lowest-direct` leg on the oldest Python.
  - An `install` job with an empty `PATH`: the docs say metaTOR needs "nothing outside PyPI".
  - A `wheels` job, for pyhmmer, pyrodigal, igraph and pysam.
  - `timeout-minutes`, `changelog.yml`, the ruff-pin check, zizmor.
  - Codecov's `project` at 80 %, rather than `auto` with a 1 % threshold.
- [ ] **MT-3 Make ty required.** M, no. Fix what `ty check src` reports, then drop `continue-on-error`.
- [ ] **MT-4 Release and image.** S, docker.
  - Run CI on the tag; extract the notes with B.2.
  - Image tags: add `X.Y.Z` beside the published `vX.Y.Z` and `latest`, and keep publishing `vX.Y.Z`.
  - linux/arm64, and `type=gha` caching.
  - Leave the old tags (`refs-tags-v1.3.7`, `py-3.11`) alone: deleting them breaks whoever pinned them.
- [ ] **MT-5 Documentation and metadata.** S, no. The changelog's headings already follow the
  conventions.
  - `CITATION.cff`, whose `preferred-citation` is the Frontiers in Genetics 2019 paper.
  - `AGENTS.md`, taking CLAUDE.md's rules on pull requests, which become every repository's (conventions
    § 21). Then CLAUDE.md goes, and `.gitignore` lists it.
  - `chromokit check` in docs.yml.
- [ ] **MT-7 The bioconda recipe for 2.0**, after 1.1, as metaTOR's `pyproject.toml` comment describes.
  S, conda.
  - Require `metacooler>=0.4.0`, pysam, pyrodigal and pyhmmer.
  - Drop hicstuff, pypairix, pyfastx, prodigal and hmmer.

### 2.T Build the template, in chromokit (L, no)

**Done 26 November:** chromokit v0.2.0 (`63ecc3d`) carries the template and the report kit (R-0); v0.2.1
(`3d3721c`) adds the template fixes that metacooler's pull request found (#2). The repositories' pull
requests start from v0.2.1.

Built before the repositories' pull requests (see "Order"), in the same chromokit pull request as the
report kit (R-0), from the reference files the conventions name:
- metaTOR's workflows and `pyproject.toml` blocks;
- hicstuff's install checks;
- metacooler's `bounds` job;
- instaGRAAL's `docs.yml`.

`tests/test_docs.py` exists in three copies, and chromokit 0.1.0 in three repositories.
- **Where:** chromokit's `template/` folder, named by a `copier.yml` at chromokit's root
  (`_subdirectory: template`). Its versions are chromokit's tags, so one chromokit release carries the
  docs' look, the report kit and the template together. A repository applies it with
  `copier copy gh:js2264/chromokit .`, and later `copier update`.
- **What it holds:**
  - the `pyproject.toml` blocks and tool configuration;
  - pre-commit, `.editorconfig`, `.gitattributes`;
  - the workflows, `dependabot.yml`, `codecov.yml`, with the steps they share written once (the
    changelog section, B.2; the wheel's smoke test);
  - the docs skeleton, `tests/test_docs.py`, `CITATION.cff`, `AGENTS.md`.
- **Its questions:**
  - distribution name, import package and command;
  - owner and default branch;
  - docstring style and line length;
  - whether to build an image, and from which base;
  - compiled dependencies (adds the `wheels` job);
  - nothing needed outside PyPI (an empty `PATH` in `install`);
  - GPU.
- **Tests, in chromokit:** render the template with each Python repository's answers (all but
  chromosomix), then check what it renders: actionlint and zizmor on the workflows, and `uv lock` and
  `uv run pytest` in a sample package rendered from it.
- **Apply it in each repository's pull request** (`.copier-answers.yml`), starting with metacooler's,
  which validates it. Each difference becomes a template option, or a deviation written down where it is.
  A difference found after chromokit's release goes into a chromokit patch release, which the
  repositories take with `copier update` in their own pull request.

## Phase 3: instaGRAAL, chromosight and tinymapper

Apply the template to each, then do the items below.

### instaGRAAL, next release 0.3.0

#53 and #54, merged on 26 November (`07b36f0`, `6031add`), already do much of what the conventions ask:
- **Documentation:** zensical and chromokit on GitHub Pages. `docs.yml` builds every pull request
  (`--strict`) from the `docs` dependency group alone, and deploys from `master`. `tests/test_docs.py`
  keeps the CLI reference honest, and `doc/` is deleted.
- **Project pages:** `docs/changelog.md` back to 0.1, `docs/citation.md`, `docs/upgrading.md`, and a
  `CONTRIBUTING.md` pointer.
- **Releases:** bump-my-version, replacing python-semantic-release, which nothing configured.
- **pycuda as the `gpu` extra:** the pycuda stub is gone, and the GPU pods run only the GPU tests.
- **The image tags** (1.2).
- **`example/example.gif`** (24 MB) removed, from the repository and from the sdist.

0.3.0 already breaks the interface through these pull requests:
- `instagraal-endtoend`'s default pipeline (#53);
- pycuda, which a plain `pip install instagraal` no longer installs (#53);
- the default of `instagraal-polish --junction` (#49).

The user-visible changes of this plan go in the same release.

- [ ] **IG-1 Packaging.** M, soft.
  - **Groups:** the `test` and `dev` extras become dependency groups, as `docs` already is.
    `pip install instagraal[dev]` then warns. The Dockerfile installs `[gpu,dev]`; IG-3 replaces that.
  - **Unused tools to drop:**
    - pylint and twine;
    - black (ruff formats);
    - isort (ruff's `I` rules do it);
    - python-dotenv and pytest-order (no test uses them);
    - mypy, if ty replaces it (the contributing page says CI does not run it).
  - **License:** drop the classifier; `license = "GPL-3.0-only"` stays.
  - **`requires-python = ">=3.10,<3.13"`:** the cap came in with #14, without a reason. Add 3.13 and
    3.14 to the GPU combinations; lift the cap if they pass, or comment it.
  - **Floors** from the `bounds` job. Only biopython has one.
  - **`hatchling>=1.27`.**
  - **`uv.lock`:** take it out of `.gitignore`, commit it, and add hicstuff's `uv.lock` entry to
    bump-my-version.
    - Check first that `uv lock` resolves without CUDA. It resolves every extra, `gpu` included, and
      pycuda publishes only an sdist.
    - If uv has to build pycuda to read its metadata, declare that metadata in
      `[[tool.uv.dependency-metadata]]`.
    - CPU CI then installs without the `gpu` extra; no stub is needed any more.
  - **`scripts/instagraal-setup`** installs from a `requirements.txt` that does not exist, and compiles
    pycuda by hand. Delete it (the sdist ships it), or rewrite it around the `gpu` extra.
  - **The snapshot (A).**
- [ ] **IG-2 CI.** M, no.
  - **test-cpu:**
    - install through uv (`uv sync --locked`), without the `gpu` extra;
    - it runs on every push and every pull request alike: filter pushes to `master`;
    - add `concurrency` and `timeout-minutes`.
  - **Lint** with ruff only, dropping black and its configuration.
  - **One type checker:** ty (advisory), or the mypy configuration, run.
  - **Codecov** uploads with `cpu` and `gpu` flags: the GPU job uploads too, and the project's 80 % counts
    both.
  - **`changelog.yml`, and Dependabot with auto-merge.** Its first pull request brings test-cpu's and
    docker.yml's actions up to date: checkout 4, setup-python 5, login 3, build-push 5.
- [ ] **IG-3 Release.** M, docker.
  - **`cd.yml` becomes `release.yml`,** with:
    - a tag check (`cd.yml` has none), and a CI run on the tag;
    - `twine check`, and a wheel smoke test (without a GPU, every command's `--help` works since #53);
    - trusted publishing: register `release.yml` on PyPI, then delete `PYPI_TOKEN`;
    - a GitHub release from the changelog.
  - **The image** is built from the released wheel, with `[gpu]` only. Today's images install
    `[gpu,dev]`, and so carry the test and lint tools, which they would no longer have.
    - Keep #54's tags and its `master-cuda*` builds.
    - Drop the Dockerfile's hand-written `LABEL` and its bump-my-version entry: metadata-action writes
      the labels.
- [ ] **IG-4 Documentation, after #54.** S, no.
  - Set **Settings → Pages → Source** to GitHub Actions, or the deploy job fails (#54's to-do list).
  - Delete the Read the Docs project (1.4).
  - Run `npx github:js2264/chromokit check` in docs.yml; #54 ran it by hand.
- [ ] **IG-5 Metadata.** S, no.
  - `CITATION.cff`, from `docs/citation.md` (Genome Biology 2020, `10.1186/s13059-020-02041-z`).
  - `AGENTS.md`.
  - pre-commit as in conventions § 5. It replaces hooks v2.3.0, black 22.3.0 and ruff v0.6.9 (from the
    old `charliermarsh` URL).
  - `.editorconfig`, `.gitattributes`, and a Codecov badge once IG-2 uploads.

### chromosight, next release 1.7.0

- [ ] **CS-1 Packaging.** M, soft.
  - **`pyproject.toml` (hatchling)** replaces `setup.py`, `setup.cfg`, `MANIFEST.in` and
    `requirements.txt`.
  - **Version:** written in `pyproject.toml`, as the conventions say. `chromosight/version.py`, which
    `setup.py` rewrote at every build, stays importable, and reads the version with
    `importlib.metadata`.
  - **Layout:** optionally `src/`; imports do not change. `kernels/*` and `cli/logo.txt` sit inside the
    package and ship with it. Compare the snapshot (A) with the 1.6.3 wheel on PyPI.
  - **Python:** `>=3.6` is declared, the classifiers say 3.8–3.10, and CI tests 3.10–3.13. Make it
    `>=3.10`, with matching classifiers. Python 3.6–3.9 users keep getting 1.6.3.
  - **Floors** from the `bounds` job. Only `scipy>=1.3` has one today.
  - **docopt** is unmaintained, and emits `SyntaxWarning`s on Python 3.13 (invalid escape sequences).
    Swap it for docopt-ng, a maintained fork with the same module name and API, and check that the
    snapshot's `--help` and the tests do not change.
  - **Tests:** nose2 → pytest, which collects both the unittest classes and the plain test functions.
  - **The `dev` group.**
- [ ] **CS-2 CI.** M, no.
  - The template's ci.yml replaces `build.yml`, which:
    - runs on every push and every pull request alike;
    - uses actions v2;
    - uploads with the retired `codecov` uploader;
    - runs nose2.
  - Delete `codeql.yml`, which runs CodeQL v2, in favour of default setup (0.2).
  - Dependabot with auto-merge, and `changelog.yml`.
- [ ] **CS-3 Release.** S, no.
  - The template's `release.yml` replaces `python-publish.yml`, which:
    - runs on `release: created`;
    - has a `branches:` filter, which release events ignore;
    - publishes with a token, built with `setup.py sdist bdist_wheel`.
  - Register the trusted publisher, then delete `PYPI_TOKEN`.
- [ ] **CS-4 Documentation.** L, no.
  - Sphinx → zensical and chromokit; `TUTORIAL.md` becomes guides.
  - The two notebooks: link to them (GitHub renders them), or convert them to pages.
  - The API reference from mkdocstrings.
  - Then delete the Read the Docs project (1.4).
- [ ] **CS-5 Metadata.** S, no.
  - `docs/changelog.md`, rebuilt from the GitHub releases.
  - `CITATION.cff`, citing the Nature Communications 2020 paper.
  - `CONTRIBUTING.md`, `AGENTS.md`, pre-commit, `.editorconfig`, `.gitattributes`.
  - Delete the Makefile: its targets call `setup.py`, twine and nose2.
- [ ] **CS-6 Delete the Dockerfile.** S, docker. Its base is miniconda 4.8.2, and no workflow ever built
  it. The README already points to the BioContainers image.
- [ ] **CS-7 Large files.** The two 12 MB GIFs stay in history, and the pre-commit hook stops new ones.

### tinymapper, next release 0.15.0

tinymapper maps and processes ChIP-seq, RNA-seq, ATAC-seq, MNase-seq, Hi-C and shotgun reads. It
orchestrates programs that pip cannot install: bwa-mem2, bowtie2, STAR, samtools, deepTools, MACS3,
hicstuff and cooler. Its Hi-C mode runs `hicstuff pipeline` and `hicstuff distancelaw`. It is on PyPI
(0.14.25), and the README installs it from a conda lock.

What it already does as the conventions say:
- uv with a lock that CI installs (`--frozen`);
- hatchling, and the version in `pyproject.toml`;
- trusted publishing;
- a sdist that leaves out its 200 MB of test data;
- integration tests run with the real programs, in a micromamba environment made from the conda lock.

- [ ] **TM-1 Packaging.** S, soft.
  - **conda-lock** moves from the runtime dependencies to the dev group. No module imports it: only the
    README's development cycle runs it, to lock the conda environment again. Today every install pulls it
    and its dependencies.
  - **License:** `license = { text = "GPL-3.0" }` becomes `license = "GPL-3.0-or-later"`, as metaTOR's.
  - **The `dev` extra** becomes a dependency group. `pip install tinymapper[dev]` then warns.
  - **Metadata it lacks:** authors, keywords, classifiers (Python versions, operating systems), and
    `[project.urls]`.
  - **`hatchling>=1.27`.**
  - **The `tinyMapper.sh` shim** (`data/tinyMapper.sh`, installed by `shared-scripts`) stays: Slurm scripts
    and autotinymapper call it.
  - **The legacy `tinyMapper.sh`** at the root (1,617 lines of bash, "LEGACY_MODE") is not installed. Keep
    it, or move it to an `archive/` folder with a line in the README.
  - **The snapshot (A).**
- [ ] **TM-2 CI.** M, no.
  - **Pythons:** 3.13 and 3.14, and 3.15 allowed to fail; today it tests 3.10 to 3.12 only.
  - **Triggers:** pushes filtered to `master` (CI also names a `devel` branch), concurrency,
    `permissions` and `timeout-minutes`.
  - **lint:** `uv lock --check`, ty (advisory), and the ruff-pin check (B.1). pre-commit pins ruff
    v0.11.8 while the lock has 0.15.12.
  - **Coverage:** uploaded to Codecov, and held to 80 %, overall and on changed lines. Only
    `--cov-fail-under=60` checks it today, so its tests have to catch up before the project check
    passes.
  - **Also:** a `bounds` job, `changelog.yml`, Dependabot with auto-merge.
  - **`test_commands`** (the micromamba integration tests) stays: it is how a tool that needs conda
    programs runs its `install` job.
  - **A weekly leg against hicstuff's default branch** (X-2), allowed to fail. The conda lock pins
    hicstuff 3.2.5, so the Hi-C mode has not met hicstuff 4.0.0 yet.
- [ ] **TM-3 Release.** M, no.
  - **One `release.yml`** replaces two workflows that a tag starts side by side:
    - `publish.yml` publishes to PyPI without waiting for the tests or checking the tag;
    - CI's `release` job makes the GitHub release, with no notes.

    The new workflow checks the tag and runs CI on it, then `twine check`, the wheel's smoke test,
    trusted publishing (the pypa action), and a GitHub release with the changelog's section.
  - **bump-my-version:** `pyproject.toml`, `uv.lock`, `zensical.toml`, the changelog, `CITATION.cff`.
  - **The conda lock, locked again at every release.**
    - Today it pins tinymapper 0.14.24, installed by pip, while 0.14.25 is out.
    - `env/tinymapper.yaml` installs tinymapper from the head of the default branch, not from a release.
    - Since the README installs `env/conda-lock.yml` from `master`, that path stays, and it pins the
      release it was locked for.
- [ ] **TM-4 Documentation.** L, no.
  - zensical and chromokit replace `_config.yml`, the Jekyll theme that GitHub Pages renders the README
    with. The site stays where it is.
  - The README's sections become pages:
    - installation;
    - one guide per mode;
    - the output layout, and its `<sample>^<operation>^<hash>.<ext>` names, on `formats.md`;
    - Slurm;
    - the development cycle, on the contributing page.
  - `cli.md`, `tests/test_docs.py`, docs.yml.
  - `docs/changelog.md`, started from the GitHub releases; a citation page.
- [ ] **TM-5 Test data.** M, no.
  - `tests/` holds two genomes with their bowtie2 and bwa-mem2 indexes, about 200 MB. The largest index
    files are 36 MB, and even a shallow clone fetches 151 MB.
  - Build the indexes in CI from the FASTA files, or fetch them from a release asset, and add the
    large-file hook. History keeps the files: it is not rewritten.
- [ ] **TM-6 Metadata.** S, no.
  - `CITATION.cff`, a `CONTRIBUTING.md` pointer and `AGENTS.md`.
  - pre-commit as in conventions § 5.
  - `.editorconfig` and `.gitattributes`.
  - README badges.
  - `.gitignore`: its generator left an error line in it ("jupyternotebook is undefined").
- [ ] **TM-7 Conda and image.** M, conda and docker.
  - A bioconda recipe with every program as a run dependency, so that `conda install tinymapper`
    replaces the lock for users. BioContainers then builds the image.
  - It needs hicstuff 4 on conda first (1.1), if it requires hicstuff 4.

## Phase 4: chromosomix

- [ ] **CX-1 Dependabot:** item 1.3.
- [ ] **CX-2 Widgets.** M, no.
  - Have the `version` hook write the application's version into `widgets/python/pyproject.toml` and
    `widgets/r/DESCRIPTION`. Today they read 0.17.0.dev0 and 0.16.0.9000, against 0.21.0. The hook is
    `scripts/changelog-check.mjs`, or a script beside it.
  - Have `release.yml` publish the Python widget to PyPI, by trusted publishing (check that the name
    `chromosomix` is free there), and the R widget to R-universe.
- [ ] **CX-3 Node 22.12.** S, no. Build with it too, since `engines` names it as the minimum.
- [ ] **CX-4 Security.** S, no. CodeQL default setup for JavaScript/TypeScript, Python and Actions;
  dependency review on pull requests.
- [ ] **CX-5 Metadata.** S, no. `CITATION.cff`, `.editorconfig` and `AGENTS.md`.
  - **`AGENTS.md`** takes CLAUDE.md's rules (the UI gallery, the dev-feature markers, SPECS, the docs,
    the validation notebook). Then CLAUDE.md goes, and `.gitignore` lists it.
  - **The ui-gallery skill** (`.claude/skills/ui-gallery/SKILL.md`, 558 lines) becomes a contributor
    page that `AGENTS.md` links to, and that people can follow too. It runs
    `node scripts/ui-gallery.mjs gallery` against `npm run dev`, then looks at every picture and sends
    them. `.claude/skills/` goes.
  - **Commits:** CONTRIBUTING's rule becomes Conventional Commits, keeping the SPECS section in the
    summary: `fix(notebooks): ask again when a download's server fails (§18.10)`.
- [ ] **CX-6 Community files:** chromosomix keeps its own, since they are specific to the app. They are
  0.1's source.

## Phase 5: across repositories

- [x] **X-1 metacooler's `downstream` job** (B.5): hicstuff's and metaTOR's tests, run against each
  metacooler pull request. M, no. Done in #35, allowed to fail (`continue-on-error`).
- [ ] **X-2 A weekly leg against an upstream's default branch,** allowed to fail: hicstuff and metaTOR
  against metacooler's, tinymapper against hicstuff's. S, no.
  - metacooler's repository was private when this plan was written (26 November 2026). It was public when
    checked later that day, so the legs against it need no token. If it goes private again, they need a
    GitHub App token that can read it (conventions § 19).
- [x] **X-3 The sdist contract:** in MC-5. Done in #35: `release.yml` checks for `tests/data` in the sdist.
- [ ] **X-4 An end-to-end toy dataset.** L, no. Reads, run through hicstuff, then metaTOR, then
  chromosomix's fixtures. Stored as a release asset, and run weekly.
- [ ] **X-5 Versioned documentation for the zensical sites.** L, no. A copy per release, `/dev/` and
  pull-request previews, following chromosomix's `docs.yml` and `preview.yml`, built into chromokit.
- [ ] **X-6 Benchmarks for hicstuff and metaTOR**, on metacooler's pattern. L, no; optional.

## Phase 6: logs and reports

- [x] **R-0 The report kit, in chromokit,** in the same pull request as the template (2.T). L, no. Done:
  chromokit v0.2.0.
  - **Taken from metaTOR's `report.py` and `templates/report.html.j2`,** generalised:
    - the base Jinja2 template of the page, with metaTOR's header, navigation, cards, sortable and
      filterable tables, *Download CSV* buttons, plotly plots and footer, restyled with chromokit's look;
      each tool's template extends it with its own sections;
    - the writers: `report_files`, `write_txt` and `read_txt`, `write_mqc`, `write_html`,
      `write_report`, with the tool's name, its `FORMAT_VERSION` and its General Statistics columns as
      parameters.
  - **Dependencies:** jinja2 and pyyaml, as metaTOR declares them (`jinja2>=3.1`, `pyyaml>=6.0`).
  - **chromokit's manifest:** a `report` kit beside `zensical`; today `chromokit.json` names a single
    `target`. `sync` writes `src/<package>/_report/`, and `check` verifies it.
  - **Tests, in chromokit:** write a sample report; read its `report.txt` back; run MultiQC on two
    samples' files, and check that their General Statistics rows merge.
- [ ] **R-1 Logs.** S per repository.
  - **hicstuff:** `hicstuff pipeline` already writes `<prefix>.hicstuff_<date>.log`, with the header. Its
    other commands that write files (`digest`, `filter`, `rebin`, `subsample`, `convert`, …) start
    writing theirs. No break.
  - **metaTOR:** compliant: `<name>.metator_<date>.log`, the header and the closing line. Check that every
    command that writes files writes its log.
  - **metacooler:** new. The commands that write a map or a file write `<sample>.metacooler_<date>.log`
    beside it, `<sample>` being the output's name. The commands that only read (`info`, `ls`, `tree`,
    `show`, a `dump` to stdout) keep logging to stderr only. No break. In #35 (26 November): `cli/logging_utils.py`
    and a `setup_logging` call in the commands; the log's name and header were not checked.
  - **instaGRAAL:** `log.py` opens `instagraal-<date>.log` in the working directory when it is imported,
    even for `--help`. Instead, each command attaches `<sample>.instagraal_<date>.log` in its output
    directory, with the header. Soft: the log moves and is renamed. The Makefile's `clean` target, which
    removes `instagraal-*.log`, follows.
  - **chromosight:** new. `detect`, `quantify` and `generate-config` write
    `<prefix>.chromosight_<date>.log` beside their outputs. No break.
  - **tinymapper:** `logs/<YYMMDD>-<sample>_<hash>.log` becomes
    `logs/<sample>.tinymapper_<YYYYMMDDHHMMSS>.log`. The run's hash moves into hicstuff's header,
    replacing tinymapper's own (`yy-mm-dd HH:MM:SS | [INFO] …`). Its `.err` and `_script.txt` follow the
    same name. Soft: scripts that read a log by its old name; a glob on `logs/*.log` still finds it.
  - **chromosomix:** its command line writes `<stem>.chromosomix_<date>.log` beside what it writes.
    Optional.
- [ ] **R-2 metaTOR adopts the kit.** S, no.
  - Its template now extends the kit's base, and `report.py` calls the kit's writers.
  - `report.txt` (format 1), `report_mqc.yaml` (id `metator`, the same columns) and the page's content
    are unchanged. The page takes chromokit's look.
- [ ] **R-3 `hicstuff report`.** L, no.
  - Reads a run's output directory, and runs at the end of `hicstuff pipeline`.
  - **The reads and the pairs,** from the numbers `hicstuff stats` reads off the log:
    - pairs sequenced and pairs mapped;
    - pairs on short contigs;
    - pairs filtered (loops, uncuts, abnormal);
    - PCR duplicates;
    - cis and trans contacts, and the trans ratio.
  - **Then** the distance law, the matrix's bins, and the restriction fragments.
  - **MultiQC's General Statistics:** pairs, mapped share, duplicate share, trans ratio.
  - **Dependencies:** jinja2 and pyyaml are new for hicstuff.
- [x] **R-4 `metacooler report`.** L, no. Done in #35.
  - Reads a map: a `.metacool`, or an entry of a collection.
  - **Content:**
    - genomes, contigs, bins and contacts;
    - where the contacts land: the QC block's four categories, stored by `metacooler qc` or computed;
    - a table of the genomes;
    - P(s).
  - **MultiQC's General Statistics:** genomes, contigs, contacts, the inter-genome share.
  - **Dependencies:** jinja2 is new for metacooler, which has pyyaml already.
- [ ] **R-5 `instagraal-report`.** L, no.
  - Reads a run's output directory, and runs at the end of `instagraal-endtoend`.
  - **Content:**
    - the mode, and the steps it ran;
    - the assembly before and after: `instagraal-stats`' N50, L50 and sequence count;
    - refine's joins and circular chromosomes, from `junctions.tsv` and `refine_report.txt`;
    - the contact maps before and after.
  - **MultiQC's General Statistics:** the N50 before and after, and the scaffolds.
  - **Dependencies:** jinja2 and pyyaml are new for instaGRAAL.
- [ ] **R-6 `chromosight report`.** L, no.
  - A new docopt subcommand. It reads a run's `<prefix>` outputs: `<prefix>.tsv`, and the pileup.
  - **Content:**
    - the pattern, the kernel and its parameters;
    - how many patterns were found;
    - their scores and distances;
    - the pileup.
  - **MultiQC's General Statistics:** the patterns found.
  - **Dependencies:** jinja2 and pyyaml are new for chromosight.
- [ ] **R-7 chromosomix.** S, no.
  - Its map report stays its own page, written in TypeScript.
  - Optionally, its MultiQC files add `parent_id: chromosomix`.
- [ ] **R-8 `tinymapper-report`.** L, no.
  - A second console script, beside `tinymapper`, which is a single command. Every run also writes the
    report at its end.
  - **Content, by mode:**
    - reads, mapped, filtered and duplicates (samtools);
    - the spike-in calibration (ChIP);
    - peaks and FRiP (ChIP, ATAC);
    - fragment sizes (MNase, ATAC);
    - strands (RNA).
  - **Hi-C runs:** link to hicstuff's own report. tinymapper passes `<sample>^<hash>` as hicstuff's
    prefix, so for the two tools' MultiQC rows to line up, settle how that report learns the plain sample
    name.
  - **For MultiQC's own modules:** keep the samtools statistics, STAR's `Log.final.out` and MACS3's
    peaks under the sample's name, so that MultiQC reads them too.
  - **MultiQC's General Statistics:** mode, reads, mapped share, kept share, duplicates, peaks, FRiP.
  - **Dependencies:** jinja2 and pyyaml are new for tinymapper. Today they reach it through conda-lock,
    which TM-1 removes.

Each of R-2 to R-6, and R-8, also brings:
- its section on `formats.md`;
- its entry in `cli.md`;
- a test of the report on the test data;
- a CI step that runs MultiQC on that report.

## Releases

| Repository | Next release | What users may notice from this plan |
|---|---|---|
| chromokit | its next minor | the report kit and the copier template; a repository sees nothing until it syncs or applies them |
| metacooler | 0.5.0 | the `docs` extra removed; logs; a report |
| hicstuff | 4.1.0 | a report; the `dev` and `docs` extras removed; new dependency floors; release notes on GitHub; the image built from the wheel, same tags |
| metaTOR | 2.0.0 | 2.0's own changes, plus: the license declared as `GPL-3.0-or-later`; the `docs` extra removed; image tags `X.Y.Z` added |
| instaGRAAL | 0.3.0 | the breaking changes of #49 and #53 (in its changelog), and #54's documentation move and removed `docs` extra; from this plan: the `test` and `dev` extras removed, dependency floors, the Python cap reviewed, the log moved, a report, images without the dev tools |
| chromosight | 1.7.0 | Python ≥ 3.10 declared (≥ 3.6 before); dependency floors; docopt-ng; the documentation on GitHub Pages; a log; a report; no Dockerfile |
| chromosomix | 0.22.0 | the widgets on PyPI and R-universe |
| tinymapper | 0.15.0 | conda-lock no longer installed with it; the `dev` extra removed; the log renamed; a report; a release workflow that waits for the tests; documentation |

With one pull request per repository, each repository's logs and report ship in the release above.

The order of releases:
- chromokit releases first, with the kit and the template (done: v0.2.0, v0.2.1).
- metacooler releases next (0.5.0: merged, not yet released).
- hicstuff and metaTOR follow.
- instaGRAAL and chromosight do not depend on the others.
- Bioconda follows every release whose run dependencies changed.

## What users may notice

| Change | Repositories | Who notices, and how | Avoidable? |
|---|---|---|---|
| Dependency floors made true, and chromosight's Python range corrected | hicstuff, instaGRAAL, chromosight, and any repository whose `bounds` job fails | Environments pinned to old versions: pip upgrades the dependency, or refuses to install. Those environments do not work today either: the metadata promises combinations that fail | **No**, if the metadata is to be true |
| Published extras removed: `hicstuff[dev,docs]`, `instagraal[test,dev]` (its `docs` extra went in #54), `metator[docs]`, `metacooler[docs]`, `tinymapper[dev]` | 5 | Anyone installing with one: pip and uv warn that it does not exist, then install the package without the tools | **Yes:** keep them for one more minor release, as deprecated |
| Read the Docs deleted; the documentation on GitHub Pages | chromosight; instaGRAAL, in #54 | Links to `*.readthedocs.io`, in papers, pages and old READMEs on PyPI, stop working | **Yes,** by keeping the projects; deleting them is your decision |
| Build rewritten: setup.py → hatchling (chromosight), setuptools → hatchling (hicstuff), `src/` layout (chromosight) | 2 | A user, only if a file drops out of the wheel, which the snapshot (A) shows. Building from a checkout with `python setup.py …` stops working; `pip install .` works | **Yes,** except the end of `python setup.py …`, which comes with removing `setup.py` |
| docopt → docopt-ng | chromosight | No one expected (same module, same parsing). Checked with the `--help` snapshot and the tests | **Yes:** optional, though docopt itself may stop working on a future Python |
| `py.typed` added | hicstuff, metaTOR (optional) | Projects that type-check their calls into these packages get new diagnostics | **Yes:** add it only once the public API is annotated |
| instaGRAAL's log: from `instagraal-<date>.log` in the working directory to `<sample>.instagraal_<date>.log` in the output directory | instaGRAAL | Scripts that read the log where it was. hicstuff's and metaTOR's logs already follow the convention, and metacooler and chromosight gain logs they did not have | **No,** if its logs are to follow the convention |
| Reports: new `report` commands and files; metaTOR's page restyled on the kit | all | Readers see new pages. metaTOR's numbers, its `report.txt` and its MultiQC columns are unchanged | Nothing breaks: these are additions |
| New runtime dependencies for the reports: jinja2 and pyyaml | hicstuff, instaGRAAL, chromosight, tinymapper; metacooler (jinja2) | Installs pull two small pure-Python packages, and the bioconda recipes add them | Nothing breaks |
| tinymapper: conda-lock no longer installed with it | tinymapper | Anyone who used the `conda-lock` command a tinymapper install brought | **Yes,** but it is a development tool that tinymapper never imports |
| tinymapper's log: `logs/<YYMMDD>-<sample>_<hash>.log` becomes `logs/<sample>.tinymapper_<date>.log` | tinymapper | Scripts that read a log by its name; a glob on `logs/*.log` still finds it | **No,** if its logs are to follow the convention |
| Renaming commands, options, modules, test-data paths, doc pages or branches | none | — | Not in this plan: aliases only |

Conda and Docker changes are left out of this table, as asked: items 1.1, HS-5, MT-4, MT-7, IG-3, CS-6 and TM-7,
and #54's image tags.

instaGRAAL 0.3.0 also carries breaking changes this plan did not make, decided in #49 and #53:
- `instagraal-endtoend`'s default pipeline;
- pycuda, now the `gpu` extra;
- the default of `instagraal-polish --junction`.

## Appendix A: the packaging snapshot

Tested on metacooler (click, 44 help pages) and chromosight (docopt, `setup.py`); it leaves the
repository clean.

```bash
#!/usr/bin/env bash
# packaging-snapshot.sh OUTDIR DIST PACKAGE
# Writes down what a user installs, so that two snapshots can be diffed:
#   packaging-snapshot.sh /tmp/base hicstuff hicstuff    # at the base commit
#   packaging-snapshot.sh /tmp/head hicstuff hicstuff    # at the pull request's head
#   diff -r -x venv -x dist /tmp/base /tmp/head
# Run from the repository's root.
set -euo pipefail
out=$1 dist=$2 pkg=$3
rm -rf "$out" && mkdir -p "$out"
uv build -q --out-dir "$out/dist"          # the sdist, then the wheel built from it
tar -tzf "$out"/dist/*.tar.gz | cut -d/ -f2- | grep -v '/$' | sort > "$out/sdist.txt"
unzip -Z1 "$out"/dist/*.whl | grep -v '\.dist-info/' | sort > "$out/wheel.txt"
unzip -p "$out"/dist/*.whl '*.dist-info/METADATA' |
  grep -E '^(Requires-Python|Requires-Dist|Provides-Extra|License-Expression|License):' |
  sort > "$out/metadata.txt"
unzip -p "$out"/dist/*.whl '*.dist-info/entry_points.txt' > "$out/entry_points.txt"
uv venv -q "$out/venv"
uv pip install -q --python "$out/venv/bin/python" "$out"/dist/*.whl
COLUMNS=100 "$out/venv/bin/python" - "$out" "$dist" "$pkg" <<'EOF'
"""Every command's --help, and __version__, as the installed wheel gives them."""
import importlib, importlib.metadata as md, pathlib, subprocess, sys

out, dist, pkg = pathlib.Path(sys.argv[1]), sys.argv[2], sys.argv[3]
(out / "version.txt").write_text(f"{importlib.import_module(pkg).__version__}\n")
try:
    import click
    from click.testing import CliRunner
except ImportError:
    click = None

for ep in md.distribution(dist).entry_points:
    if ep.group != "console_scripts":
        continue
    root = ep.load()
    if click is None or not isinstance(root, click.Command):
        # docopt and other parsers: the top-level help only.
        run = subprocess.run([pathlib.Path(sys.executable).parent / ep.name, "--help"],
                             capture_output=True, text=True)
        (out / f"help-{ep.name}.txt").write_text(run.stdout + run.stderr)
        continue

    def walk(command, path):
        result = CliRunner().invoke(root, [*path, "--help"], prog_name=ep.name)
        (out / f"help-{'-'.join([ep.name, *path])}.txt").write_text(result.output)
        if isinstance(command, click.Group):
            for name in sorted(command.commands):
                walk(command.commands[name], [*path, name])

    walk(root, [])
EOF
ls "$out"
```

## Appendix B: snippets

### B.1 Pre-commit's ruff is the locked ruff

For the `lint` job. Tested: it passes on metaTOR, and fails on hicstuff (v0.11.8 against 0.17.5) and on
metacooler (v0.15.14 against 0.17.4).

```bash
locked=$(awk '/^name = "ruff"$/ { getline; gsub(/"/, "", $3); print $3; exit }' uv.lock)
grep -A3 'astral-sh/ruff-pre-commit' .pre-commit-config.yaml | grep -q "rev: v$locked\$" ||
  { echo "::error::pre-commit's ruff is not the ruff in uv.lock ($locked)"; exit 1; }
```

### B.2 This version's changelog section

For `release.yml`, replacing metaTOR's and metacooler's two versions.
- **What it reads:** the `## [X.Y.Z]` heading, then everything up to the next `## ` heading or a `---`
  line. `## Unreleased` and `## [Unreleased]` both work.
- **Tested on** instaGRAAL's 0.2.0 (#54), metaTOR's 1.3.10 and 1.4.0, and chromosomix's 0.21.0.
- **No section:** a missing version, or a heading without brackets (metacooler's today), gives nothing,
  and the check below fails.

```bash
awk -v ver="$VERSION" '
  /^## / { if (found) exit; if (index($0, "## [" ver "]") == 1) found = 1; next }
  found && /^---$/ { exit }
  found
' docs/changelog.md > release-notes.md
grep -q '[^[:space:]]' release-notes.md ||
  { echo "::error::docs/changelog.md has no entry for $VERSION"; exit 1; }
```

### B.3 `changelog.yml`

A workflow of its own, so that adding the label re-runs this check and nothing else.

```yaml
name: Changelog

on:
  pull_request:
    types: [opened, synchronize, reopened, labeled, unlabeled]

permissions:
  contents: read

jobs:
  entry:
    name: Changelog entry
    # Skipped, which counts as passed, when the pull request is labelled no-changelog.
    if: ${{ !contains(github.event.pull_request.labels.*.name, 'no-changelog') }}
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0
          persist-credentials: false
      - name: A change to src/ comes with a changelog entry
        env:
          BASE: ${{ github.event.pull_request.base.sha }}
        run: |
          changed=$(git diff --name-only "$BASE"...HEAD)
          if grep -q '^src/' <<<"$changed" && ! grep -qx 'docs/changelog.md' <<<"$changed"; then
            echo "::error::src/ changed but docs/changelog.md did not. Add an entry under Unreleased, or label the pull request no-changelog."
            exit 1
          fi
```

### B.4 instaGRAAL's image tags

Done in #54: its `docker.yml` is the reference for images in several variants (conventions § 15).

### B.5 metacooler's `downstream` job

```yaml
  downstream:
    name: ${{ matrix.repository }} with this metacooler
    runs-on: ubuntu-latest
    timeout-minutes: 45
    continue-on-error: true   # advisory until both suites are known to be stable here
    strategy:
      fail-fast: false
      matrix:
        repository: [koszullab/hicstuff, koszullab/metaTOR]
    steps:
      - uses: actions/checkout@v7
        with:
          path: metacooler
          persist-credentials: false
      - uses: actions/checkout@v7
        with:
          repository: ${{ matrix.repository }}
          path: downstream
          persist-credentials: false
      - uses: astral-sh/setup-uv@v7
      - name: Its locked environment, with this metacooler
        working-directory: downstream
        run: |
          uv sync --locked          # hicstuff needs --extra dev until HS-2
          uv pip install ../metacooler
      - name: Its tests
        working-directory: downstream
        # --no-sync: plain `uv run` would put the locked metacooler back.
        run: uv run --no-sync pytest -q --no-cov
```
