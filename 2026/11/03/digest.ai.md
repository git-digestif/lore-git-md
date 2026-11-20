# Git mailing list daily digest for 2026/11/03

## Long threads

### Adding path keys to `git repo info`

*Long thread (root 2026/07/16/01-21-31): 52 replies so far from 5 participants over 111 days; 7 messages that day.*

K Jayatheerth's GSoC series adds `path.*` keys (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, `path.cdup`) with absolute and relative suffixes to `git repo info`. Today he agreed to Justin Tobler's review points across the series: exercise the existing `git rev-parse --show-superproject-working-tree` caller in the patch 2/8 test instead of the not-yet-introduced `path.superproject-root`, drop the early returns in the `path.superproject-root` getters for a single `if (get_superproject_working_tree(...))` form, print `/dev/null` for both `path.hooks` suffixes when `core.hooksPath=/dev/null` (special-cased in C so it also works on Windows), drop the `path.grafts` patch since grafts are deprecated, and fix the `path.cdup` test whose `cd` was not in a subshell. Justin Tobler replied that the `/dev/null` divergence would not block a shared helper between `git rev-parse` and `git repo info`.

### `%gb` placeholder naming the branch of each reflog entry

*Long thread (root 2026/10/15/13-13-55): 32 replies so far from 6 participants over 20 days; 1 message that day.*

Harald Nordgren's v3 series replaces `--operations`/`--historical-branch` with a `%gb` format placeholder for `git log -g --format` that names the branch HEAD was on when each reflog entry was recorded. Phillip Wood reviewed the patch and said "Looks good", suggesting rewording "the entries that branch switches... leave behind" to "created by" and criticizing "labelled with the branch being rebased" as sounding "very LLM"; he accepted the "detached at <commit>" output after trying it, but reported that with `GIT_REFLOG_ACTION` set during `git rebase`, `%gb` shows incorrect branch names for "sync switch" entries, displaying "sequencer-delete-REBASE_HEAD".

### Fixing `loose-object-idx` migration with batch fsync

*Long thread (root 2026/10/22/23-40-07): 23 replies so far from 4 participants over 13 days; 1 message that day.*

Justin Tobler's series fixes ODB transactions failing on commit under a compatibility hash because tmp-objdir migration did not handle the loose object index; v3 reads the source mapping before locking and writes the merged index atomically. Karthik Nayak agreed today that the series is already in good shape after Tobler said the remaining rename and error-message suggestions were not worth a reroll.

### Root refs vs pseudorefs documentation

*Long thread (root 2026/10/22/21-28-48): 22 replies so far from 4 participants over 13 days; 5 messages that day.*

Patrick Steinhardt remained unconvinced by the storage-focused pseudoref definition in the v2 series and proposed one emphasizing different semantics: pseudorefs can record multiple object IDs, carry extra metadata, and are not handled by the ref backend. Julia Evans suggested describing `FETCH_HEAD` with a concrete example entry. Junio C Hamano preferred Steinhardt's definition, suggesting "additional" for "different" and "extra" for "carry additional".

### Ejecting unreviewed topics from `seen`

*Long thread (root 2026/07/27/03-09-12): 22 replies so far from 5 participants over 100 days; 3 messages that day.*

Patrick Steinhardt proposed rewording "Will merge to 'next'?" to "Will merge to next [on <day>] unless told otherwise" and setting the `seen` entry bar at one favorable review of the overall direction. Junio C Hamano rejected the rewording as "quite a mouthful", said authors must ensure their topics mesh with in-flight work until a topic attracts interest, and defined the bar as a trusted reviewer saying the topic has potential to be a reasonable change, not a drive-by comment.

## The day in brief
Patrick Ohly reported a regression breaking `git merge` in blobless partial clones since 5946a564, with Kubernetes reverting to 2.52.0. Junio C Hamano marked the early-option-scanning series for `next` and queued the pack-objects `size_t` fix, while review continued on the `git repo info` path keys, the `%gb` reflog placeholder, and the quarantine alternate-refs fix.

