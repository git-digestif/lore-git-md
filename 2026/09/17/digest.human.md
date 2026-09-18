# Git mailing list daily digest for 2026/09/17

## The day in brief

The Git project saw significant progress on several fronts today. A long-running series to defer auto maintenance during sequencer operations reached its fifth iteration, while a critical bugfix for `--force-if-includes` in `git push` was refined in its sixth version. A new bug in `git rerere remaining` was reported, and a seven-patch hardening series addressing Coverity-reported issues in Git for Windows was posted. The `advice: add scope hint` series also saw movement toward consensus.

## Notable threads

### Sequencer: defer auto maintenance until rebase completion (v5 posted)

Thomas Bachem posted the fifth iteration of a three-patch series adjusting when auto maintenance runs during sequencer-driven operations (`git rebase`, `git cherry-pick`, `git revert`). The series defers auto maintenance until sequence completion, matching the apply backend’s behavior and preventing maintenance from interfering with intermediate steps.

The v5 iteration is code-identical to v4; only commit messages, header comments, and test annotations have been updated to address reviewer feedback. The core design remains unchanged: auto maintenance is disabled for individual commands (`git commit`, `git merge`, exec commands) during a sequencer-driven operation by passing `maintenance.auto=false` via `GIT_CONFIG_PARAMETERS`, and `git maintenance run --auto` is invoked exactly once at sequence completion. The implementation delegates this to the built-in commands (`builtin/rebase.c` and `builtin/revert.c`) rather than the sequencer itself, a choice that was previously contested but is now settled and documented.

The series touches `sequencer.c`, `config.c`, `builtin/rebase.c`, and `builtin/revert.c`, and includes expanded test coverage in `t/t3418-rebase-continue.sh` and `t/t3510-cherry-pick-sequence.sh`. Junio C Hamano previously signaled intent to queue the series ("Will queue"), and it is now in `seen`. Unless new objections arise, this version is likely to proceed to `next`.

### Fix --force-if-includes to check the correct reflog (v6 posted)

Tyler Cipriani posted the sixth iteration of a three-patch series fixing critical flaws in the `--force-if-includes` safety mechanism in `git push`. The series addresses two issues: (1) the feature incorrectly checks the reflog of the *local* branch named after the *remote* destination rather than the branch actually being pushed, and (2) it unnecessarily rejects fast-forward pushes of non-branch refs (tags, detached HEADs) even when no force is needed.

The v6 reroll renames the `deferred_reject_reason` variable to `needs_force_reject_reason` for clarity and fixes commit-message typos. The core logic in `remote.c` now consults the reflog of the pushed ref (`ref->peer_ref->name`) and defers reflog checks until after fast-forward determination, ensuring fast-forward pushes are exempt from `--force-if-includes` checks. This resolves an existing bug where workflows like `git push origin <TAG>:hotfix` were broken.

The series also introduces a new `ref->unverifiable` flag and `advice.forceIfIncludesDetachedHead` config knob to provide actionable advice for rejected non-branch pushes. It includes 108 lines of new test coverage in `t/t5533-push-cas.sh` and is based on `maint`. All prior review feedback from Patrick Steinhardt, Junio C Hamano, and D. Ben Knoble is resolved, and the series is ready for final review.

### Bug in `git rerere remaining` skipping consecutive conflicted paths

Mikko Rantalainen reported a bug in `git rerere remaining` where consecutive conflicted paths with only stage-1 index entries are incorrectly skipped. The issue manifests in `git mergetool`, which exits successfully after resolving only the first of multiple unresolved conflicts, leaving the rest untouched.

The root cause is traced to `check_one_conflict()` in `rerere.c`, where a loop intended to skip multiple stage-1 entries for the same pathname also skips entries for *subsequent* pathnames. The author provides a minimal reproducer script that sets up a rebase with two files renamed differently in two branches, producing consecutive stage-1-only conflicts. On affected versions (tested on Git 2.43.0), `git rerere remaining` lists only the first conflicted path, while `git diff --name-only --diff-filter=U` correctly lists both.

