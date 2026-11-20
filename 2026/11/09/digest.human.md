# Git mailing list daily digest for 2026/11/09

## Long threads

### Path keys for `git repo info`

*Long thread: 55 replies so far from 6 participants over 117 days; 2 messages that day.*

The GSoC series adds path keys to `git repo info` (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.git-prefix`, `path.cdup`) and refactors `get_superproject_working_tree()` onto a `struct repository *`. After Phillip Wood earlier suggested renames, K Jayatheerth today agreed "superproject-working-tree" is the better superproject key name, agreed to change `path.git-prefix`, and defended keeping `path.hooks` because scripts still query it via `git rev-parse --git-path hooks`. He also contrasted `path.cdup`'s slash-counting `../` output with relative path formatting. Junio C Hamano noted Git generally avoids `-` in configuration variable names.

### Community voting process

*Long thread: 31 replies so far from 14 participants over 35 days; 1 message that day.*

Following Contributor's Summit notes and earlier debate over voter eligibility and platforms, Stéphane Glondu today proposed Belenios as the voting platform: it supports Condorcet voting, an instance already runs at vote.debian.net for anyone with a Debian Salsa account, and a new instance could be deployed elsewhere.

### Worktrees rebasing or bisecting a branch

*Long thread: 26 replies so far from 8 participants over 20 days; 4 messages that day.*

Jasper Mayone's v4 series makes `git worktree remove` refuse a worktree mid-operation without `--force`, documents reading `%(worktreepath)` safely, makes `%(worktreepath)` name the worktree rebasing or bisecting a branch, and stops `git branch` marking it with `+`. Today D. Ben Knoble questioned the claim that only one worktree is shown for a multiply-checked-out branch; Phillip Wood answered this is possible via `git worktree add --force`, `git checkout --ignore-other-worktrees`, or `git symbolic-ref`, and Ben Knoble retracted his impression as faulty memory. Junio C Hamano objected to the "possible races" rationale: with `main` checked out in two worktrees, a commit in one makes `git status` in the other report the change as reverted.

### `test-tool genbytes` for slow `tr(1)` on macOS

*Long thread: 23 replies so far from 4 participants over 3 days; 8 messages that day.*

Kristofer Karlsson's v2 adds `test-tool genbytes <hex-byte> [<count>]` to replace slow `genzeros | tr` calls in tests such as t4205 on macOS, and removes `genzeros`. Today Junio C Hamano suggested a `static` buffer to avoid the 256kB stack allocation and said a "not thread-safe" comment is unwarranted for throw-away test tooling. Patrick Steinhardt proposed `git_parse_signed()` for the count; Kristofer Karlsson weighed buffer options and leaned toward keeping `genzeros` plus a plain-character `genbytes`. Johannes Sixt reported a `printf` alternative passes CI with caveats.

### Detached HEAD documentation rewrite

*Long thread: 23 replies so far from 4 participants over 34 days; 1 message that day.*

Julia Evans's v4 rewrites the detached HEAD explanation as a standalone `gitdetached-head` guide wired into `advice.c` and `command-list.txt`. After earlier debate over describing HEAD in users' own words, Julia Evans today proposed text explaining what the two HEAD states share -- `HEAD` always refers to the current commit, either as a branch or as a commit ID -- and pushed back on the objection to "useful for `HEAD` to be a commit ID," asking whether it misleads users.

## The day in brief

The `git repo info` path-keys series incorporated rename feedback, with one key name defended on script-compatibility grounds. A 15-patch series fixing segfaults on valueless settings drew positive reviews with one behavioral tweak agreed for v2. The fsmonitor untracked-cache fix was re-queued on its original base after a `next` merge was reverted.

## Notable threads

### Segfaults on valueless settings
Thomas Bachem's 15-patch series fixes segfaults when settings such as `rebase.missingCommitsCheck` or `fetch.bundleCreationToken` are given without a value. Today Derrick Stolee reviewed several patches positively, Phillip Wood suggested a missing `rebase.missingCommitsCheck` value warn and fall back to "ignore" like an invalid value, Junio C Hamano agreed, and Thomas Bachem confirmed v2 will change only that patch.

### fsmonitor untracked cache after a trivial response
Ilia K's patch stops the untracked cache being saved with stale valid bits after the fsmonitor daemon sends a trivial response. Today Ilia K posted v2, invalidating the whole cache when the worktree cannot be determined; Derrick Stolee said it LGTM, and Junio C Hamano reverted the merge to `next`, queued v2 on the original base, and said he will mark the topic ready for `next` again after a few days.

### Default hash algorithm in `setup.c`
brian m. carlsson's patch defaults the hash algorithm to `GIT_HASH_DEFAULT` (SHA-256 in breaking-changes mode) instead of SHA-1. Today Junio C Hamano diagnosed a regression and a latent bug in `repository_format_configure()` where `init.defaultObjectFormat` can corrupt an existing repository on re-init, and Patrick Steinhardt confirmed `git init` does not migrate formats, proposing the hash block be restructured like the ref-storage checks.

## In brief
- 井上司 corrected his reftable HEAD reflog report: the files backend behaves the same, and he proposed a `refs.c` helper returning the new oid of a ref updated in the same transaction.
- Miklos Vajna posted v5 speeding up `log --follow` on non-linear history, now also following copies and renames across parents with no merge base.
- Tanga Joris and reviewers agreed `git apply` should reject corrupt `diff --git` headers with an error instead of silently applying them.
- Junio C Hamano posted v2 of the `unpack-trees.c` the_repository removal and explained the paranoid repository-match checks in `unpack_repo()`.
- Reviewers agreed the `git diff` EXAMPLES rewrite should use anchors working in both Asciidoctor and AsciiDoc; Junio C Hamano will queue squash fixes.
