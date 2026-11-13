# Git mailing list daily digest for 2026/11/12

## The day in brief
The Git mailing list saw significant activity around performance optimizations, bug fixes, and documentation improvements. Key developments include a project-level decision to make `git log --follow` opt-in for merge commits, multiple bugfix series addressing segfaults and protocol issues, and continued progress on the `the_repository` removal effort. Documentation updates and test modernizations also featured prominently.

## Notable threads

### `git log --follow` performance regression resolved with opt-in toggle
**Thread root: 2026/09/19/14-47-26**

The long-running discussion about the performance regression in `git log --follow` (14× slower after commit 304812ed33) reached a decisive conclusion. Junio C Hamano ejected the prior speedup branch and queued a single-patch replacement that introduces an opt-in toggle (`--follow=merges` or `log.follow=merges`) for the expensive per-parent diffing logic. This pragmatic solution preserves the correctness fixes while mitigating the performance cost for most users.

The toggle will be disabled by default, addressing GitLab's deployment constraints while allowing users who need the thorough logic to opt in. The decision reflects a balance between performance, correctness, and operational requirements, with the maintainer framing the toggle as "sensible" and "far better" than reverting the original commit or accepting the performance cost as permanent.

**What changed?**
- The expensive rename detection across merges is now opt-in via `--follow=merges` or `log.follow=merges`
- Plain `--follow` restores the pre-304812ed33 performance
- The toggle is disabled by default, preserving the correctness fixes while addressing the performance regression

**Impact:**
This change directly addresses a severe performance regression affecting large repositories (e.g., the Linux kernel) and unblocks GitLab's deployment. The opt-in design allows users to choose between speed and thoroughness, making the trade-off explicit.

---

### Bugfix series eliminates segfaults from unvalued config keys
**Thread root: 2026/11/07/08-42-32**

A 15-patch bugfix series from Thomas Bachem systematically eliminates segfaults triggered by unvalued config keys (e.g., `git -c foo.bar` or bare `bar` under `[foo]`). The series addresses 23 distinct crash sites across Git commands, including `git rebase -i`, `git fetch`, `git add -p`, and `git config list --type=color/expiry-date`. One patch fixes a regression introduced in Git 2.54.0.

The v3 update harmonizes error handling for `remote.*.tagOpt` and `color.blame.*` settings, treating missing values the same as invalid ones (warn and ignore). The series is now queued for `next` after receiving final review approval from Phillip Wood and Derrick Stolee. The only remaining blocker is resolving test conflicts with two topics in `seen`.

**What changed?**
- Unvalued config keys now consistently trigger `config_error_nonbool()`, warnings, or graceful handling
- Specific fixes for `safe.bareRepository`, `bitmapPseudoMerge.<name>.pattern`, `scalar.repo`, and `includeIf.hasconfig:remote.*.url`
- Harmonized error handling for `remote.*.tagOpt` and `color.blame.*` (missing values now warn and are ignored)

**Impact:**
This series fixes a widespread class of crashes affecting multiple Git commands, improving robustness and user experience. The fixes are particularly important for partial clones and large repositories where config-related crashes can disrupt workflows.

---

### Promisor-remote protocol v2 capability fix
**Thread root: 2026/11/12/19-58-59**

Christian Couder identified and fixed a long-standing bug in the promisor-remote protocol v2 capability, where the Git client omits the `promisor-remote=...` reply during `fetch` commands. This omission breaks interactions with servers that handle each request independently (e.g., Git over HTTP or JGit/Gerrit), causing fetch failures when objects are only available on promisor remotes.

The three-patch series:
1. Prepares the groundwork by allowing `write_command_and_capabilities()` to send a promisor-remote reply
2. Ensures the client sends the reply during `fetch` protocol v2 commands
3. Updates documentation to clarify the expected behavior

Junio C Hamano raised architectural concerns about the design of patch 1/3, suggesting that the `promisor-remote` capability should be handled via the existing `server_options` string list rather than a dedicated parameter. This feedback may require a v2 of the series.

**What changed?**
- The client now sends `promisor-remote=...` during `fetch` protocol v2 commands
- The fix addresses a bug that has existed since the capability was introduced in Git v2.49
- New tests cover both local and HTTP scenarios

**Impact:**
This fix enables reliable promisor-remote functionality over HTTP and other stateless transports, improving support for partial clones and large repositories that rely on promisor remotes for scalability.

---

### Documentation and terminology cleanup for root refs vs. pseudorefs
**Thread root: 2026/10/22/21-28-48**

Andy Koppe's five-patch series aligns Git's documentation, code comments, and tests with the 2024 terminology change that narrowed "pseudoref" to `FETCH_HEAD` and `MERGE_HEAD` only, while introducing "root ref" for refs outside the `refs/` hierarchy (e.g., `HEAD`, `ORIG_HEAD`). The v4 iteration addresses all remaining editorial feedback and is now ready for merging.

The series updates 38 files, including the glossary, man pages, test scripts, and code comments. Key changes include:
- A new "root ref" glossary entry with an exhaustive list of irregular root refs
- Clarification that pseudorefs are not handled by the ref backend and can record multiple object IDs
- Consistent terminology updates across documentation and error messages

Junio C Hamano's final sign-off confirms the series is ready for `master`.

**What changed?**
- Documentation now consistently uses "root ref" for refs outside `refs/` and reserves "pseudoref" for `FETCH_HEAD`/`MERGE_HEAD`
- The glossary provides clear, precise definitions and behavioral distinctions
- Test scripts and code comments are updated to reflect the new terminology

**Impact:**
This cleanup resolves long-standing inconsistencies in Git's documentation and provides clearer guidance for users and contributors. The improved terminology will help avoid confusion in future development and discussions.

