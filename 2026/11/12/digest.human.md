# Git mailing list daily digest for 2026/11/12

## The day in brief
The Git project made significant progress on several fronts today. A long-standing performance regression in `git log --follow` was addressed with an opt-in toggle, the `rerere` subsystem was modernized with proper subcommand handling, and multiple bugfix series advanced toward integration. Documentation improvements and test modernizations continued, while architectural discussions about refspec handling and object format interoperability revealed deeper design considerations.

## Notable threads

### `git log --follow` performance regression resolved with opt-in toggle
**What changed**: Miklos Vajna introduced a patch that makes the expensive rename detection in `git log --follow` across merges opt-in via `--follow=merges` or `log.follow=merges`, restoring pre-regression performance while preserving the improved correctness for users who need it.

**Problem/goal**: A June 2026 commit made `git log --follow` ~14× slower on non-linear history by diffing each merge against all parents with rename detection enabled, creating a severe performance regression for common workflows like `git log --stat --follow` on single files in large repositories.

**Subsystem**: revision walking, diff machinery

**Kind of change**: performance bugfix with feature addition

**Impact**: Users can now choose between fast operation (default) and thorough rename detection (opt-in), addressing a deployment blocker for GitLab while preserving correctness improvements.

**Today's developments**: Junio C Hamano ejected a prior speedup branch and queued this toggle patch as the project-level resolution, settling the policy question after extensive discussion about performance vs. correctness trade-offs.

---

### `git rerere` subcommand handling modernized
**What changed**: Patrick Steinhardt completed a 12-patch series that replaces `git rerere`'s custom subcommand-parsing logic with Git's standard `OPT_SUBCOMMAND()` mechanism, bringing it in line with other multi-subcommand built-ins.

**Problem/goal**: The `rerere` subsystem used hand-rolled subcommand parsing that was inconsistent with other Git commands and harder to maintain. The series was purely internal restructuring with no behavior changes.

**Subsystem**: `rerere` (reuse recorded resolution)

**Kind of change**: refactoring

**Impact**: Improves code maintainability and consistency across Git's command-line interface. The series also added a `BUG()` check to catch invalid flag combinations and per-subcommand usage strings.

**Today's developments**: Karthik Nayak provided final review approval, confirming the series is ready for `next`. The patch set demonstrates how Git is systematically modernizing its internal APIs while preserving backward compatibility.

---

### Config-related segfaults systematically fixed
**What changed**: Thomas Bachem's 15-patch series eliminates Git segfaults when config settings are given without values (e.g., `git -c foo.bar` or bare `bar` under `[foo]`), addressing 23 distinct crash sites across various commands.

**Problem/goal**: Unvalued config keys could trigger NULL pointer dereferences in config callbacks, causing crashes in commands like `git rebase -i`, `git fetch`, `git add -p`, and others. The series audits config callbacks and `repo_config_get_value()` callers to handle missing values gracefully.

**Subsystem**: config system, multiple commands

**Kind of change**: bugfix

**Impact**: Prevents crashes in common workflows and improves Git's robustness. The series also fixes a regression introduced in Git 2.54.0 affecting `git config list --type=color/expiry-date`.

**Today's developments**: Junio C Hamano queued the series for `next` after Derrick Stolee and Phillip Wood confirmed all feedback was addressed. The only remaining blocker is resolving test conflicts with two topics in `seen`.

---

### Refactor of `t9300-fast-import.sh` advances
**What changed**: Karthik Nayak posted v2 of a 20-patch series that splits the monolithic `t9300-fast-import.sh` (4000+ lines) into smaller, self-contained test files grouped by functionality.

**Problem/goal**: The original test file had tests that depended on side effects from earlier tests, making it difficult to maintain and extend. The series makes each test run in a fresh repository with consistent configuration.

**Subsystem**: test suite, `git fast-import`

**Kind of change**: test modernization

**Impact**: Significantly improves test maintainability and reduces the risk of regressions in the `fast-import` subsystem. The series demonstrates Git's ongoing effort to modernize its test infrastructure.

**Today's developments**: The v2 series incorporated feedback from Junio C Hamano and resolved conflicts with two in-flight topics. The patches systematically make test groups independent before moving them to dedicated files, with strong maintainer support.

---

