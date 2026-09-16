# Git mailing list daily digest for 2026/09/15

## The day in brief
The Git mailing list saw a flurry of activity today, with key developments including a v5 bugfix series for `--force-if-includes` in `git push`, architectural discussions about ODB transaction races, and multiple patch series advancing toward integration. Memory leak fixes, documentation updates, and build system improvements also featured prominently, alongside new bug reports and feature patches.

## Notable threads

### `--force-if-includes` bugfix series reaches v5
**[2026/09/04/21-01-20]** Tyler Cipriani posted a three-patch v5 series fixing long-standing flaws in `git push --force-if-includes`. The series corrects the core logic to check the reflog of the pushed ref (`ref->peer_ref->name`) rather than the local branch matching the remote destination, and exempts fast-forward pushes from reflog checks entirely. This resolves an existing bug where fast-forwards of non-branch refs (tags, detached HEADs) were incorrectly rejected, addressing a backwards-compatibility concern raised by D. Ben Knoble in v4.

The implementation is technically sound, updating all transport layers (`send-pack.c`, `transport-helper.c`, `transport.c`) and introducing a new advice message (`advice.pushRefUnverifiable`) for pushes lacking a reflog. The series includes 81 lines of new test coverage in `t/t5533-push-cas.sh` and is based on `maint`. All prior review feedback from Patrick Steinhardt, Junio C Hamano, and Knoble is resolved or superseded by the v5 design. The only remaining loose end is explicit test coverage for the fast-forward exemption in patch 3/3, which the author notes is logically sound but pending.

### ODB transaction race fix sparks architectural discussion
**[2026/09/14/11-31-17]** Justin Tobler proposed an alternative architectural fix for Qin ShiCheng's six-part bugfix series targeting a data-loss race in `git repack -d`. The race occurs when concurrent pushes cause the repack machinery to delete packs whose objects were never copied elsewhere, leaving dangling refs. Tobler's proposal—stop relying on `git index-pack` to create `.keep` files prematurely and instead have the ODB transaction's commit phase create them explicitly—would eliminate the need for pre-migration tracking of `.keep` files entirely.

Tobler's review is substantive and tested: he mentions prototyping this approach locally as part of a separate series, indicating feasibility. The proposal is framed as a preference rather than a demand, leaving room for Qin to proceed with the incremental fix (patch 1/6) or collaborate on a larger refactor. This discussion highlights a tension between immediate fixes and long-term maintainability, with potential implications for Patrick Steinhardt's ongoing ODB abstraction effort.

### `git var` extension series to be reorganized
**[2026/08/25/20-46-42]** Andrew Pleeter agreed to split the v8 patch extending `git var` into three smaller patches: (1) adding `-z` output mode, (2) enabling multi-variable queries, and (3) introducing new variables (`GIT_AUTHOR_NAME`, `GIT_SIGNING_KEY`, etc.). The series provides a unified, scriptable interface for Git's identity and signing configuration, addressing prior feedback from Junio C Hamano and Phillip Wood. The v9 series will be the next step toward integration, now with a clearer, more maintainable patch structure.

### Precompiled headers series advances to v2
**[2026/09/09/19-50-02]** SZEDER Gábor posted v2 of a four-patch series introducing precompiled headers for `git-compat-util.h` to speed up Git builds by ~35%. The update addresses Junio C Hamano's editorial feedback on patch 3/4's commit message, reordering the explanation to prioritize the series' goal, exclusion rationale, and mechanical change. The series is self-contained, touching only the `Makefile`, `contrib/buildsystems/CMakeLists.txt`, and `.gitignore`. While the 35% speedup claim remains anecdotal, the implementation is mechanical and unlikely to be controversial. The CMake adjustments (patches 2/4 and 3/4) may draw scrutiny from Windows builders to confirm cross-platform consistency.

### Documentation backtick-quoting fixes merged
**[2026/09/12/19-14-59]** Junio C Hamano accepted Todd Zullinger's v3 documentation patch series, which ensures consistent backtick-quoting of command, option, and subcommand references in the synopsis-style AsciiDoc source for `git-refs` and `git-pack-refs`. The series converts the SYNOPSIS block from `[verse]` to `[synopsis]`, backtick-quotes all subcommand entries in `git-refs.adoc`, and updates `pack-refs-options.adoc` to backtick-quote configuration keys while leaving command-line options unquoted. The changes are purely visual, aligning with the ongoing synopsis-style documentation effort led by Jean-Noël Avila.

## In brief
- **[2026/06/14/14-15-40]** Kaartic Sivaraam resubmitted the already-merged v4 patch fixing memory leaks in `git history reword`, confirming no new developments.
- **[2026/08/11/17-02-00]** Grayson Gordon posted the final combined v7 patch for `http.sslVerifyStatus`, now in `next` and awaiting merge to `master`.
- **[2026/09/04/10-36-01]** Junio C Hamano confirmed `OPT_ALIAS_F()` with `PARSE_OPT_HIDDEN` is the correct deprecation mechanism for Patrick Steinhardt's ref-storage terminology unification series.
- **[2026/09/07/04-44-23]** D. Ben Knoble confirmed and reproduced the stash/merge autostash bug, tracing the root cause to premature `MERGE_AUTOSTASH` deletion via `remove_merge_branch_state()`.
- **[2026/09/08/01-46-40]** Pia Park pinged Derrick Stolee seeking agreement on returning silent success (exit status 0) for the "no packs" case in MIDX writes.
- **[2026/09/11/16-41-17]** Harald Nordgren posted v3 of `--matched-only` for `git range-diff`, reverting the error-handling layering violation from v2 and restoring architectural separation.
- **[2026/09/12/22-34-19]** Junio C Hamano reviewed a bugfix patch preventing `git pull` segfaults on invalid merge heads, suggesting a minor test simplification.
- **[2026/09/13/20-26-20]** Justin Tobler declined Karthik Nayak's optimization suggestion for the ODB transaction race fix, leaving the patch unchanged.
- **[2026/09/14/13-31-40]** René Scharfe posted a patch removing an optimization in `dir.c` that caused `git status --ignored` to perform substring matching instead of exact matching.
- **[2026/09/14/19-52-16]** Junio C Hamano confirmed he will pick up v4 of `ks/history-commit-leakfix` for `next`, and Toon Claes endorsed `ps/odb-pluggable-fsck` (v3) after examining the range-diff.
- **[2026/09/15/02-43-13]** Junio C Hamano requested minor stylistic adjustments to Brigham Campbell's patch teaching `git-contacts` to read patch content from stdin.
- **[2026/09/15/15-06-38]** Junio C Hamano disputed André Kießling's report of inconsistent behavior between `+` refspec modifier and `--force` in `git fetch`, providing reproduction steps showing no pruning with `+` on Unix.
- **[2026/09/15/20-24-31]** Phil Sainty reported that `GIT_WORK_TREE` is not exported to `post-checkout` hooks in worktrees, causing `git rev-parse --show-toplevel` to return the wrong path.