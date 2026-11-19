# Git mailing list daily digest for 2026/09/08

## Long threads

### history: add squash subcommand to fold a range

*Long thread: 91 replies so far from 9 participants over 87 days; 2 messages that day.*

Harald Nordgren posted v15 of the series adding `git history squash <revision-range>` to fold a range of commits in `builtin/history.c`, fixing `istarts_with()` -> `starts_with()` in `squash_check_autosquash_subject()` so `fixup! ABCdef` mixed-case hex is no longer accepted, fixing a `squshed` typo, and adding `squash` to the `git history` subcommand completion test. Junio C Hamano replied that with the "complete 'git history'" topic now in, he no longer needs an evil merge for the `t9902-completion.sh` change.

### hook: introduce the receive-report hook

*Long thread: 46 replies so far from 5 participants over 22 days; 6 messages that day.*

Karthik Nayak posted v8 of the series introducing the receive-report hook in `builtin/receive-pack.c`, which runs after the reference transaction completes but before the report reaches the client; its stdout replaces the report and a non-zero exit marks all references rejected. v8 fixes the `UNKOWN` spelling, removes a stale comment, and passes `version` instead of `false` when regenerating the report after hook failure. Junio C Hamano objected to the `BUG()` in the switch's default case, since `version` comes from the peer's report-status requests, and proposed `case REPORT_STATUS_UNKNOWN: break;` instead.

### Introduce uploadpack.lazyFetchTrusted

*Long thread: 23 replies so far from 3 participants over 61 days; 10 messages that day.*

Christian Couder posted v3 of the series adding the protected `uploadpack.lazyFetchTrusted` config variable (global/system scope only) that sets `GIT_NO_LAZY_FETCH=0` in `builtin/upload-pack.c` for trusted repos. v3 reworks `path_allowlist_apply()` around a caller-supplied `allow_path` callback, identifies repos by git dir with absolute paths only, and adds `GIT_INTERNAL_LAZY_FETCH_DEPTH` (max 5) to stop infinite recursion when a promisor remote resolves back to the served repo. Junio C Hamano reviewed the patches, preferring a plain `int` depth counter and questioning a config variable overriding `GIT_NO_LAZY_FETCH`.

### sequencer: leave auto maintenance to the end of a sequence

*Long thread: 22 replies so far from 5 participants over 5 days; 5 messages that day.*

Thomas Bachem posted v3 of the series leaving auto maintenance to the end of a sequence: rebase, cherry-pick and revert now run `git maintenance run --auto` once when done, from `builtin/rebase.c` and `builtin/revert.c` instead of inside the sequencer, while spawned commands receive `maintenance.auto=false` via `GIT_CONFIG_PARAMETERS`. v3 drops `gc.auto=0`, builds the config string once as a `char *`, and extends the tests. Junio C Hamano said "Will queue"; Kristoffer Haugsbakk backed his complaint that commit messages are accumulating irrelevant detail.

### http: add http.sslVerifyStatus to check stapled OCSP responses

*Long thread: 21 replies so far from 4 participants over 29 days; 1 message that day.*

Grayson Gordon's patch adds the `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` in the HTTP transport, defaulting to false so servers without stapled OCSP responses are not blocked. Earlier, Patrick Steinhardt supplied OCSP test infrastructure and Junio C Hamano concluded a combined patch taking the best of both test sets is wanted. Today Grayson Gordon said he would assemble the combined patch and send it over.

## The day in brief

The most consequential developments: v15 of the `git history squash` series fixed `istarts_with()` to `starts_with()` so mixed-case hex in `fixup!` subjects is no longer accepted; Taylor Blau diagnosed a bitmap bug where `git bundle create` omits a tree required by an advertised ref and proposed a fix; and v3 of the sequencer auto-maintenance series, which Junio C Hamano said he would queue, moved the maintenance run into the rebase and revert builtins.

## Notable threads

### git bundle create with bitmaps omits tree required by advertised ref
Peter Elmers reported that `git bundle create` with bitmaps omits a tree required by an advertised ref when the included commit shares a root tree with an excluded sibling; `-c pack.useBitmaps=false` works around it. Taylor Blau reproduced the issue, diagnosed that the non-bitmap traversal overcounts objects past the boundary while the bitmap walk builds an exact answer, and proposed "bundle: restrict pack haves to recorded prerequisites" with a new bitmap test.

### push: fix --force-if-includes consulting wrong ref
Tyler Cipriani posted v2 fixing `--force-if-includes` in push: `check_if_includes_upstream()` now consults the pushed ref's reflog (`ref->peer_ref`) instead of the destination-named local branch, explicitly rejects detached HEAD, and skips deletions. Patch 2/2 adds `REF_STATUS_REJECT_UNVERIFIABLE` and advice pointing to `--force-with-lease=<ref>:<expect>` or `--no-force-if-includes` instead of the misleading `git pull` advice.

### var: support broken-down idents, signing key, multiple args, and -z
Andrew Pleeter posted v5 of the `git var` extension adding `GIT_AUTHOR_NAME`/`EMAIL`/`DATE`, committer equivalents, `GIT_SIGNING_KEY`, multiple variable arguments, and `-z`. v5 addresses Phillip Wood's review: `git_signing_key()` calls `get_signing_key()` directly, an unset variable in a multi-variable query prints an empty record and continues with exit code 1, and multi-valued variables get an extra delimiter. Junio C Hamano asked whether that delimiter also appears without `-z` and suggested `ident_part()` use `BUG()` on impossible cases.

## In brief
- Pia Park posted v2 of "midx-write: skip writes with no object entries", moving the empty-entries check so all write modes return silent success instead of publishing a zero-object MIDX that later breaks incremental bitmap writes; Taylor Blau found silent exit 0 acceptable.
- Junio C Hamano decided to queue Ted Nyman's "fetch-pack: trace packfile URI downloads", adding a Trace2 region around the packfile URI download loop in `fetch-pack.c`.
- Patrick Steinhardt questioned the `xmemdupz()` copy in the imap-send ASN1_STRING fix, suggesting `memchr`-based inspection bounded by the encoded length instead.
- Kaartic Sivaraam asked whether `git init`'s old `--ref-format=` should be deprecated or aliased to `--ref-storage-format`, and noted the "unknown ref storage format" error now prints the whole backend URI.
