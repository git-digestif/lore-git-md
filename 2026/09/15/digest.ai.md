# Git mailing list daily digest for 2026/09/15

## Long threads

### receive-report hook in git-receive-pack

*Long thread (root 2026/08/18/07-55-55): 58 replies so far from 7 participants over 29 days; 1 message that day.*

Karthik Nayak's four-patch series introduces a `receive-report` hook that runs after the reference transaction completes but before the report is sent to the client: the hook receives the pkt-line encoded report on stdin, its stdout replaces the report sent to the user, and a non-zero exit marks all references as rejected. Earlier, v10 fixed a leak in `override_cmds_error()` when reassigning command strings, and Junio C Hamano queued a "receive-pack: coccinelle fix" patch on top of the v10 series. Today Karthik Nayak said that coccinelle fix "looks good".

### unifying ref storage format terminology

*Long thread (root 2026/09/04/10-36-01): 29 replies so far from 4 participants over 12 days; 2 messages that day.*

Patrick Steinhardt's 13-patch series renames `--ref-format=`, `GIT_REFERENCE_BACKEND`, `GIT_DEFAULT_REF_FORMAT` and `init.defaultRefFormat` to `ref-storage-format` equivalents across `git init`, `git clone`, `git refs`, submodule, rev-parse and help, and lets `--ref-storage-format=` accept a `format://payload` URI, keeping old options as hidden aliases. Earlier, reviewers debated quiet deprecation versus `OPT_ALIAS()`. Today Kaartic Sivaraam noted that existing deprecated aliases such as `--recursive` and `--negotiation-tip` are not hidden and still appear in help output, and Junio C Hamano explained that `OPT_HIDDEN` is intended for a later deprecation stage, adding that if `OPT_ALIAS()` does not allow `OPT_HIDDEN` that is a bug in the infrastructure, which need not block the topic and could be marked with `#leftoverbit`.

### http.sslVerifyStatus for OCSP stapling

