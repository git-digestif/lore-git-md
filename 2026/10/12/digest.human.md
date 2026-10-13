# Git mailing list daily digest for 2026/10/12

## The day in brief
The Git mailing list saw significant progress on several fronts today. Junio Hamano provided key feedback on the incremental connectivity check series, suggesting architectural improvements. The `git blame` default ignore-revs feature received maintainer approval with a preference shift toward working-tree file handling. Multiple bugfix patches advanced toward integration, including fixes for `fast-import` crashes, Windows credential helper parsing, and ZIP archive timestamp handling. Documentation improvements for merge conflicts and contribution guidelines were finalized.

## Notable threads

### Incremental connectivity check for fetch/push
Kristofer Karlsson posted v3 of the incremental connectivity check series, splitting it into four patches (documentation, performance tests, config plumbing, implementation). Junio Hamano reviewed the plumbing patch (3/4) and suggested adding a `struct repository *repo` field to `check_connected_options` to reduce reliance on `the_repository`, aligning with ongoing architectural improvements. The maintainer also expressed enthusiasm for the feature ("This is exciting") in response to the v3 submission, signaling strong interest in the performance optimization for large repositories.

### `git blame` default ignore-revs file
The `git blame` default ignore-revs feature received maintainer feedback that shifts the implementation direction. Junio Hamano rejected the "match hosting platforms" justification for reading the committed `HEAD:.git-blame-ignore-revs` blob and explicitly preferred consulting the working-tree file in non-bare repositories as "more ergonomic." This feedback changes the uncommitted changes handling from a neutral trade-off to a clear direction for the next revision. The series remains approved for integration but will likely require adjustments to match this preference.

### `git stash` shell completion improvements
D. Ben Knoble posted a five-patch series improving `git stash` shell completion. The series includes refactoring, a Zsh-specific bug fix for stash name completion (e.g., `stash@{0}`), and the removal of completion and documentation support for the deprecated `git stash save` subcommand. The Zsh fix addresses a real usability issue where brace expansion prevented proper completion of stash names. The removal of the deprecated `save` subcommand (deprecated since 2017) may spark discussion about accelerating the deprecation of long-standing but obsolete features.

### `fast-import` crash fix
Marc Becker posted v2 of a bugfix for `git fast-import` that addresses crashes when resolving commits created earlier in the same session but not yet written to the object database. The v2 patch adds comprehensive test coverage in `t/t9300-fast-import.sh` that verifies resolution of pending commits and tags in both SHA-1 and SHA-256 repositories. The fix remains minimal, modifying only the `parse_from_existing` function to check the fast-import object cache before falling back to the ODB.

### Windows credential helper parsing fix
Marc Becker posted v3 of a bugfix for the Windows credential helper (`contrib/credential/wincred`) that fixes multi-line secret parsing. The patch addresses unsafe string operations that could cause buffer overreads and memory corruption when handling credentials containing newlines (e.g., OAuth refresh tokens alongside passwords). The v3 patch incorporates all prior feedback, including refactoring into helper functions and consistent wide-character handling.

## In brief
- **`git worktree repair` validation**: Yoichi NAKAYAMA clarified that the refactored `infer_backlink()` function rejects `.git` files with missing spaces after `gitdir:` (e.g., `gitdir:foo/bar`), prioritizing fixing common issues over repairing manually corrupted files.
- **ZIP archive timestamp handling**: Matthew E. Luallen submitted a test infrastructure fix for René Scharfe’s ZIP archive patch, replacing `TIME_IS_64BIT` with `TIME_T_IS_64BIT` to ensure tests pass on 64-bit platforms and are skipped on 32-bit systems where `zipinfo(1)` lacks UT field support.
- **`imap-send` OpenSSL compatibility**: Aniket Gupta identified a build-breaking deprecation warning in OpenSSL 4.1.0-beta2 due to the use of `ASN1_STRING_length` instead of `ASN1_STRING_get_length`, which breaks the build when `DEVELOPER=1` is set.
- **`git mergeconflicts` documentation**: Junio Hamano provided a squashable patch refining the `diff3` example in `Documentation/gitmergeconflicts.adoc` to clarify the pedagogical intent and avoid ambiguity in the conflict marker illustration.
- **`git branch --delete-merged`**: Harald Nordgren agreed to explore incorporating authorship information from `git range-diff` as a performance filter for rebase detection, aligning with Junio Hamano’s proposal to reuse `git range-diff`’s commit-matching logic.
- **`git fetch` shallow tag backfilling**: Junio Hamano raised three substantive questions about the patch’s logic, including whether the regression can occur when the main fetch transaction is empty and the impact of skipping the two-transaction fix for `--atomic` fetches.
- **Contribution guidelines**: Junio Hamano queued a documentation patch updating `SubmittingPatches` to discourage splitting test and fix into separate commits, finalizing the guideline after editorial feedback.
- **Unicode character width tables**: Beat Bolli posted a patch updating Git’s Unicode character width tables to Unicode 18, and Junio Hamano confirmed its correctness and queued it for integration.
- **`tig` pager configuration**: Beat Bolli posted a patch adding syntax highlighting for the `Fixes:` and `Link:` trailers in the `tig` pager’s configuration file.