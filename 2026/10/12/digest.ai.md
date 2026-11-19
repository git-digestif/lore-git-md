# Git mailing list daily digest for 2026/10/12

## Long threads

### New `gitmergeconflicts(7)` documentation page

*Long thread (root 2026/09/24/14-44-15): 59 replies so far from 6 participants over 19 days; 6 messages that day.*

Julia Evans posted v3 of her six-patch documentation series, which adds a new `gitmergeconflicts(7)` manual page explaining merge conflicts with examples and links to it from the git-merge, git-rebase, git-revert, git-cherry-pick, and git-pull docs instead of repeating the guidance in each command. The v3 changelog switches `Reviewed-by` to `Helped-by`, adds a semantic-conflict note, replaces the zdiff3 explanation with a brief ambiguity warning, and rewords several examples. D. Ben Knoble asked why her `Signed-off-by` appeared twice and noted his preferred email address; Julia Evans acknowledged both, calling the duplicate a copy-paste error. Junio C Hamano reviewed patch 1/6, objecting that the "one side added 'cherry' and the other added 'banana'" example is ambiguous in rcs-merge-style output, and supplied a squashable patch with a corrected diff3 example; he said everything else in v3 looked good.

### rerere waiting for `MERGE_RR.lock`

*Long thread (root 2026/09/02/08-31-37): 30 replies so far from 4 participants over 41 days; 1 message that day.*

Thomas Bachem's v7 series makes `setup_rerere()` wait up to `rerere.lockTimeout` milliseconds (default 1000) for `MERGE_RR.lock` before failing, so a rebase that hits a conflict while a background `git rerere gc` holds the lock no longer dies, and adds a hidden `--skip-locked` option to `git rerere gc` for auto-maintenance. Today Junio C Hamano endorsed Thomas Bachem's plan to move the test's `wait` into `test_when_finished` (as pack-refs-tests.sh does) so it runs however the test ends, calling it "quite sensible".

### `branch --delete-merged` detecting squash-merged branches

*Long thread (root 2026/09/29/07-30-30): 29 replies so far from 5 participants over 14 days; 7 messages that day.*

Harald Nordgren's v2 series makes `git branch --delete-merged` treat a branch as merged when some upstream commit since the fork point contains all of its changes, naming that commit as "landed as", and adds `--no-squashed` to skip the slower squash search. Phillip Wood asked for a detailed cover-letter explanation of the detection approach and its tradeoffs versus patch IDs or test merges, characterizing it as roughly checking whether the upstream tree already contains the branch's changes via per-path `merge_ll()` calls and noting that rename detection is missed. Harald Nordgren rejected commit-message matching as brittle, clarified the check is "does C already contain everything B changed" rather than exact squash equality, and proposed reorganizing into three commits. Junio C Hamano called the tradeoff analysis sensible, said one-to-one rebase matching should learn from range-diff's log-message and authorship similarity, and questioned whether a single `--delete-merged` with a `--no-squashed` knob is an extensible UI. Harald Nordgren agreed to look at the range-diff code and use authorship as a speed-up filter.

### `checkout -m` recreating original conflict labels

*Long thread (root 2026/09/30/09-48-47): 20 replies so far from 3 participants over 13 days; 2 messages that day.*

Phillip Wood's v3 series teaches the ort merge machinery to write conflict labels to `.git/MERGE_LABELS` when switching to a conflicted result, so `git checkout -m <path>` recreates conflicts with the original labels, falling back to "base"/"ours"/"theirs". Phillip Wood agreed to re-roll using Junio C Hamano's diff introducing a `checkout_merged_data` struct with `checkout_merged_prepare()`/`checkout_merged_release()` in builtin/checkout.c, which reads `merge.renormalize` and the labels once per invocation instead of once per conflicted path, and asked whether he could add Junio's sign-off; Junio C Hamano confirmed but said it probably counts more as "helped-by" than co-authorship.

## The day in brief
Julia Evans's v3 documentation series for a new `gitmergeconflicts(7)` page drew detailed review of its diff3 example, while Harald Nordgren's squash-merge detection for `branch --delete-merged` prompted a design discussion about detection tradeoffs and UI shape. Karthik Nayak's shallow-fetch tag-backfill fix and Ravi Mistry's blame default-ignore-revs v4 also drew substantive review questions.