*Long thread (root 2026/08/11/17-02-00): 21 replies so far from 4 participants over 36 days; 1 message that day.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS`, so libcurl requests the OCSP "Certificate Status Request" extension and rejects certificates whose stapled response is not "good"; it defaults to false because enabling it would block connections to servers that send no stapled response. Earlier rounds reworked the commit message and documentation and debated OCSP test infrastructure. Today v7 was posted as the combined version discussed with Junio C Hamano and Patrick Steinhardt: tests needing no OCSP setup stay in `t5551` (which `t5559` runs over https), everything needing a responder moves to a new `t5585` with `LIB_HTTPD_OCSP` support in lib-httpd, the `SSL_VERIFYSTATUS` prerequisite moves into `lib-httpd.sh` so both files share it, and the Apache OCSP block sets `DefaultRuntimeDir` to keep the `SSLUseStapling` mutex in the server root.

## The day in brief
Tyler Cipriani posted v5 of the `--force-if-includes` push series, adding a fix so valid fast-forward pushes are no longer rejected. Grayson Gordon sent v7 of `http.sslVerifyStatus`, combining the succinct and full OCSP test approaches into `t5551` and a new `t5585`. Royce Remer posted a fix for `upload_pack_v2()` response ordering that broke shallow clones with `uploadpack.allowRefInWant=true`.

## Notable threads

### repack deleting packs kept by a mid-run ".keep"
A series fixes a race where `git repack -d` can delete a pack whose objects were never copied, because a concurrent push links a `.keep` onto an existing pack between repack's two scans. Today Justin Tobler analyzed the receive-pack side: a `.keep` registered as a tempfile must know its post-migration path to be deleted safely, two concurrent identical pushes generate the same `.keep` name with different messages, and he suggested not registering `.keep` tempfiles before the ODB transaction commit and instead letting the commit phase create the `.keep` explicitly, aborting if it already exists; he offered to send upstream patches doing that.

### push --force-if-includes checks and fast-forward fix
Tyler Cipriani posted v5 of the three-patch series checking the pushed ref for `--force-if-includes`. The new third patch defers the reflog verification until after determining whether a force is needed, so valid fast-forward pushes are not rejected when the local reflog is expired -- an existing bug the earlier patches made worse. Patch 1/3 makes `check_if_includes_upstream()` consult the reflog of the ref actually being pushed, rejecting non-branch pushes while allowing deletions, and 2/3 adds a `pushRefUnverifiable` advice suggesting `--force-with-lease=<ref>:<expect>` or `--no-force-if-includes`.

### upload-pack v2 shallow fetch with ref-in-want
Royce Remer posted a one-line fix in `upload-pack.c` swapping `send_wanted_ref_info()` and `send_shallow_info()` in `upload_pack_v2()` so shallow-info is sent before wanted-refs, plus two tests in `t/t5703-upload-pack-ref-in-want.sh`. Junio C Hamano asked why the new ordering is more correct and whether it would break existing clients; Remer explained that with `uploadpack.allowRefInWant=true` on his Gitea servers, `--depth` clones failed with "fatal: expected 'packfile', received 'shallow-info'", that the protocol documentation specifies the corrected ordering, and that there is no compatibility concern because this combination never worked before.

### git status --ignored matching partial pathspec names
A bug report shows `git status --porcelain --ignored -- ba` matching `bar/baz/` and `bar/quux/` when `bar/` is ignored. Today René Scharfe bisected the bug to commit 95c11ecc73 ("Fix error-prone fill_directory() API; make it only return matches"), confirmed the reproduction, and proposed removing the `!excluded` optimization check in `treat_directory()`, while noting the optimization might instead be repaired.

### stash.index=true autostash leaving a redundant entry
With `stash.index=true`, a successfully applied merge autostash is still stored as a stash entry and deleting `MERGE_AUTOSTASH` fails, because `git stash apply --index` reaches a reset mode that discards branch state. Today D. Ben Knoble reproduced the bug, confirmed the analysis, and said he is experimenting with replacing the `git reset` subprocess call with a direct `reset_working_tree()` call, while rejecting the alternative of disabling `stash.index` for merge autostashes as papering over the problem.

## In brief
- SZEDER Gábor posted v2 of "make: precompile git-compat-util.h", adding Makefile rules to precompile the header for all objects except `COMPAT_OBJS`, `REFTABLE_OBJS` and third-party sources, with a `NO_PRECOMPILED_HEADER` knob and a reported ~35% build speedup.
- Kaartic Sivaraam posted v4 of "builtin/history: unuse the commit buffer after use", adding the missing `repo_unuse_commit_buffer()` call that fixes a leak in `git history` not caught by leak-checking tests.
- Junio C Hamano reviewed v8 of Andrew Pleeter's `git var` extensions and asked for a split into three patches (a `-z` mode, multiple variable arguments, and the new identity/signing-key variables); Pleeter agreed to restructure for v9.
- Todd Zullinger posted v3 of the doc lint fixes for `git-refs` and `git-pack-refs`, and Junio C Hamano replied "Thanks, will replace."
- Justin Tobler's two-patch fix flushing a transaction packfile before migrating objects in `object-file.c` drew review from Karthik Nayak; Tobler explained the flush call already returns early when there is nothing to flush.
- Jiri Kuncar's patch adds NULL guards in `builtin/pull.c` so an unresolvable merge head drives a failing merge instead of a segfault; Junio C Hamano called it a good improvement and suggested simplifying the test.
- Harald Nordgren posted v3 of `range-diff --matched-only`, replacing the incompatible-options `die` with an explicit check and setting both `left_only` and `right_only` internally.
- Brigham Campbell's patch makes `contrib/contacts/git-contacts` read patch contents from stdin when argv is empty and stdin is not a terminal; Junio C Hamano suggested a flag-based implementation and a usage comment update.
- Phil Sainty reported that `GIT_WORK_TREE` is not exported for `post-checkout-hook` in a worktree, so `git rev-parse --show-toplevel` returns the current directory instead of the genuine toplevel.
- Junio C Hamano explained that the `+` refspec prefix acts like `--force` for remote-tracking updates but pruning is a separate concept, and could not reproduce the reported tag deletion.
- Pia Park followed up on the `midx-write` empty-layer patch, offering to prepare v3 with silent success in the no-pack case.
- In the "What's cooking" report, Junio C Hamano decided to use v4 of the `ks/history-commit-leakfix` topic.
