# Git mailing list daily digest for 2026/09/05

## Long threads

### Adding `path.*` keys to `git repo info`

*Long thread: 31 replies so far from 5 participants over 52 days; 1 message that day.*

K Jayatheerth's GSoC series extends `git repo info` with keys that output paths of repository components: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, and `path.cdup`, most supporting absolute and relative suffixes, with documentation and tests. Previously, Junio C Hamano identified that `get_superproject_working_tree()` does not accept a `repo` parameter and instead relies on the process's current working directory, so `git --git-dir=... repo info path.superproject-root.absolute` returns an empty value while `git -C ...` returns the correct root. Today K Jayatheerth confirmed the function relies on `xgetcwd()` rather than inspecting the target repository, and committed to updating it to take a `repo` parameter and update its callers, along with adding `--git-dir` test coverage in v6.

## The day in brief

K Jayatheerth committed to fixing `get_superproject_working_tree()` so the `git repo info` path series works under explicit repository contexts. Thomas Bachem revised his rerere lock patch so callers wait out the timeout and then fail rather than proceeding without rerere. Junio C Hamano said he will queue Jeff King's bump of the debian-11 CI job to debian-12.

## Notable threads

### Keeping background `rerere gc` from killing a rebase

Thomas Bachem's patch addresses a lock clash in the rerere code: `git rerere gc` holds `MERGE_RR.lock` while pruning `rr-cache`, and auto-maintenance can start one during a rebase, so the sequencer's `repo_rerere()` at the next conflict dies on the lock, leaving a rebase whose later continues refuse with "you have staged changes". Today Bachem conceded that his v3 approach of proceeding without recording the preimage after `rerere.lockTimeout` is "the wrong trade": in v4 every caller will wait the timeout and then fail as today, with only `git rerere gc` giving up at once. Junio C Hamano argued that proceeding without recording the preimage is a hard-to-accept regression and proposed instead loudly warning when `git rerere` fails to record a preimage, with instructions to run `git rerere` manually before resolving.

### Bumping the debian-11 CI job to debian-12

Jeff King proposed changing the debian-11 job to debian-12 in both `.github/workflows/main.yml` and `.gitlab-ci.yml`, since Debian 11 recently exited its LTS period; the job exists to cover older LTS releases, and debian-12 maintains that coverage for about two more years. He noted an apt failure on a debian-11 job that day, and that he tested the GitHub job but not the GitLab one. Junio C Hamano replied that he had also noticed the failing job and will queue the patch.

### Participating in Outreachy's December 2026 cohort

Christian Couder announced that, with no objections, he will submit a Community Application for Git for Outreachy's December 2026 cohort, thanked Usman, Kaartic and Pablo for volunteering as co-mentors, and proposed two intern project ideas: continuing to remove global state and perhaps libifying parts of the code base, and improving how command arguments and options are scanned by porting hand-rolled code to the parse-options API or his early-scan API. Mentors still need to sign up and projects must be submitted before the deadline less than one week away; he will also approach GitLab and other companies about sponsoring an intern.

## In brief

- Ben Knoble reviewed Tyler Cipriani's fix for `--force-if-includes` consulting the wrong ref, agreed the detached-HEAD rejection is preserved rather than changed, and said "this may be all we can do for now" since no suitable reflog exists.
- Aleksei Sviridkin posted v3 of a fix for `--force-if-includes` when the remote-tracking ref has no reflog, initializing the timestamp to zero so the reflog walk is not cut off, with a new test in `t/t5533-push-cas.sh`.
- Kristoffer Haugsbakk posted v3 of a documentation series advertising `gitdatamodel(7)`: listing it as a concept guide, linking it from `git(1)`, linking four glossary terms to it, and linking back to the glossary.
- Aleksei Sviridkin posted v3 documenting that `cherry-pick --no-commit` skips `CHERRY_PICK_HEAD` by design, restoring a `test_ref_missing` check in the conflicting-pick test.
- Junio C Hamano objected to mechanically converting `test -f` to `test_path_is_file` inside `if` conditionals in Mark C. Chu-Carroll's test cleanup, since the semantic helpers are meant to shout on failure, and declined to review the rest.
