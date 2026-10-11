# Git mailing list daily digest for 2026/10/10

## The day in brief

The `rerere` lock contention bugfix series reached final form after Thomas Bachem dropped the controversial third patch, leaving a conservative two-patch solution that makes concurrent `rerere` operations wait briefly for the lock. Harald Nordgren’s matching-inspired fetch mode for shallow repositories received targeted feedback on patch 2/5, requiring a respin to defer control-flow changes and restore a NULL-check. The `git blame` default ignore-revs file series saw procedural clarifications, with Ravi Mistry agreeing to move a helper extraction to patch 1/2 and drop the NUL-byte check if Junio prefers. Meanwhile, the `reference-transaction` hook fix remains stalled on process concerns, with brian m. carlson reinforcing that patches must meet quality standards regardless of origin.

## Notable threads

### rerere: fix race condition between rebase and background maintenance

Thomas Bachem’s two-patch series (v7) addressing `rerere` lock contention is now final after dropping the controversial third patch. The series introduces a configurable timeout (`rerere.lockTimeout`, defaulting to 1000 ms) for the `MERGE_RR.lock` file, allowing processes to wait briefly for the lock instead of failing immediately. Patch 2 adds a hidden `--skip-locked` flag to `git rerere gc`, enabling auto-maintenance tasks to skip silently if the lock is held. The decision to drop the warning-and-proceed behavior reflects feedback from Patrick Steinhardt, who argued it was unnecessary without real-world user reports. The series is technically complete, with all prior feedback incorporated, and awaits Junio’s final review.

### blame: default to HEAD:.git-blame-ignore-revs if no ignore file configured

Ravi Mistry’s v2 series making `HEAD:.git-blame-ignore-revs` the default ignore-revs file for `git blame` saw procedural clarifications today. Mistry agreed to move the `parse_oidset_line()` helper extraction to patch 1/2 in the next revision, addressing Junio’s preference for logical patch organization. He also offered to drop the NUL-byte check entirely if Junio prefers, signaling flexibility on this stylistic choice. The series remains feature-complete, with the security hardening (promisor fetch prevention, tag type verification) and default file lookup logic unchanged. Broader review of the hardening is still needed, particularly from security-interested contributors like Jeff King or brian m. carlson.

### `reference-transaction` hook omits new ref on branch rename

The stalled `reference-transaction` hook fix thread saw further meta-discussion today. Maciej Ciemborowicz offered to step aside entirely, saying he would be "more than happy" if Karthik Nayak (or anyone else) wanted to write the patches using an LLM. brian m. carlson reinforced that it is better not to submit a patch at all than to submit one that does not meet quality standards or violates contribution guidelines. The series remains under review, with no merge status reported. The core issue—omitting the new ref when a branch is renamed or copied—is technically sound, but the process concerns around AI-assisted contributions and pre-version engagement with feedback remain unresolved.

### fetch: introduce `+:` refspec for matching-inspired fetch mode

Harald Nordgren’s v8 series introducing a matching-inspired fetch mode for shallow repositories received targeted feedback on patch 2/5. Junio requested that the `if/else if/else` control-flow reorganization be deferred to patch 3/5 and that the NULL-check for `branch` in `collect_upstream_from_remote()` be restored. Nordgren acknowledged both points and will respin the patch. The series is otherwise technically sound, with all prior review concerns addressed. The core feature—dynamically inferring which branches to fetch based on local branches’ `@{upstream}` settings—remains uncontroversial and ready for integration once the respin is complete.

### checkout -m: remember original conflict labels

Junio submitted a follow-up refactoring patch to Phillip Wood’s series teaching `git checkout -m` to remember original conflict labels. The patch optimizes performance by caching per-invocation constants (e.g., `merge.renormalize`, `.git/MERGE_LABELS`) in a new `struct checkout_merged_data`, avoiding redundant file I/O and config reads. The change is a pure refactoring with no user-visible behavior change and aligns with Git’s pattern of caching per-invocation state. The patch is uncontroversial and ready for integration, though Junio suggested it could be applied as a separate "PATCH 1.5/2" or squashed into Phillip’s patch 2/2.

## In brief

- **repo: add revision filtering support to 'git repo structure'**: Mark C. Chu-Carroll posted v4 of the series, fixing the accidental corruption in `revision.c` and `revision.h` from v3. The patch remains blocked by unaddressed feedback on commit message motivation, documentation precision, and the double ref-walk inefficiency.
- **git stash create: support --include-untracked and --all**: Kazumasa Shigeta proposed limiting the current patch to `-m/-q/-u/-a` and leaving `--patch`, `--staged`, and pathspec support for follow-ups. The design discussion remains unresolved.
- **sha1dc-accel: faster SHA-1 collision detection**: D. Ben Knoble and Todd Zullinger clarified how distributions handle Git’s mixed-license codebase, reinforcing that the series is legally ready for integration.
- **git repack --drop-filtered --dry-run**: Siddharth Shrimali posted v2 of the bugfix, incorporating all review feedback and documenting the historical context. D. Ben Knoble provided his `Reviewed-by`.
- **index refresh inconsistency with line-ending conversions**: Curtis Allen Smith proposed splitting the series into a minimal v2 with only the core fix, deferring the optimization and config option to follow-ups.
- **contrib/credential/wincred: fix multi-line secret parsing**: Marc Becker posted v2 of the patch, fully addressing Junio’s feedback on commit message clarity, code readability, and wide-character handling.
- **fast-import: fix crash when resolving unreachable in-session commits**: Marc Becker posted a bugfix adding a cache lookup in `parse_from_existing` to avoid crashes when resolving commits created earlier in the same session.