# Git mailing list daily digest for 2026/10/12

## The day in brief
The Git mailing list saw significant activity around platform compatibility fixes (macOS CRLF handling, OpenSSL 4.1 compatibility), performance features (incremental connectivity checks for fetch/push), and documentation improvements (new `gitmergeconflicts(7)` guide). Maintainer feedback advanced several series toward integration, while new bug reports and feature proposals emerged in areas like `git stash` completion and Unicode 18 support.

## Notable threads

### macOS CRLF conversion bugfix (read-cache.c)
**[2026/10/12/19-11-16 by Junio C Hamano]**
Junio Hamano provided substantive review of Torsten Bögershausen's RFC patch series addressing a macOS-specific CRLF conversion bug in `read-cache.c`. The patch introduces `size_change_is_conclusive()`, a helper that prevents Git from blindly trusting post-conversion file sizes when clean filters or CRLF conversion alter file size without changing content. Junio's feedback focused on performance implications of the unconditional `would_convert_to_git()` calls and requested data on overhead in repositories without clean filters. The series remains in RFC status pending incorporation of this feedback.

### Incremental connectivity check for fetch/push
**[2026/10/12/12-07-53 by Kristofer Karlsson via GitGitGadget]**
Kristofer Karlsson posted v3 of the incremental connectivity check series, splitting it into four well-organized patches (documentation, performance tests, config plumbing, implementation). The feature introduces `transfer.connectivityCheck=incremental` to reduce the cost of verifying object connectivity during fetches and pushes by processing incoming commits in topological order. Junio Hamano signaled strong interest ("This is exciting") and accepted the performance trade-offs (1.5x slowdown for very long histories) as reasonable for an opt-in feature. The series is now effectively cleared for integration pending final review.

### New `gitmergeconflicts(7)` documentation
**[2026/10/12/18-37-04 by Junio C Hamano]**
Junio Hamano provided the final squashable patch for Julia Evans' `gitmergeconflicts(7)` documentation series, refining the `diff3` example to clarify pedagogical intent. The series consolidates scattered merge conflict advice into a single, comprehensive guide and is now fully approved for integration. Junio's feedback on the `diff3` section improved the example's clarity by showing how the common ancestor reveals the *intent* behind conflicting changes, making the guide more effective for users.

### `git branch --delete-merged` squash/rebase detection
**[2026/10/12/16-50-43 by Junio C Hamano]**
Junio Hamano proposed a fundamental reorganization of Harald Nordgren's patch series, suggesting separate patches for rebase detection (using `git range-diff` commit-matching logic) and squash detection. He raised architectural concerns about the CLI design, questioning whether a single `--delete-merged` flag with a `--no-squashed` opt-out can scale to future workflows. Harald agreed to explore incorporating authorship information from `git range-diff` as a performance filter for rebase detection, but the series' long-term design remains under discussion.

### ZIP archive date-handling fixes
**[2026/10/12/15-07-56 by Junio C Hamano]**
Junio Hamano confirmed the correctness of Matthew E. Luallen's test infrastructure fix for René Scharfe's ZIP archive patch, resolving CI failures on 32-bit platforms. The fix replaces `TIME_IS_64BIT` with `TIME_T_IS_64BIT` in test prerequisites, ensuring tests pass on 64-bit platforms and are skipped on 32-bit systems where `zipinfo(1)` lacks UT field support. The patch omits the Extended Timestamp (UT) extra field when commit times exceed 32-bit range, preventing silent timestamp wrap. Junio noted a broader cleanup opportunity to audit `timestamp_t` coercion into `time_t` for external formats.

## In brief
- **[2026/10/12/21-13-40 by Junio C Hamano]** Maintainer rejected the "match hosting platforms" justification for `git blame`'s default ignore-revs file behavior, preferring to consult the working-tree file in non-bare repositories.
- **[2026/10/12/19-55-11 by Junio C Hamano]** Maintainer raised three substantive questions about Karthik Nayak's shallow fetch bugfix, probing edge cases around empty transactions, atomic fetches, and cleanup paths.
- **[2026/10/12/22-50-20 by Marc Becker via GitGitGadget]** Author posted v2 of the `fast-import` crash fix, adding comprehensive test coverage for pending commits and tags in both SHA-1 and SHA-256 repositories.
- **[2026/10/12/17-48-26 by D. Ben Knoble]** Author posted a five-patch series improving `git stash` shell completion, including a Zsh bug fix and removal of deprecated `git stash save` completion.
- **[2026/10/12/21-04-50 by Beat Bolli]** Author posted a patch updating Git's Unicode character width tables to Unicode 18, adjusting intervals for zero-width and double-width code points.
- **[2026/10/12/15-17-48 by Aniket Gupta]** Reviewer identified a build-breaking deprecation warning in OpenSSL 4.1.0-beta2 due to use of `ASN1_STRING_length` instead of `ASN1_STRING_get_length`.
- **[2026/10/12/19-57-06 by Yoichi NAKAYAMA]** Author clarified that the refactored `infer_backlink()` function rejects `.git` files with missing spaces after `gitdir:`, prioritizing common fixes over manually corrupted files.
- **[2026/10/12/18-49-08 by Junio C Hamano]** Maintainer suggested adding a `struct repository *repo` field to `check_connected_options` to reduce reliance on `the_repository`, aligning with the ongoing `the_repository` removal effort.
- **[2026/10/12/21-25-21 by Junio C Hamano]** Maintainer queued Beat Bolli's Unicode 18 character width update after confirming its correctness with the `update_unicode.h` script.