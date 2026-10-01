# Git mailing list daily digest for 2026/09/30

## The day in brief
The Git mailing list saw active discussion around several key topics today. A new test case was contributed for the `filter-branch` bugfix, while the `parse-options` API refactoring faced questions about behavioral consistency. The CI/build system improvements continued with a v3 series using twice the CPU count for parallel jobs. A sparse-checkout regression was reported, and the `git stash pop` feature extension faced fundamental design questions. The `gitmergeconflicts(7)` documentation series saw productive discussion about commit accessibility and `diff3` promotion.

## Notable threads

### filter-branch: fix commit mapping inversion with --state-branch
**What changed**: Michele Locati provided a new test case that directly exercises incremental runs with `--prune-empty` and `--subdirectory-filter`, ensuring the state branch file contains correctly ordered "original:rewritten" entries.

**Why it matters**: This test case confirms the bugfix works correctly and prevents future regressions in this area. The test fails on current `master` without the fix and passes with it, providing clear verification of the patch's effectiveness.

### Key technical details

- The test specifically targets incremental runs with `--prune-empty` and `--subdirectory-filter`
- It verifies the state branch file contains correctly ordered "original:rewritten" entries
- The test passes on Git v2.49.1 (last version before the regression)

### parse-options: new sub-API for early argument scanning
**What changed**: Kaartic Sivaraam identified a behavioral inconsistency with `parse_options()` where the API's decision to ignore abbreviations could misinterpret arguments.

**Why it matters**: This inconsistency could lead to subtle bugs in commands adopting the new API. For example, in `git fast-import --quiet --export-pack --allow-unsafe-features`, `--export-pack` is an abbreviation of `--export-pack-edges`, so `--allow-unsafe-features` should be treated as its value, not as a separate option.

### Key technical details

- The API ignores abbreviations, which could misinterpret arguments
- Proposed solution: stop the scan at the first unrecognized argument
- Also suggested expanding test coverage for edge cases

### ci: use cmp and align job-count selection
**What changed**: Tamir Duberstein posted v3 of the CI/build system series, now using twice the CPU count for both GitHub and GitLab CI.

**Why it matters**: This change provides a data-driven approach to parallel job selection, showing a 7% speedup on Linux and avoiding a 26% slowdown on macOS compared to the one-job-per-CPU policy.

### Key technical details

- Replaces GitHub's fixed `-j10` and GitLab's one-job-per-CPU policy
- Uses native CPU-count queries: `nproc` (Linux), `sysctl -n hw.logicalcpu` (macOS), `NUMBER_OF_PROCESSORS` (Windows)
- Benchmark data shows optimal trade-off between Linux speed and macOS stability

### sparse-checkout regression: checkout clobbers untracked in-cone files
**What changed**: Webstrand reported a regression in Git v2.27.0 where `git checkout` in a sparse-checkout working tree silently overwrites untracked files that lie within the sparse-checkout cone.

**Why it matters**: This is a serious correctness issue that can silently destroy untracked work. The expected behavior is to preserve untracked content and abort with an error, but instead the checkout completes and overwrites the untracked file.

### Key technical details

- Regression introduced in commit 681c637b4a
- Affects all versions from v2.27.0 through current master
- Misleading warning message: "paths were already present and thus not updated"
- The bug breaks the invariant that sparse checkouts should behave identically to non-sparse checkouts

### stash: extend `pop` with custom conflict-label options
**What changed**: Junio C Hamano questioned the fundamental motivation for exposing `--label-ours`, `--label-theirs`, and `--label-base` options in user-facing commands.

**Why it matters**: This design discussion challenges whether these options should exist in user-facing commands at all. Junio traced their origin to internal machinery for `git checkout` and suggested they may have been added to `stash apply` only for debugging convenience.

### Key technical details

- Options allow customizing conflict markers when popping a stash
- Junio suggests the options might be better removed from `stash apply` rather than extended to `pop`
- No technical flaws in the implementation were raised

### gitmergeconflicts(7): new man page for merge conflict resolution
**What changed**: Junio C Hamano and Julia Evans discussed making commit information more accessible during conflict resolution and promoting `merge.conflictstyle=diff3` as a best practice.

**Why it matters**: These discussions are elevating the guide from passive documentation to active tooling advice. Junio provided concrete examples of how commit information can aid resolution, while Julia's user feedback supports making `diff3` a best practice.

### Key technical details

- Junio suggested commands like `git show $commit` and `git diff ...$commit` to reveal intent behind changes
- Julia is open to including commit messages but skeptical about commit IDs
- Junio strongly advocated for making `diff3` the default conflict style
- Julia's user feedback: "every time I show people diff3 someone tells me how happy they are"

## In brief
- **rerere: fix race condition**: Patrick Steinhardt reviewed the second patch in Thomas Bachem's series, focusing on documentation clarity and flag naming for `--auto` mode.
- **submodule merge: stale delta-base cache**: Philippe Blain bisected the regression to two commits, narrowing the root cause to the delta-base cache.
- **ci: improve leak sanitizer failure reporting**: Phillip Wood reported that clickable links don't work in GitHub UI, creating a critical gap in the patch's functionality.
- **gitbreaking-changes(7)**: Junio C Hamano proposed a hybrid URL+message-ID format to preserve raw message-IDs for future reference.
- **gittutorial-2 removal**: Junio C Hamano expressed confusion about Julia Evans's stance on preserving the older `gittutorial`.
- **xdiff: switch mmfile_t buffers**: Jeff King clarified that the bug fixed in follow-up patches predates the series and is unrelated to the `NULL` behavior.
- **reftable: fix timezone handling**: Junio C Hamano endorsed explicit sign-flipping logic for readability in timezone conversion.
- **git backfill --dry-run**: Derrick Stolee objected to the feature on conceptual grounds, arguing the uncompressed size estimate may not be useful.
- **MIDX reachability closure**: Jeff King identified several edge cases that may remain unaddressed in the series.
- **checkout -m: remember original conflict labels**: Johannes Sixt suggested using an index extension instead of `.git/MERGE_LABELS` to store conflict labels.
- **refs/packed-backend: optimize rewrites**: Karthik Nayak posted a performance optimization patch for the packed-refs backend.
- **revision: add @{p} shorthand**: Junio C Hamano queued the patch for integration, directing the author to strengthen the commit message.