## Notable threads

### Fetch committing refs before backfilling tags
Karthik Nayak posted v2 of a fix for shallow fetches: since batched reference updates were introduced, tag backfilling happened before refs were committed, so shallow clients reported no "have" refs and servers could send all objects. The patch commits the previous batched update before tag backfill and adds a regression test verifying objects beyond the specified depth are not fetched. Junio C Hamano asked whether the problem also occurs when the main fetch transaction is empty, whether `--atomic` users should expect a degenerate full clone since the fix is skipped there, and whether the `goto cleanup` on tag-backfill failure inappropriately skips post-fetch processing such as FETCH_HEAD and upstream setup.

### Blame defaulting to `HEAD:.git-blame-ignore-revs`
Ravi Mistry posted v4 of a series teaching git-blame and git-annotate to use the `HEAD:.git-blame-ignore-revs` blob by default so local runs match hosting platforms; v4 caps the blob at 100MB via `odb_read_object_info()`, skipping oversized blobs with a warning, and hardens the parser against NUL bytes and partial-clone promisor fetches. Phillip Wood questioned ignoring uncommitted working-tree changes and the previously unbounded blob size; Ravi Mistry explained the HEAD-blob choice matches forges and that the cap addresses the size concern. Junio C Hamano said reading only from HEAD needs justification beyond matching forges, and that consulting the working-tree copy would be more ergonomic in non-bare repos.

### Standardizing early option scanning in parse-options
Christian Couder posted v3 of a four-patch series adding an "early-scan" sub-API to parse-options so commands can safely skip option values during early argument scans; `early_scan_options()` reuses long-option lookup, skips values of options that take them, and stops at anything it cannot reliably skip. git fast-import now uses it for `--allow-unsafe-features`, fixing disagreements where abbreviations like `--allow-unsafe` were mishandled. He also replied to Kaartic Sivaraam's v2 review, explaining the abbreviation handling and new test cases.

### imap-send letting OpenSSL verify hostnames
Beat Bolli posted v2 replacing the deprecated `SSLv23_method()` with `TLS_method()` and deleting the custom `host_matches()`/`verify_hostname()` functions in favor of `SSL_set1_host()` so OpenSSL performs hostname verification, printing `X509_verify_cert_error_string()` on failure. Aniket Gupta objected that the post-`SSL_connect()` verification check may be unreachable because `SSL_VERIFY_PEER` makes `SSL_connect()` itself fail on a hostname mismatch, suggesting the verify result be printed when `SSL_connect()` fails instead.

### read-cache not trusting size changes during conversion
Torsten Bögershausen posted v3/RFC of a patch picked up from Curtis Allen Smith, adding `size_change_is_conclusive()` in read-cache.c, which runs worktree contents through clean filters and CRLF conversion when file size differs from the index, with tests in t/t0020-crlf.sh. Junio C Hamano reviewed it with wording suggestions, questions about whether the cited examples change only size, whether Torsten has taken over the series or is collaborating (and whether co-authored-by is wanted once RFC status is lost), and how much overhead the attribute look-up adds when no clean filter is involved.

## In brief
- Marc Becker posted v2 fixing a `git fast-import` crash when querying commits or tags created in the same session, resolving them via the object cache first, with a new t/t9300-fast-import.sh test.
- René Scharfe's archive-zip patch omitting the UT extra field for out-of-range mtimes hit i386 CI failures; Matthew E. Luallen proposed using `TIME_T_IS_64BIT` instead of `TIME_IS_64BIT` for the beyond-2038 tests, and Junio C Hamano agreed, suggesting an audit of timestamp_t-to-time_t coercions.
- **Incremental connectivity check v3**: Kristofer Karlsson posted v3 of an incremental connectivity check gated behind `transfer.connectivityCheck=incremental`; Junio C Hamano suggested threading a `struct repository *` through `check_connected_options` and commented on benchmarks and the deepening-fetch fallback.

- Isabella Caselli sent a three-patch series adding a `hostname:` includeIf condition and `GIT_HOSTNAME` to `git var`; Junio C Hamano asked about the t6500 test's hostname-mismatch implications and remarked the helper makes conditional inclusion easy.
- D. Ben Knoble posted a five-patch stash completion series fixing Zsh stash-name quoting and proposing to drop `git stash save` from the docs and completion.
