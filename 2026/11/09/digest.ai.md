# Git mailing list daily digest for 2026/11/09

## Long threads

### Path keys for `git repo info`

*Long thread (root 2026/07/16/01-21-31): 55 replies so far from 6 participants over 117 days; 2 messages that day.*

K Jayatheerth's GSoC series adds path keys to `git repo info` -- `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.git-prefix`, and `path.cdup`, in absolute and relative forms -- and refactors `get_superproject_working_tree()` to operate on a `struct repository *`. Earlier, Phillip Wood suggested renaming several keys (`path.working-tree.*`, `path.superproject-working-tree.*`, `path.prefix`) and questioned whether `path.hooks` is needed. Today K Jayatheerth agreed that "superproject-working-tree" is the more suitable name for the superproject key, with the same logic applying to toplevel, and agreed to change `path.git-prefix` in the next series. He argued `path.hooks` should remain because existing scripts and third-party tools still query the hooks directory via `git rev-parse --git-path hooks`. He also explained that `path.cdup` differs from relative path formatting: it counts slashes in the prefix to emit `../` sequences, while relative formatting uses Git's path utility library to compute true relative paths. Junio C Hamano noted that Git generally avoids '-' in configuration variable names, and K Jayatheerth said he would look into optimizing the tests to avoid initializing a new repository for each path test.

### Community voting process documentation

*Long thread (root 2026/10/06/18-05-36): 31 replies so far from 14 participants over 35 days; 1 message that day.*

This thread continues from the Contributor's Summit notes toward a documented community voting process for Git. Earlier discussion covered eligibility thresholds (commit counts intersected with recent activity), abstentions, secret votes, and platform choice, with Emily Shaffer planning a reroll. Today Stéphane Glondu proposed Belenios as the voting platform, noting that it supports Condorcet voting, that an instance already runs at vote.debian.net accessible to anyone with a Debian Salsa account, and that a new instance could be deployed elsewhere if needed.

### Worktrees rebasing or bisecting a branch treated as checked out

*Long thread (root 2026/10/21/17-33-36): 26 replies so far from 8 participants over 20 days; 4 messages that day.*

Jasper Mayone's v4 series makes `git worktree remove` refuse a worktree in the middle of an operation (rebase, am, bisect, cherry-pick/revert) without `--force`, documents how to read `%(worktreepath)` safely, makes `%(worktreepath)` name the worktree rebasing or bisecting a branch, and stops `git branch` from marking such a branch with "+". Today D. Ben Knoble questioned the claim that only one worktree is shown when a branch is checked out in more than one worktree. Phillip Wood answered that this is possible via `git worktree add --force`, `git checkout --ignore-other-worktrees`, or by bypassing the checks with `git symbolic-ref`, and Ben Knoble retracted his earlier impression as faulty memory. Junio C Hamano objected to "possible races" as the explanation for discouraging multiple checkouts: if a commit is made in one worktree, `git status` in the other reports as if the change were reverted, which he called a grave confusion rather than a race.

### `test-tool genbytes` to avoid slow `tr(1)` on macOS

*Long thread (root 2026/11/07/14-59-22): 23 replies so far from 4 participants over 3 days; 8 messages that day.*

Kristofer Karlsson's series adds `test-tool genbytes` so tests stop piping `genzeros` through `tr(1)`, which dominates t4205's EXPENSIVE runtime on macOS. Today's exchange covered the 256kB buffer in `generate_bytes()` -- Junio C Hamano suggested declaring it `static`, and Kristofer laid out the options -- plus Patrick Steinhardt's suggestions to use `git_parse_signed()` for the count and to reconsider hex byte arguments for readability. Johannes Sixt's `printf` alternative was confirmed by Kristofer to pass on macOS CI, cutting t4205 from 800-900s to 170-280s, though Johannes rejected a chunked printf helper because shells without a printf builtin would spawn ~2000 processes, preferring the ~2.1GB memory cost instead; Junio said a "not thread-safe" comment on a static buffer is unwarranted for throw-away test tooling.

### Detached HEAD documentation rewrite

*Long thread (root 2026/10/07/14-52-01): 23 replies so far from 4 participants over 34 days; 1 message that day.*

