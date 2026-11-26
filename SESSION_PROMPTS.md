# Prompts for the migration sessions

One cloud session per pull request. Start each with the repositories in its row attached, and paste its
prompt as the first message. Every session reads the plan itself, from `koszullab/.github`, and follows
`MIGRATION.md`'s "Making each pull request", so the prompts only say what is particular to each.

**Before session 1:** push both `.github` folders, koszullab's with the updated `CONVENTIONS.md` and
`MIGRATION.md`. Each session stops if its clone lacks "Making each pull request".

| # | Session | Attach (push to the first only) | Start when |
|---|---|---|---|
| 1 | chromokit: report kit and template | js2264/chromokit, koszullab/metaTOR, koszullab/hicstuff, koszullab/instaGRAAL, js2264/metacooler | now |
| 2 | metacooler | js2264/metacooler, js2264/chromokit (also pushed to), koszullab/hicstuff, koszullab/metaTOR | chromokit's release is tagged |
| 3 | hicstuff | koszullab/hicstuff, js2264/chromokit, js2264/metacooler | session 2's chromokit fixes, if any, are released |
| 4 | metaTOR | koszullab/metaTOR, js2264/chromokit, js2264/metacooler | as 3 |
| 5 | instaGRAAL | koszullab/instaGRAAL, js2264/chromokit | as 3, and instaGRAAL's #53 and #54 merged |
| 6 | chromosight | koszullab/chromosight, js2264/chromokit | as 3 |
| 7 | tinymapper | js2264/tinyMapper, js2264/chromokit, koszullab/hicstuff | as 3 |
| 8 | chromosomix | js2264/chromosomix | now: it does not use the template |

Sessions 3 to 7 can run side by side. hicstuff's and metaTOR's pull requests merge after metacooler's.
Each session ends with a draft pull request; you review it, squash-merge it, then release
(`bump-my-version`, then the tag) and do the manual steps its description lists.

---

## 1. chromokit: the report kit and the template

```text
You are making the first pull request of a migration across seven repositories: js2264/chromokit's, which adds the report kit (R-0) and the copier template (2.T) that every later pull request applies.

The plan is in koszullab/.github, which no session can attach (its name starts with a dot). Clone it into your scratchpad: git clone --depth 1 https://github.com/koszullab/.github. Read CONVENTIONS.md in full, then MIGRATION.md: "Ground rules", "Order", "Making each pull request", 2.T, R-0, § 12 of the conventions again, and appendix B. If MIGRATION.md has no "Making each pull request" section, stop and tell me: the clone is out of date.

Read chromokit before designing anything: its README, the format of chromokit.json, the sync and check commands, its tests and its release process. hicstuff, metaTOR and instaGRAAL (after its #54) use chromokit 0.1.0 with "target": "zensical": their chromokit.json must keep working unchanged.

One pull request to chromokit, holding:

1. The report kit (R-0), generalised from koszullab/metaTOR's src/metator/report.py and src/metator/templates/report.html.j2:
   - the page's base Jinja2 template, with metaTOR's header, navigation, cards, sortable and filterable tables, Download CSV buttons, plotly plots (pinned, from its CDN) and footer, restyled with chromokit's look in both palettes; a tool's template extends it with its own sections;
   - the writers report_files, write_txt and read_txt, write_mqc, write_html and write_report, taking the tool's name, its FORMAT_VERSION and its General Statistics columns as parameters;
   - `sync` writes them into src/<package>/_report/ and `check` verifies their hashes; since they are never edited, they import relatively and name no package;
   - tests: a sample report written from metaTOR's, report.txt read back by read_txt, and two samples' report_mqc.yaml through `uvx multiqc`, with their General Statistics rows merged into one table;
   - screenshots of the sample page, light and dark, sent to me.
2. The copier template (2.T), in template/, named by a copier.yml at chromokit's root (_subdirectory: template), and versioned by chromokit's tags:
   - the files and questions 2.T lists, built from the references it names (metaTOR's workflows and pyproject.toml blocks, hicstuff's install checks, js2264/metacooler's bounds job, instaGRAAL's docs.yml from #54) and appendix B's snippets;
   - the files a repository owns once rendered (README, docs pages, AGENTS.md, CITATION.cff) are _skip_if_exists; no _tasks, so applying it never needs --trust;
   - tests, run by chromokit's CI: render it with the answers of metacooler, hicstuff, metaTOR, instaGRAAL, chromosight and tinymapper; actionlint and zizmor on the rendered workflows; uv lock and uv run pytest in a sample package rendered from it.

Every commit is a Conventional Commit. When everything passes, open one draft pull request into chromokit's default branch, describing what each repository will see and how to cut the release (I tag it). Push only to chromokit: the other repositories are references.
```