The proposed fix involves modifying the loop in `check_one_conflict()` to include a same-path check (e.g., using `ce_same_name()`). The bug affects workflows involving large rebases with many conflicts, particularly when file renames are involved, and has a straightforward workaround (using `git mergetool -- .`). The code in `rerere.c` has been stable for years, so the issue likely persists in the latest Git versions.

### Coverity-reported issues in Git for Windows (seven-patch series posted)

Johannes Schindelin posted a seven-patch series addressing Coverity-reported static-analysis warnings that surfaced in Git for Windows after merging v2.56.0-rc0. The issues span potential signed-overflow in writev, missing input validation in GPG signature parsing, MIDX pack-ID checks, rerere conflict resolution edge cases, reftable iterator initialization, fuzz-testing harnesses, and a latent crash in the `test-read-midx` helper.

The series is purely defensive programming with no intended behavior changes. Key patches include:
- A fix for potential signed overflow in `writev_in_full()` using `signed_add_overflows()` and returning `EOVERFLOW` on detection.
- Replacing `starts_with()` with `starts_with_mem()` in `get_format_by_sig()` to prevent buffer overreads in GPG signature parsing.
- Adding bounds checks on local pack IDs in MIDX to prevent underflow or invalid memory access.
- Adding error handling around `handle_file()` and `copy_file()` in `do_rerere_one_path()` to prevent recording failed conflict resolutions.
- Plugging NULL-dereference bugs in reftable unit tests and the oss-fuzz harness.

Junio C Hamano reviewed the GPG signature-prefix matching patch and suggested an alternative design: confining the length-aware check to `get_format_by_sig()` without propagating a new parameter through `check_signature()`. The series is queued as GitGitGadget PR-2231 and is likely to be merged as-is once review completes.

### Advice: add scope hint for disabling advice messages (v5 posted)

Junio C Hamano posted the fifth iteration of a patch adding a scope hint to Git’s advice settings. The patch implements the now-consensus design: always recommending `--global` in the hint for *all* `advice.*` settings. It touches `advice.c` and updates twelve test scripts to expect the new `--global` wording.

The series has converged on a uniform `--global` hint for all `advice.*` settings, eliminating per-setting scope tracking entirely after Junio and Jeff King explicitly endorsed this simplification. Vsevolod Myalitsyn, the original author, reviewed v5 and endorsed the uniform `--global` approach, while reiterating his suggestion to refactor the `vadvise()` interface in a follow-up patch. Jeff King also endorsed the core logic in v4 and supported deferring the `vadvise()` interface cleanup to a follow-up patch.

The patch is minimal and mechanical, and the central design debate is resolved. No evidence of Junio picking it up for integration yet, but the series is likely to proceed to `next` soon.

## In brief

- **Stash.index=true + --autostash bugfix**: D. Ben Knoble implemented Phillip Wood’s proposed refactoring to replace subprocess-based index merge with a direct call to `merge_incore_nonrecursive()`, resolving a bug where `git merge --ff-only --autostash` left a redundant stash entry and failed to clean up the `MERGE_AUTOSTASH` ref. Phillip Wood reviewed the patch and suggested minor polish, including silencing merge verbosity and using `oidcpy()` for robustness.

- **Git v2.56.0-rc1 release notes**: Tuomas Ahola suggested a minor wording improvement to the release notes entry about `git history` segfaulting in corrupt repositories, proposing clearer phrasing to indicate the issue has been corrected. The suggestion is purely editorial and does not affect the code.

- **What's cooking in git.git**: SZEDER Gábor confirmed that the `sg/precompile-git-compat-util` topic, which precompiles `git-compat-util.h` to speed up builds, works with Clang and the Intel oneAPI compiler, removing the last portability blocker. Junio C Hamano acknowledged the confirmation and cleared the topic for graduation to `next`.

- **OpenBSD RUNTIME_PREFIX patch**: Junio C Hamano requested commit-message improvements for an OpenBSD `RUNTIME_PREFIX` patch, clarifying `getexecpath()`’s advantages over sysctl and its version availability. The patch is not yet ready for integration.