---

### `git rerere` subcommand handling modernized
**Thread root: 2026/10/27/10-46-43**

Patrick Steinhardt's 12-patch series replaces the custom subcommand-parsing logic in `git rerere` with Git's standard `OPT_SUBCOMMAND()` mechanism, bringing the command in line with other multi-subcommand built-ins like `git notes` and `git maintenance`. The series is purely internal restructuring with no behavior changes.

The v4 update adds a `BUG()` check in `rerere_cache_setup()` to catch the invalid state where both `RERERE_AUTOUPDATE` and `RERERE_NOAUTOUPDATE` flags are set simultaneously. The final patch adds per-subcommand usage strings to the man page and C code.

**What changed?**
- `git rerere` now uses `OPT_SUBCOMMAND()` for subcommand parsing
- Dedicated functions for each subcommand (`cmd_rerere_clear()`, `cmd_rerere_status()`, etc.)
- Per-subcommand usage strings in the man page and C code
- A `BUG()` check to catch invalid flag combinations

**Impact:**
This refactoring improves code consistency and maintainability without changing user-visible behavior. The modernization aligns `git rerere` with Git's established patterns for multi-subcommand built-ins.

---

### Test modernization for `t9300-fast-import.sh`
**Thread root: 2026/11/10/11-49-45**

Karthik Nayak's 20-patch series refactors the monolithic `t9300-fast-import.sh` (4000+ lines) into smaller, self-contained test files. The series systematically makes test groups independent of prior test state, then moves them to dedicated files like `t9301-fast-import-notes.sh` and `t9304-fast-import-marks.sh`.

The v2 iteration incorporates feedback from Junio C Hamano and resolves conflicts with two in-flight topics. The series has received strong maintainer support, with Junio calling the cleanup "long overdue."

**What changed?**
- `t9300-fast-import.sh` reduced from ~4000 to ~1200 lines
- New test files created for specific functionality (notes, marks, stream handling, etc.)
- Each test now runs in a fresh repository with consistent configuration
- Legacy "series letter" prefixes removed from test descriptions

**Impact:**
This refactoring significantly improves the maintainability of `git fast-import` tests, making them easier to debug, extend, and parallelize. The changes are purely organizational and do not affect Git's behavior or on-disk format.

---

## In brief

- **[2026/09/02/08-31-37]** Junio C Hamano questioned the fundamental rationale behind renaming `credentialStore.lockTimeoutMS` to `credentialStore.lockTimeout`, sparking a discussion about Git's timeout variable naming conventions.
- **[2026/10/07/14-52-01]** Julia Evans posted the final version (v6) of the detached HEAD documentation rewrite, addressing minor style nits and a missing test expectation.
- **[2026/10/20/10-21-34]** Andy Koppe's v4 series aligning "pseudoref" vs. "root ref" terminology is ready for merging after addressing all editorial feedback.
- **[2026/11/05/10-45-41]** Junio C Hamano confirmed the safety of `transport_get_hash_algo()` for the `--show-object-format` feature, endorsing the test coverage as adequate.
- **[2026/11/08/12-50-55]** Karthik Nayak provided substantive review feedback on the reflog symref transaction bugfix, suggesting optimizations and additional test scenarios.
- **[2026/11/12/05-57-35]** Yusuke Hayashi removed an unused `peel_iterated_oid()` declaration from `refs.h`, a mechanical cleanup following earlier ref backend work.
- **[2026/11/12/06-33-08]** Srujith A Poojary refactored the archive subsystem to accept explicit `struct repository *` parameters, advancing the `the_repository` removal effort.
- **[2026/11/12/09-05-07]** Miklos Vajna introduced `--follow=merges` to make the expensive rename detection in `git log --follow` opt-in, addressing a severe performance regression.
- **[2026/11/12/09-48-41]** Mustafa Bayar fixed a double-free crash in `git for-each-ref` when processing refs pointing to missing objects, a regression introduced in v2.53.0.
- **[2026/11/12/12-03-02]** Yoichi NAKAYAMA introduced worktree-specific submodule configuration, allowing submodule settings to vary between worktrees when `extensions.worktreeConfig` is enabled.
- **[2026/11/12/13-39-58]** Junio C Hamano confirmed a bug in `git diff`'s context output for shell scripts with multi-line function definitions, explaining the root cause and subsystem to modify.
- **[2026/11/12/15-39-01]** Jakub Kulik fixed unquoted shell variables in `t1017-cat-file-remote-object-info.sh` for Solaris compatibility, addressing test failures when paths contain spaces.
- **[2026/11/12/15-52-49]** Jakub Kulik corrected a `uname()` error-checking bug in `version.c` for Solaris, aligning the check with the POSIX.1 standard.
- **[2026/11/12/16-14-04]** Kristofer Karlsson reduced `struct commit` size by 8 bytes on 64-bit platforms by reordering fields, sparking a discussion about struct bloat awareness and code clarity.
- **[2026/11/12/17-34-01]** Karthik Nayak's 20-patch test modernization series for `t9300-fast-import.sh` continues to make progress, with v2 incorporating feedback and resolving conflicts.
- **[2026/11/12/19-31-57]** A GSoC student inquired about the availability of a specific test suite fix in `t/t4150-am.sh` for the "Avoid suppressing git's exit code in test scripts" microproject.
- **[2026/11/12/19-57-37]** Adrian Ratiu fixed three bugs in Git's object-format interoperability layer, including a crash when `compatObjectFormat` and `objectFormat` are set to the same value.
- **[2026/11/12/23-45-18]** Junio C Hamano's "What's cooking in git.git" report summarized the current state of integration branches, highlighting graduated topics, new submissions, and ongoing efforts.