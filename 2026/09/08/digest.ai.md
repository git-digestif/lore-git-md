# Git mailing list daily digest for 2026/09/08

## Long threads

### history: add squash subcommand to fold a range

*Long thread (root 2026/06/14/19-25-39): 91 replies so far from 9 participants over 87 days; 2 messages that day.*

Harald Nordgren posted v15 of the 8-patch series adding `git history squash <revision-range>` to `builtin/history.c`, fixing `istarts_with()` -> `starts_with()` in `squash_check_autosquash_subject()` so `fixup! ABCdef` mixed-case hex is no longer accepted, fixing a `squshed` -> `squashed` typo in the adjacent comment, and adding `squash Z` to the `git history` subcommand completion test. Earlier, the series had been split into patches covering parsing, range validation, branch protection, commit creation, and message editing, with extended discussion over case-insensitive hex matching. Junio C Hamano replied to patch 4/8 noting that with the "complete 'git history'" topic now in, he no longer needs to worry about making an evil merge for the `t9902-completion.sh` change.

### hook: introduce the receive-report hook

*Long thread (root 2026/08/18/07-55-55): 46 replies so far from 5 participants over 22 days; 6 messages that day.*

Karthik Nayak posted v8 of the 4-patch series adding a receive-report hook to `builtin/receive-pack.c` that runs after the reference transaction completes but before the report is sent to the client; the hook receives the pkt-line encoded report on stdin, its stdout replaces the report, and a non-zero exit marks all references as rejected. v8 fixes the `UNKOWN` -> `UNKNOWN` spelling, removes a stale comment, and passes `version` instead of `false` when regenerating the report after hook failure. Earlier iterations introduced `enum report_status_version` replacing static variables and merged `report()`/`report_v2()` into a shared `generate_report()`. Junio C Hamano objected to the `BUG("unknown report status version")` in the default case of the new switch in v8 2/4, arguing `version` is set from the peer's report-status feature requests rather than by our own code, and proposed `case REPORT_STATUS_UNKNOWN: break;` so nothing is reported when the client requested no report-status.

### Introduce uploadpack.lazyFetchTrusted

*Long thread (root 2026/07/10/08-51-34): 23 replies so far from 3 participants over 61 days; 10 messages that day.*

Christian Couder posted v3 of the 5-patch series adding a protected `uploadpack.lazyFetchTrusted` config variable (global/system scope only) that lets server operators mark repos trusted for lazy fetching, causing `builtin/upload-pack` to set `GIT_NO_LAZY_FETCH=0` for them. v3 squashes the earlier allowlist patches into a reworked `path_allowlist_apply()`/`path_allowlist_config_apply()` with a caller-supplied `allow_path` callback, identifies a served repo by its git dir (not worktree) accepting only absolute paths, and adds `GIT_INTERNAL_LAZY_FETCH_DEPTH` with a max of 5 to stop infinite recursion when a promisor remote resolves back to the served repo. Junio C Hamano reviewed the v3 patches, suggesting a fuller comment documenting `try_promisor_remotes()`'s return value, noting the silent ignoring of missing allowlist paths, preferring a plain `int` over `unsigned long` for the depth counter, and questioning a config variable overriding `GIT_NO_LAZY_FETCH`.

### sequencer: leave auto maintenance to the end of a sequence

*Long thread (root 2026/09/04/07-53-44): 22 replies so far from 5 participants over 5 days; 5 messages that day.*

Thomas Bachem posted v3 of the 3-patch series moving auto maintenance out of the sequencer: it now runs from `builtin/rebase.c` and `builtin/revert.c` once the command is done, so rebase, cherry-pick, and revert each run `git maintenance run --auto` exactly once at the end, while sequencer-spawned commands (git commit, git merge, exec) receive `maintenance.auto=false` via `GIT_CONFIG_PARAMETERS` to avoid racing the sequence for `MERGE_RR.lock` or deleting open packs. v3 drops `gc.auto=0`, builds `config_parameters` once as a `char *` via `strbuf_detach()`, and extends patch 2's tests. Phillip Wood had endorsed moving the call into the builtins; Patrick Steinhardt replied "Maybe" and asked for commit-message rationale. Junio C Hamano said "Will queue" for the v3 series, separately complaining that proposed commit messages are getting longer with more irrelevant detail.

### http: add http.sslVerifyStatus to check stapled OCSP responses

*Long thread (root 2026/08/11/17-02-00): 21 replies so far from 4 participants over 29 days; 1 message that day.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean in the HTTP transport mapping to `CURLOPT_SSL_VERIFYSTATUS`, defaulting to false because enabling it would block connections to legitimate servers that do not staple OCSP responses. Earlier, Patrick Steinhardt supplied comprehensive OCSP stapling test infrastructure, and Junio C Hamano concluded that a combined patch taking the best parts of both test approaches is what they want. Today Grayson Gordon said he would put together the combined patch and send it over.