## Notable threads

### Lazy fetch in partial clone fails with "not our ref"
Patrick Ohly reported that `git merge` in a blobless partial clone fails with "not our ref" while lazily fetching a missing object, breaking Kubernetes CI PR merges; he bisected v2.52.0 (good) to v2.54.0 (bad), identified 5946a564 ("odb/source: make read_object_info() pluggable") as first bad, and guessed the new `OBJECT_INFO_SECOND_READ` reprepare path triggers the promisor fetch differently. Kubernetes is reverting to 2.52.0.

### Standardizing early option scanning

*Long thread (root 2026/09/02/16-10-41): 20 replies so far from 4 participants over 63 days; 2 messages that day.*

The v4 series adds an "early-scan" sub-API to parse-options so commands can skip option values in hand-rolled early scans. Kaartic Sivaraam said it looks good, and Junio C Hamano marked the topic for `next`.

### Cleaning up repository config handling
Tian Yuchen's cleanup series for `environment.c`/`environment.h` config getters got Patrick Steinhardt's endorsement of Junio's proposed freeze of `struct repo_config_values`, which he said would save pushback and newcomer frustration.

### `size_t` for streamed large blobs in pack-objects
Elia Pinto's patch fixes truncation of blobs of 4GiB or more in the pack header on platforms where `long` is 32 bits but `size_t` is 64 bits (e.g. 64-bit Windows), which could corrupt objects and cause data loss on repack. Junio C Hamano found the explanation sensible and queued it.

### Not listing own refs as alternates in the quarantine
Kristofer Karlsson's patch stops `rev-list --alternate-refs` inside receive-pack's quarantine from listing the repository's own hidden refs. Junio queued it with a trial merge into `seen`; Patrick Steinhardt flagged that `is_borrowed_object_dir` misses object-storage-format extensions, and Justin Tobler proposed a short-term "read-only" mode for `odb_transaction_env()` that leaves the primary object directory unmodified.

## In brief
- herr.kaste posted a patch fixing `git log --graph` indenting visual roots that have columns to their right, breaking the drawing; Pablo Sabater reviewed it as good after a properly formatted resend.
- Patrick Peroša reported bare `--force-with-lease` is ignored when pushing to a remote group, causing non-fast-forward rejections on all group remotes.
- Junio C Hamano rejected moving `diff_auto_refresh_index` into `repo_config_values`, since the value comes from synthesized non-repository-specific config.
- Patrick Steinhardt explained his pushback on AI-looking commit messages: they are the remaining signal that a human understood the change.
- Linus Torvalds's patch fixes `git describe` miscounting distance with skewed commit dates; he accepted an opt-out `--no-exact` flag with a warning when no commit graph exists.
- Review continued on Farid Zakaria's credential-cache daemon race fixes; Junio questioned mixing `unix_ss_*` with tempfile cleanup.
- Kristofer Karlsson's patch makes the connectivity check ignore replace refs like fsck and pack-objects; Patrick Steinhardt aligned with the change.
- Okhuomon Ajayi's v2 removes `the_repository` from notes-utils.c; Karthik Nayak and Patrick Steinhardt said it looks good, while Junio noted notes-utils.h still reaches `the_repository` internally.
- Karthik Nayak declined to review v2 of the check-ref-format parse-options patch until the author engages with Junio's earlier review, per SubmittingPatches guidance.
- Ravi Mistry posted v2 of the HTTP retry series (exponential backoff, `http.retryStatusCodes`, post_rpc retries); Junio called patch 3 "looks good" and the series a pleasant read. *Long thread (root 2026/10/31/15-57-00): 8 replies so far from 2 participants over 4 days; 9 messages that day.*
- Aniket Gupta posted v2 making bundle-uri not die on unresolvable relative URIs, naming the failing URL with credentials stripped.
- Junio C Hamano queued Harald Nordgren's CI fixes for Ubuntu 26.04 failures (symlink workaround, linux32 pinned to ubuntu-24.04, apt-get).
