# Git mailing list daily digest for 2026/10/12

## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread: 59 replies so far from 6 participants over 19 days; 6 messages that day.*

Julia Evans posted v3 of her six-patch series adding a new `gitmergeconflicts(7)` manual page explaining merge conflicts with examples, linked from the git-merge, git-rebase, git-revert, git-cherry-pick and git-pull docs. The v3 changelog switches `Reviewed-by` to `Helped-by`, adds a semantic-conflict note, replaces the zdiff3 explanation with a brief ambiguity warning, and rewords several examples. Junio C Hamano reviewed patch 1/6, objecting that the "one side added 'cherry' and the other added 'banana'" example sends the wrong message because rcs-merge-style output is ambiguous; he supplied a squashable patch with a corrected diff3 example and said everything else in the series looked good. D. Ben Knoble questioned a duplicated sign-off and which email address to use; Evans acknowledged both, calling the duplicate a copy-paste error.

### rerere waiting for `MERGE_RR.lock`

*Long thread: 30 replies so far from 4 participants over 41 days; 1 message that day.*

Thomas Bachem's two-patch series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds (default 1000) for `MERGE_RR.lock` before failing, so a rebase hitting a conflict while a background `git rerere gc` holds the lock no longer dies, and adds a hidden `--skip-locked` option to `git rerere gc` so auto-maintenance does nothing while the lock is held. Today Junio C Hamano called Bachem's plan to move the test's `wait` into `test_when_finished` -- so it runs however the test ends -- "quite sensible".

### `branch --delete-merged` detecting squash-merged branches

*Long thread: 29 replies so far from 5 participants over 14 days; 7 messages that day.*

Harald Nordgren's v2 series makes `git branch --delete-merged` treat a branch as merged when some upstream commit since the fork point contains all of its changes, naming that commit as "landed as", and adds `--no-squashed` to skip the slower squash search. Today Phillip Wood asked for a detailed cover-letter explanation of how squashed merges are detected and its tradeoffs versus patch IDs or test merges; Nordgren clarified the check is "does C already contain everything B changed" rather than an exact squash match, agreed to document the approach, and argued against following renames as a basis for assuming merged. Junio C Hamano questioned whether one `--delete-merged` with a `--no-squashed` knob is an extensible UI, and Nordgren said he would study the range-diff code and use authorship as a speed filter.

### `checkout -m` recreating original conflict labels

*Long thread: 20 replies so far from 3 participants over 13 days; 2 messages that day.*

Phillip Wood's series teaches the ort machinery to write conflict labels to `.git/MERGE_LABELS` when switching to a conflicted merge result so `git checkout -m` reuses them. Today Wood agreed with Junio C Hamano's earlier suggestion to read per-invocation constants once via a new `checkout_merged_data` structure with `checkout_merged_prepare()`/`checkout_merged_release()` in `builtin/checkout.c`, said he would re-roll with the included diff, and asked about adding Hamano's sign-off; Hamano confirmed it could be added but said it probably counts more as "helped-by" than co-authorship.

## The day in brief
Julia Evans's v3 of the `gitmergeconflicts(7)` documentation series drew a mostly positive review from Junio C Hamano, Ravi Mistry posted v4 of the blame default-ignore-revs series with a 100MB blob cap, and Karthik Nayak posted v2 of the fetch fix that commits references before backfilling tags so shallow fetches avoid requesting all objects.

## Notable threads

### parse-options early-scan API
Christian Couder posted v3 of his four-patch "Standardize early option scanning" series, adding `early_scan_options()` and the `PARSE_OPT_EARLY` flag to the parse-options API so commands can safely skip option values during early scans; the scan recognizes negated forms, unambiguous abbreviations and aliases, and stops at anything it cannot reliably skip. `git fast-import` now uses it for `--allow-unsafe-features`, fixing disagreements with `parse_options()` so abbreviations like `--allow-unsafe` work.

### blame using `HEAD:.git-blame-ignore-revs` by default

Ravi Mistry posted v4 of the series teaching `git blame` and `git annotate` to use the `HEAD:.git-blame-ignore-revs` blob by default; the only change new in v4 caps that blob at 100MB via `odb_read_object_info()`, skipping oversized blobs with a warning. Earlier discussion covered reading the working-tree copy instead of the HEAD blob; Mistry left the HEAD behavior in place to match hosting sites, and Junio C Hamano said reading only from HEAD needs justification beyond that.

### fetch committing references before backfilling tags
Karthik Nayak posted v2 of "fetch: commit references fetched before backfilling tags", which commits the previous batched reference update before tag backfill so shallow fetches report refs in the 'have' section and avoid requesting all objects, with benchmarks and a regression test. Junio C Hamano asked whether the problem also occurs when the main fetch transaction is empty, whether `--atomic` users should expect a degenerate full clone, and whether the `goto cleanup` on a failed tag-backfill transaction inappropriately skips post-fetch processing.

## In brief
- Torsten Bögershausen posted v3/RFC of "read-cache: do not trust a size change when conversion is active", picked up from Curtis Allen Smith; Junio C Hamano's review asked about wording, co-authorship, and the overhead of the new attribute lookup.
- Beat Bolli posted v2 letting OpenSSL do imap-send hostname verification via `SSL_set1_host()`; Aniket Gupta objected that the post-`SSL_connect()` check may be unreachable under `SSL_VERIFY_PEER`.
- Kristofer Karlsson posted v3 of the incremental connectivity check series; Junio C Hamano suggested reducing `the_repository` use in `connected.c` via a `struct repository *` in `check_connected_options`.
- **archive-zip UT field patch, i386 failures**: René Scharfe's patch omitting the UT extra field for out-of-range mtimes hit i386 CI failures. Matthew E. Luallen proposed TIME_T_IS_64BIT; Junio C Hamano acknowledged the distinction, leaving timestamp_t auditing as #leftoverbits.

- Marc Becker posted v2 fixing a `git fast-import` crash when querying commits or tags created in the same session, resolving them via the object cache first.