## The day in brief
Several long-running series advanced: v15 of `git history squash`, v8 of the receive-report hook, v3 of `uploadpack.lazyFetchTrusted`, and v3 of the sequencer auto-maintenance change, which Junio C Hamano said he would queue. Taylor Blau diagnosed a bitmap bug in which `git bundle create` omits a tree required by an advertised ref and proposed a fix.

## Notable threads

### bundle: restrict pack haves to recorded prerequisites
Peter Elmers reported that `git bundle create` with bitmaps omits a tree required by an advertised ref when the included commit shares a root tree with an excluded sibling commit, providing a reproduction script; `-c pack.useBitmaps=false` appears to fix it and `pack.useBitmapBoundaryTraversal=true` did not prevent the omission. Taylor Blau reproduced the issue, diagnosed that the non-bitmap traversal overcounts objects beyond the boundary while the bitmap walk builds an exact answer, and proposed a lightly tested patch "bundle: restrict pack haves to recorded prerequisites" restricting pack haves to UNINTERESTING commits marked BOUNDARY, noting test fallout in t6020.13 where the expected object count rises from 3 to 40, plus a new test for the bitmap case.

### push: fix --force-if-includes consulting wrong ref
Tyler Cipriani posted v2 of the series fixing `--force-if-includes` checking the reflog of the destination-named local branch regardless of what is being pushed. Patch 1/2 makes `check_if_includes_upstream()` consult the pushed ref's `ref->peer_ref`, explicitly rejects detached HEAD, and skips deletions; patch 2/2 introduces `REF_STATUS_REJECT_UNVERIFIABLE` and a new advice message for detached HEAD rejections directing users to `--force-with-lease=<ref>:<expect>` or `--no-force-if-includes` instead of the misleading `git pull` advice.

### midx-write: skip writes with no object entries
Pia Park posted v2 of the patch fixing MIDX writes that find packs but no new objects publishing a zero-object layer without a reverse index, which makes a subsequent `write --incremental --bitmap` exit 255 with "could not load reverse index for MIDX". v2 moves the no-entries check earlier in `write_midx_internal()` so it applies to all write modes, returning silent success (exit 0) before locking or creating a temp file. Taylor Blau reviewed v2, saying silent exit 0 is acceptable but recommending the no-packs case also return zero, and suggesting test simplifications.

### imap-send: ASN1_STRING handling fixes
Beat Bolli's 3-patch series future-proofs `imap-send.c` for OpenSSL 4.1, fixes an incorrect assumption that ASN1_STRINGs are NUL-terminated, and checks the certificate subject common name only when no DNS subject alternative names are present. Junio C Hamano pointed out a bug in the proposed `host_matches()` fix where advancing `pattern` by 2 would make `free(pattern)` invalid, suggesting a separate `pattern_to_free` variable. brian m. carlson argued TLS should use subjectAltName rather than the CN and asked why git still does hostname verification by hand instead of relying on OpenSSL's built-in verification. Patrick Steinhardt questioned the need to switch to the renamed `ASN1_STRING_get_length()` and suggested `memchr`-based inspection bounded by the encoded length instead of `xmemdupz()`.

### push: fix --force-if-includes when remote-tracking ref has no reflog
The series fixes `is_reachable_in_reflog()` reading an uninitialized timestamp when the remote-tracking ref has no reflog, which stops the walk at the first entry and causes a false "remote ref updated since checkout" rejection after a fresh clone. Junio C Hamano challenged the justification for the fallback cutoff value, arguing that citing behavior affecting only users who consciously disable gc is a poor excuse for imposing it on all users.

## In brief
- Andrew Pleeter posted v5 of "var: support broken-down idents, signing key, multiple args, and -z", addressing Phillip Wood's review: `git_signing_key()` now calls `get_signing_key()` directly, an unset variable in a multi-variable query prints an empty record and processing continues with exit code 1, and multi-valued storage uses internal NUL delimiters. Junio C Hamano reviewed v5, asking whether the extra delimiter after multi-valued variables also appears without `-z` and suggesting a `string_list` refactor for the NUL-delimited iteration.
- Justin Tobler approved the direction of the "odb: write alternates at creation time" series, suggesting documentation for the new `create_repository()`, `create_reference_database()`, and `create_object_database()` functions, and briefly suggested then retracted a conditional change in the clone patch. *Long thread (root 2026/08/25/14-11-49): 20 replies so far from 4 participants over 15 days; 4 messages that day.*
- Justin Tobler reviewed the "odb: stop registering in-memory sources" series, asking for clearer wording in the submodule-config commit message about which repository is meant, and saying the multi-pack-index `--object-dir=` change looks good.
- Tuomas Ahola noted the "git worktree add deletes my entire directory" issue is being worked on, with a fix already queued to be merged to `master`.
- Kaartic Sivaraam reviewed the ref storage format terminology series, asking whether `git init` and `git clone`'s old `--ref-format=` option should be deprecated or marked as an alias via `OPT_ALIAS`, and pointing out that the "unknown ref storage format" error now prints the whole backend URI rather than just the format name.
