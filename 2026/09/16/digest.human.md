# Git mailing list daily digest for 2026/09/16

## The day in brief
The Git project queued a regression fix for `git rebase` with commit graphs and submodules, finalized the `http.sslVerifyStatus` feature for OCSP staple validation, and advanced discussions on several bugfixes, including a `--force-if-includes` logic flaw and a stash/merge autostash race. Git v2.56.0-rc1 was announced, marking the start of the stabilization period for the upcoming release.

## Notable threads

### Regression fix: `git rebase` with commit graphs and submodules
A regression in `git rebase` (and related operations like merge and fetch) that caused fatal errors when commit graphs were enabled and submodule pointer changes were involved is now resolved. The fix, authored by Orestis Floros, modifies `commit-reach.c` to propagate the `struct repository *` parameter through reachability functions, ensuring commits are parsed in the correct repository. This prevents cross-repository commit-graph lookups that previously triggered the "invalid commit position" error.

Today, Kristofer Karlsson confirmed the fix resolves the regression and noted it reduces `commit-reach.c`'s reliance on `the_repository` from 17 to 11 instances, aligning with the broader libification effort. Junio C Hamano called the patch "reasonable" and will queue it for integration. The fix includes a new test case in `t/t6437-submodule-merge.sh` to verify the regression is resolved.

### `http.sslVerifyStatus` feature graduates to `master`
The `http.sslVerifyStatus` feature, which enables OCSP staple validation for HTTPS connections, has graduated to `master`. The feature adds a boolean configuration option (default `false`) that causes Git to fail connections if the server does not provide a valid OCSP staple. This addresses a security gap for government and government-adjacent customers (e.g., US Department of Defense PKI) who mandate OCSP stapling.

Junio C Hamano confirmed the updated test structure—keeping non-OCSP cases in `t5551-http-fetch-smart.sh` and moving OCSP-specific tests to `t5585-http-ssl-ocsp.sh`—and will replace the previous version in `next` with the final combined patch. The feature is now technically complete, with all review feedback addressed and test coverage confirmed.

### `--force-if-includes` logic flaw and fast-forward exemption
A bugfix series for `--force-if-includes` in `git push` continues to advance, with the author, Tyler Cipriani, addressing feedback on the fast-forward exemption logic. The series fixes two critical flaws: (1) the feature incorrectly checked the reflog of the *local* branch matching the *remote* destination branch, and (2) it unnecessarily rejected fast-forward pushes of non-branch refs (tags, detached HEADs).

Today, D. Ben Knoble suggested the `deferred_reject_reason` variable name in `set_ref_status_for_push` could be clearer for future maintainability. Tyler acknowledged the name is "a little broad" and invited further feedback on a rename or comment. The series is technically complete, with 108 lines of new test coverage in `t/t5533-push-cas.sh`, and is poised for a v6 reroll or final approval.

### Stash/merge autostash race during fast-forward merges
A bug in Git 2.55.0 (and current master) causes `git merge --ff-only --autostash` to leave behind a redundant stash entry and fail to clean up the `MERGE_AUTOSTASH` ref when `stash.index=true` is set and a staged change exists at merge time. The operation succeeds but emits an error message, and the stash list contains an extra entry.

Today, Phillip Wood proposed two fixes: (1) a surgical fix replacing `reset_head()` with `reset_tree(&c_tree, 0, 1)` to avoid branch-state cleanup, and (2) a more ambitious refactoring using `merge_incore_nonrecursive()` for a three-way index merge. D. Ben Knoble committed to testing both approaches and may contribute implementation work. Eli Barzilay, the original reporter, offered to test patches as a non-developer observer, signaling end-user demand for a fix.

### Git v2.56.0-rc1 announced
Junio C Hamano announced the first release candidate of Git v2.56.0, marking the start of the stabilization period for the upcoming release. The release candidate includes 721 non-merge commits since v2.55.0, contributed by 90 people, 37 of whom are first-time contributors.

Key changes span UI/workflows (e.g., new `git refs` subcommands, `git history drop`), performance optimizations (e.g., ODB abstraction, reftable backend improvements), and development support (e.g., build system updates, CI infrastructure). The release also includes memory safety fixes, platform-specific fixes, and test suite improvements. The final release is expected soon, pending community testing.

## In brief
- **Repack race fix**: Qin ShiCheng agreed to withdraw patch 1/6 of the repack data-loss race series in favor of Justin Tobler’s ODB-focused series, which will handle `.keep` file ownership more cleanly. Patches 2/6–6/6 remain relevant and will proceed independently.
- **Git protocol v2 shallow fetch regression**: Royce Remer posted v2 of a bugfix for a client-side crash during shallow fetches when `uploadpack.allowRefInWanted=true` is enabled. The patch swaps the order of `send_shallow_info()` and `send_wanted_ref_info()` in `upload_pack_v2()` to match client expectations.
- **`GIT_WORK_TREE` export for worktree hooks**: Junio C Hamano proposed a patch to export `GIT_WORK_TREE` alongside `GIT_DIR` when a worktree is detected during repository setup. The fix addresses a bug where `git rev-parse --show-toplevel` returns the current working directory instead of the worktree root in `post-checkout` hooks.
- **Windows ANSI emulation bugfix**: Yongqiang Tian posted a patch fixing a formatting bug in `compat/winansi.c:die_lasterr()` that caused Windows API failure messages to display a `va_list` address instead of the intended integer handle. Johannes Sixt and René Scharfe proposed alternative approaches, sparking a discussion about diagnostic precision vs. code simplicity.
- **"What's cooking" report**: Junio C Hamano summarized the state of integration branches, noting 87 commits in `next` and 166 in `seen`. Graduated topics include Rust cross-compilation fixes, ODB pluggable fsck, and a new `receive-report` hook. Stalled topics include a Rust crate subdirectory move and a `diff.<driver>.process` feature.