## 2. metacooler

```text
You are making metacooler's pull request in a migration across seven repositories, which follows koszullab/.github's CONVENTIONS.md and MIGRATION.md.

The plan is in koszullab/.github, which no session can attach. Clone it into your scratchpad: git clone --depth 1 https://github.com/koszullab/.github. Read CONVENTIONS.md in full, then MIGRATION.md's "Ground rules", "Order", "Making each pull request", metacooler's section (MC-1 to MC-6), 1.1, R-0, R-1, R-4, X-1 and the appendices, and follow "Making each pull request" step by step. If it has no such section, stop and tell me. metacooler's CLAUDE.md and .github/ instruction files hold until MC-6 replaces them.

Your pull request holds MC-1 to MC-6, R-1, R-4 and X-1, into devel (metacooler has no main branch).

It is the first to apply chromokit's template, from its latest tag. Where the template is wrong rather than metacooler, fix it in one pull request to js2264/chromokit, holding every fix, so that a patch release follows; link the two pull requests to each other.

Points to watch:
- R-4, `metacooler report`: keep metacooler's layering (reading in io/, computing in ops/, the command in cli/cmd_report.py with its imports inside the function, listed in COMMAND_GROUPS and in docs/cli.md). jinja2 becomes a runtime dependency.
- R-1: the commands that write a map or a file write <sample>.metacooler_<YYYYMMDDHHMMSS>.log beside it, with hicstuff's header; those that only read keep logging to stderr.
- MC-6: CLAUDE.md and the Copilot files fold into AGENTS.md, under about 200 lines; the long procedures become docs pages it links to. Before deleting a file, check each of its rules against AGENTS.md and those pages: none may be lost.
- X-1: the downstream job runs koszullab/hicstuff's and koszullab/metaTOR's tests (attached) against the pull request's metacooler. Run it once here.
- 1.1: check whether metacooler is on bioconda (say so if anaconda.org is out of reach). If it is not, draft its recipe in the description (grayskull pypi metacooler, noarch: python).
- metacooler's repository is private: say in the description what that blocks (X-2's weekly legs in hicstuff and metaTOR).
- The release, 0.5.0, is prepared in docs/changelog.md, not cut.

When everything passes, open the draft pull request, and chromokit's if there is one. Push only to metacooler and, for the template's fixes, to chromokit.
```

## 3. hicstuff

```text
You are making hicstuff's pull request in a migration across seven repositories, which follows koszullab/.github's CONVENTIONS.md and MIGRATION.md.

The plan is in koszullab/.github, which no session can attach. Clone it into your scratchpad: git clone --depth 1 https://github.com/koszullab/.github. Read CONVENTIONS.md in full, then MIGRATION.md's "Ground rules", "Order", "Making each pull request", hicstuff's section (HS-1 to HS-6), R-0, R-1, R-3, X-2 and the appendices, and follow "Making each pull request" step by step, with chromokit's latest tag (js2264/chromokit, attached). If it has no such section, stop and tell me.

Your pull request holds HS-1 to HS-6, R-1, R-3 and X-2, into master.

Points to watch:
- R-3, `hicstuff report`: it reads a run's output directory and never runs the analysis again; `hicstuff pipeline` writes it at its end. Its numbers are the ones `hicstuff stats` reads off the log. The report kit comes from `npx github:js2264/chromokit sync` and is never edited. jinja2 and pyyaml become runtime dependencies.
- R-1: every command that writes files (digest, filter, rebin, subsample, convert, …) writes <prefix>.hicstuff_<YYYYMMDDHHMMSS>.log, as `hicstuff pipeline` already does.
- X-2: a weekly leg against metacooler's default branch, allowed to fail. metacooler's repository is private (js2264/metacooler is attached, so you can try the leg here). If it still is, ask me before adding any token; otherwise leave the leg out and say why.
- HS-5: the image is built from the wheel, with the same tags as today.
- Template fixes: list them in the description; do not edit chromokit.
- The release, 4.1.0, is prepared in docs/changelog.md, not cut.

When everything passes, open the draft pull request. Push only to hicstuff.
```

## 4. metaTOR