Julia Evans's patch rewrites the detached HEAD explanation into a standalone `gitdetached-head` guide, wired into `advice.c`, `command-list.txt`, and the build files, addressing unclear motivation, accidental detachment, and confusing diagrams. Today she proposed rewording that explains what the two HEAD states have in common -- `HEAD` always refers to the current commit, either as a branch whose latest commit is current or as a commit ID -- and pushed back on Junio C Hamano's objection to the phrase "useful for `HEAD` to be a commit ID," calling it a simple, short way to express that HEAD directly refers to a commit and asking whether it is misleading to users.

## The day in brief
Junio C Hamano reverted the merge of Ilia K's fsmonitor untracked-cache fix to `next` and re-queued v2 on its original base, saying he will mark it ready for `next` again after a few days. The community voting process thread gained a concrete platform proposal in Belenios, and the `git repo info` path-keys series moved toward renamed keys after review feedback.

## Notable threads

### fsmonitor untracked cache after a trivial response
Ilia K's patch fixes the case where a trivial fsmonitor response leaves stale untracked-cache valid bits that later `git status` runs trust. Today Ilia posted v2, which drops the whole untracked cache when the worktree cannot be determined and removes the test comments; Derrick Stolee said the patch LGTM but advised against rebasing against ToT. Junio C Hamano endorsed that and reverted the merge to `next`, queued v2 applied on the same base as v1, and said he will mark the topic ready for `next` again after waiting a few days for further feedback.

### Default hash algorithm in `setup.c`
brian m. carlsson's patch makes repository setup default the hash algorithm to `GIT_HASH_DEFAULT` instead of leaving it unset, so breaking-changes mode and Git 3.0 choose SHA-256. Today Junio C Hamano diagnosed a regression plus a latent bug in `repository_format_configure()`: re-initializing with `init.defaultObjectFormat` in `~/.gitconfig` already corrupts an existing repository because the hash-algo cascade is not guarded by the version check. Patrick Steinhardt agreed the blocks are hard to read, said he prefers explicit migration over silent format changes, and proposed restructuring so the default applies only to new repositories.

### `git apply` on corrupt "diff --git" lines
Tanga Joris's patch addresses `git apply` keeping names from an unused "diff --git" line, which previously crashed with an assertion. Today Phillip Wood argued that a blank line after a "diff --git" line or an indented "new file mode" line is a corrupt patch that should be rejected with a better error, since the file mode is silently ignored today. Tanga agreed and planned the next version to die with an error naming the "diff --git" line when corrupt headers precede a patch, while dropping the empty-section acceptance; Junio C Hamano stated that nothing `git format-patch` or `git diff` would not produce should be accepted.

### `unpack-trees.c` without `the_repository`
Junio C Hamano's v2 series removes `USE_THE_REPOSITORY` from `unpack-trees.c` and consistently uses `struct index_state *istate`. Today Patrick Steinhardt called the `unpack_repo()` helper an okayish intermediate step that verifies repository instances match, Derrick Stolee called it a good implementation, and Junio posted v2, whose only changes are removals of two blank lines, explaining that the paranoid repository-match checks exist to detect missed cases where the structs point at different repositories.

### Segfaults on settings given without a value
Thomas Bachem's 15-patch series fixes segfaults when a config setting is given without a value, covering `rebase.missingCommitsCheck`, alternate-refs settings, color slots, bundle tokens, and others, each with a regression test. Today Derrick Stolee reviewed several patches positively, Phillip Wood suggested that a missing `rebase.missingCommitsCheck` value should warn and fall back to "ignore" like an invalid value, Junio C Hamano agreed, and Thomas confirmed v2 will change only patch 01/15 in that way.

## In brief
- 井上司 corrected his reftable HEAD reflog report: the files backend records the same wrong old oid when the symref update precedes the branch update, so he proposed a helper in `refs.c` returning the new oid of a ref updated in the same transaction; Karthik Nayak endorsed the approach.
- Miklos Vajna posted v5 of the `log --follow` speedup series, simplifying patch 5/5 to diff only against parents that still need renames followed, and adding rename and copy following for subtree merges of unrelated histories.
- **ps/grep-recurse-submodules-fix acked**: Toon Claes acked v2 as small and uncontroversial, and Junio C Hamano said he will mark the topic ready for `next`.

- Discussion of Patrick Steinhardt's loose-object stale-cache fix centered on where a NEEDSWORK comment about `OBJECT_INFO_QUICK` belongs; Jeff King cautioned that removing QUICK would require auditing all its users for the original performance problems.