### Object format interoperability bugs addressed
**What changed**: Adrian Ratiu posted a 3-patch series fixing bugs in Git's `compatObjectFormat` interoperability layer, which handles conversions between SHA-1 and SHA-256 repositories.

**Problem/goal**: Three distinct issues were addressed: (1) a crash when `compatObjectFormat` and `objectFormat` are set to the same value, (2) silent incorrect OID return from `git rev-parse --output-object-format` when mapping fails, and (3) inconsistent handling of unknown commit headers during conversion.

**Subsystem**: object format interoperability

**Kind of change**: bugfix

**Impact**: Improves reliability of mixed-format repositories and fixes edge cases that could leave repositories in unusable states. The third patch is particularly interesting as it reverses a 2023 safety check to improve compatibility with external tools like Jujutsu.

**Today's developments**: The series was posted for review, with the third patch expected to generate discussion about the trade-off between consistency with external tools and safety of rejecting unrecognized headers.

---

### Promisor-remote protocol v2 capability fixed
**What changed**: Christian Couder posted a 3-patch series fixing a bug where the Git client omits the `promisor-remote=...` reply during `fetch` protocol v2 commands, breaking interactions with servers that handle each request independently.

**Problem/goal**: The bug has existed since the capability was introduced in Git v2.49 and causes fetch failures when objects are only available on promisor remotes, particularly affecting Git over HTTP and JGit/Gerrit.

**Subsystem**: protocol v2, promisor remotes

**Kind of change**: bugfix

**Impact**: Enables reliable partial clone operations with servers that don't maintain state between requests. The fix is explicitly requested for the `maint` branch, indicating its urgency.

**Today's developments**: Junio C Hamano questioned the design choice of adding a dedicated parameter to `write_command_and_capabilities()`, suggesting instead to generalize the existing `server_options` mechanism. This architectural discussion highlights the tension between targeted fixes and broader API design.

---

## In brief

- **`git ls-remote --show-object-format`**: Junio C Hamano confirmed the safety of the implementation and endorsed the test coverage as adequate, moving this feature closer to integration. The patch enables tools like Jujutsu to detect a remote repository's hash algorithm without cloning.

- **GitLab CI speedups**: Johannes Sixt reported successful testing of Patrick Steinhardt's CI optimization series on Windows hardware, confirming significant runtime reductions (32% for `t0027`, 11% for `t1092`).

- **Detached HEAD documentation**: Julia Evans posted v6 of the documentation rewrite, addressing final style nits and test expectations. The series creates a standalone guide that explains detached HEAD in plain language.

- **Glossary terminology alignment**: Andy Koppe's v4 series addressing the 2024 terminology change that narrowed "pseudoref" to `FETCH_HEAD` and `MERGE_HEAD` only received final sign-off from Junio C Hamano, completing this documentation cleanup.

- **Reflog symref transaction bug**: Karthik Nayak provided substantive review of Tsukasa Inoue's patch, suggesting implementation refinements and additional test scenarios for the reflog handling fix.

- **Worktree-specific submodule configuration**: Yoichi NAKAYAMA posted a 3-patch series introducing worktree-specific submodule configuration when `extensions.worktreeConfig` is enabled, addressing a gap in worktree support.

- **`git diff` shell function context**: Junio C Hamano confirmed a bug in `git diff`'s context header logic for shell scripts where function definitions span multiple lines, identifying the userdiff pattern as the subsystem to modify.

- **Solaris compatibility fixes**: Two platform-specific fixes were posted: one for unquoted shell variables in `t1017-cat-file-remote-object-info.sh` and another for `uname()` error-checking in `version.c`.

- **`struct commit` memory optimization**: Kristofer Karlsson's patch to reduce `struct commit` size by 10% on 64-bit platforms sparked discussion about struct bloat awareness and code clarity, with Junio C Hamano suggesting additional comments to clarify field purposes.

- **GSoC microproject inquiry**: A student inquired about the availability of a specific test suite fix in `t4150-am.sh` for the "Avoid suppressing git's exit code in test scripts" microproject.

The day's activity demonstrates Git's ongoing evolution across multiple dimensions: performance improvements, architectural modernization, platform compatibility, and documentation clarity. The project continues to balance the need for backward compatibility with the desire to address long-standing issues and improve the developer experience.