```text
You are making metaTOR's pull request in a migration across seven repositories, which follows koszullab/.github's CONVENTIONS.md and MIGRATION.md.

The plan is in koszullab/.github, which no session can attach. Clone it into your scratchpad: git clone --depth 1 https://github.com/koszullab/.github. Read CONVENTIONS.md in full, then MIGRATION.md's "Ground rules", "Order", "Making each pull request", metaTOR's section (MT-1 to MT-5, MT-7), R-0, R-1, R-2, X-2 and the appendices, and follow "Making each pull request" step by step, with chromokit's latest tag (js2264/chromokit, attached). If it has no such section, stop and tell me. metaTOR's CLAUDE.md holds until MT-5 replaces it with AGENTS.md.

Your pull request holds MT-1 to MT-5, R-1, R-2 and X-2, into master. metaTOR is the reference for most of the conventions: where the template differs from metaTOR's own file, work out which is right before taking either, and list the template's errors in the description; do not edit chromokit.

Points to watch:
- R-2: the report adopts chromokit's kit. report.txt (format 1), report_mqc.yaml (id metator, the same columns) and the page's content stay as they are; only the page's look changes. Show it: one run's report.txt and report_mqc.yaml identical byte for byte before and after, and screenshots of the page before and after.
- MT-1: the license expression becomes GPL-3.0-or-later.
- R-1: check that every command that writes files writes its log.
- X-2: a weekly leg against metacooler's default branch, allowed to fail. metacooler's repository is private (js2264/metacooler is attached, so you can try the leg here). If it still is, ask me before adding any token; otherwise leave the leg out and say why.
- MT-7: draft the bioconda recipe for 2.0 in the description. It waits for metacooler on bioconda (1.1).
- The release, 2.0.0, is prepared in docs/changelog.md with metaTOR's own 2.0 changes, not cut.

When everything passes, open the draft pull request. Push only to metaTOR.
```

## 5. instaGRAAL

```text
You are making instaGRAAL's pull request in a migration across seven repositories, which follows koszullab/.github's CONVENTIONS.md and MIGRATION.md.

First check that koszullab/instaGRAAL's pull requests #53 and #54 have merged into master. If either has not, stop and tell me.

The plan is in koszullab/.github, which no session can attach. Clone it into your scratchpad: git clone --depth 1 https://github.com/koszullab/.github. Read CONVENTIONS.md in full, then MIGRATION.md's "Ground rules", "Order", "Making each pull request", instaGRAAL's section (IG-1 to IG-5), 1.2, 1.4, R-0, R-1, R-5 and the appendices, and follow "Making each pull request" step by step, with chromokit's latest tag (js2264/chromokit, attached) and its GPU answer. If it has no such section, stop and tell me.

Your pull request holds IG-1 to IG-5, R-1 and R-5, into master.

Points to watch:
- GPU runs cost money. Never start a RunPod pod or a GPU workflow run yourself, even though this environment can reach RunPod's API: run the CPU tests here, and ask me before any GPU run.
- IG-1: uv.lock has to resolve pycuda without CUDA ([[tool.uv.dependency-metadata]]), and the image installs the wheel with its gpu extra only.
- R-1 is a soft change: log.py stops opening instagraal-<date>.log in the working directory when it is imported, even for --help; each command attaches <sample>.instagraal_<YYYYMMDDHHMMSS>.log in its output directory, with hicstuff's header. The changelog says so, and the Makefile's clean target follows.
- R-5, `instagraal-report`: a console script beside the others, which instagraal-endtoend runs at its end. It reads the run's output directory only.
- Template fixes: list them in the description; do not edit chromokit.
- The release, 0.3.0, is prepared in docs/changelog.md with #49's and #53's changes, not cut.

When everything passes, open the draft pull request. Push only to instaGRAAL.
```

## 6. chromosight

```text
You are making chromosight's pull request in a migration across seven repositories, which follows koszullab/.github's CONVENTIONS.md and MIGRATION.md.

The plan is in koszullab/.github, which no session can attach. Clone it into your scratchpad: git clone --depth 1 https://github.com/koszullab/.github. Read CONVENTIONS.md in full, then MIGRATION.md's "Ground rules", "Order", "Making each pull request", chromosight's section (CS-1 to CS-7), 1.4, R-0, R-1, R-6 and the appendices, and follow "Making each pull request" step by step, with chromokit's latest tag (js2264/chromokit, attached). If it has no such section, stop and tell me.

Your pull request holds CS-1 to CS-7, R-1 and R-6, into master. It changes more than any other: setup.py becomes pyproject.toml built by hatchling, Python 3.10 or newer is declared, docopt becomes docopt-ng, Sphinx on Read the Docs becomes zensical on GitHub Pages, and the Dockerfile goes.

Points to watch:
- The snapshot (appendix A) guards the rewrite. The command line is docopt's, so the script's fallback runs each command's --help: the usage text, the version and the installed files change only as the plan says.
- No result changes (ground rule 2): run detect and quantify on the test data at the base and at the head, and compare their outputs.
- R-6, `chromosight report`: a new docopt subcommand that reads a run's <prefix> outputs. jinja2 and pyyaml become runtime dependencies.
- R-1: detect, quantify and generate-config write <prefix>.chromosight_<YYYYMMDDHHMMSS>.log beside their outputs.
- CS-4: every page of the Read the Docs site finds its place on the new one. List the addresses that stop working once I delete the Read the Docs project (1.4).
- Template fixes: list them in the description; do not edit chromokit.
- The release, 1.7.0, is prepared in docs/changelog.md, not cut.

When everything passes, open the draft pull request. Push only to chromosight.
```

## 7. tinymapper

```text
You are making tinymapper's pull request in a migration across seven repositories, which follows koszullab/.github's CONVENTIONS.md and MIGRATION.md. The repository is js2264/tinyMapper; the package and its command are tinymapper.

The plan is in koszullab/.github, which no session can attach. Clone it into your scratchpad: git clone --depth 1 https://github.com/koszullab/.github. Read CONVENTIONS.md in full, then MIGRATION.md's "Ground rules", "Order", "Making each pull request", tinymapper's section (TM-1 to TM-7), R-0, R-1, R-8, X-2 and the appendices, and follow "Making each pull request" step by step, with chromokit's latest tag (js2264/chromokit, attached). If it has no such section, stop and tell me.

Your pull request holds TM-1 to TM-6, R-1, R-8 and X-2, into master.

Points to watch:
- tinymapper runs programs pip cannot install (bwa-mem2, bowtie2, STAR, samtools, deepTools, MACS3, hicstuff, cooler). Its integration tests run in the micromamba environment built from env/conda-lock.yml: run what this environment allows, and say what you could not.
- R-1 is a soft change: logs/<YYMMDD>-<sample>_<hash>.log becomes logs/<sample>.tinymapper_<YYYYMMDDHHMMSS>.log; the run's hash moves into hicstuff's header; the .err and _script.txt files take the same name. The changelog says so.
- R-8, `tinymapper-report`: a second console script, and every run writes its report at its end. The samtools, STAR and MACS3 outputs that MultiQC reads keep the sample's name. A Hi-C run links to hicstuff's report: settle how that report learns the plain sample name rather than <sample>^<hash> (koszullab/hicstuff is attached to read), and say how in the description.
- TM-5: the indexes leave git, without rewriting history: built in CI from the FASTA files, or fetched from a release asset; the large-file hook is added.
- Coverage is held to 60 % today: measure it, and say how far it is from 80 %.
- X-2: a weekly leg against hicstuff's default branch, allowed to fail.
- TM-7 waits for hicstuff 4 on bioconda: draft the recipe in the description only.
- Template fixes: list them in the description; do not edit chromokit.
- The release, 0.15.0, is prepared in docs/changelog.md, not cut.

When everything passes, open the draft pull request. Push only to tinyMapper.
```

## 8. chromosomix

```text
You are making chromosomix's pull request in a migration across seven repositories, which follows koszullab/.github's CONVENTIONS.md and MIGRATION.md. chromosomix is a TypeScript application: § 23 of the conventions says which parts apply to it.

The plan is in koszullab/.github, which no session can attach. Clone it into your scratchpad: git clone --depth 1 https://github.com/koszullab/.github. Read CONVENTIONS.md (§ 23 first, then the sections it names), then MIGRATION.md's "Ground rules", "Making each pull request" (without steps 3 and 4, and without the Python checks), 1.3, chromosomix's section (CX-1 to CX-6), R-1 and R-7. If it has no such section, stop and tell me. chromosomix's CLAUDE.md, SPECS.md and CONTRIBUTING.md hold until CX-5 replaces CLAUDE.md with AGENTS.md.

Your pull request holds CX-1 to CX-5, R-1 and R-7, into main.

Points to watch:
- CX-2: check that the name chromosomix is free on PyPI before wiring the widget's publishing, and say what the trusted publisher and R-universe need from me.
- CX-4: CodeQL's default setup is a setting, which I turn on; the pull request adds the dependency review.
- CX-5: AGENTS.md takes CLAUDE.md's rules. The ui-gallery skill (.claude/skills/ui-gallery) becomes a contributor page of the documentation site, which AGENTS.md links to. Then CLAUDE.md and .claude/skills/ go, and .gitignore lists CLAUDE.md. Commits become Conventional Commits that keep the SPECS section in the summary, starting with this pull request's.
- R-1 and R-7 are optional: the command line's log, and parent_id: chromosomix in its MultiQC files. Do them only where SPECS allows without changing what users rely on, and update SPECS and CHANGELOG.md if you do.
- Checks: npm run typecheck, npm test, npm run test:dev and npm run docs:build. If anything on screen changes, run the UI gallery and send me the screenshots.
- The release, 0.22.0, is prepared in CHANGELOG.md, not cut.

When everything passes, open the draft pull request. Push only to chromosomix.
```
