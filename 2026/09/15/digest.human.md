# Git mailing list daily digest for 2026/09/15

## Long threads

### receive-report hook for git-receive-pack

*Long thread: 58 replies so far from 7 participants over 29 days; 1 message that day.*

Karthik Nayak's series introduces a new `receive-report` hook in `builtin/receive-pack.c` that runs after the reference transaction completes but before the status report is sent to the client: the hook receives the pkt-line encoded report on stdin, its stdout replaces the report sent to the user, and a non-zero exit marks all references as rejected. Earlier, v10 fixed a leak when reassigning command strings in `override_cmds_error()`, and Junio C Hamano queued a "receive-pack: coccinelle fix" patch on top of the series that drops the nullness check before `FREE_AND_NULL`. Today Karthik Nayak said that coccinelle fix "looks good".

### unifying ref storage format terminology

*Long thread: 29 replies so far from 4 participants over 12 days; 2 messages that day.*

Patrick Steinhardt's 13-patch series renames the inconsistent ref storage format knobs -- `--ref-format=`, `GIT_REFERENCE_BACKEND`, `GIT_DEFAULT_REF_FORMAT`, `init.defaultRefFormat` -- to `ref-storage-format` equivalents across `git init`, `git clone`, `git refs`, submodule, rev-parse and help, keeping old names as hidden aliases and letting `--ref-storage-format=` accept a `format://payload` URI. Earlier reviewers debated quiet deprecation versus `OPT_ALIAS()`. Today Kaartic Sivaraam pointed out that existing deprecated aliases such as `--recursive` and `--negotiation-tip` are not hidden and still appear in help output, and Junio C Hamano explained that `OPT_HIDDEN` belongs to a later stage of the deprecation lifecycle, saying that if `OPT_ALIAS()` cannot take `OPT_HIDDEN` that is a bug in the infrastructure, to be marked with a `#leftoverbit`.

### http.sslVerifyStatus for stapled OCSP responses

*Long thread: 21 replies so far from 4 participants over 36 days; 1 message that day.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so git can check stapled OCSP responses, defaulting to false because enabling it blocks connections to servers that send no stapled response. Earlier rounds reworked the commit message and error text, and Patrick Steinhardt supplied OCSP test infrastructure. Today v7 was posted as the combined version discussed with Junio C Hamano and Patrick Steinhardt: tests needing no OCSP setup stay in `t5551` (run over https by `t5559`), everything needing a responder moves to the new `t5585`, the Apache OCSP block sets `DefaultRuntimeDir` so the stapling mutex stays in the server root, and `LIB_HTTPD_OCSP` support moved into lib-httpd so both test files share one prerequisite.

## The day in brief

The day's most consequential developments were v7 of the `http.sslVerifyStatus` patch arriving as the combined version with OCSP tests split across two files, v5 of the `--force-if-includes` push series fixing fast-forward rejections, and Karthik Nayak saying the `receive-report` hook series' coccinelle fix "looks good".

## Notable threads

### --force-if-includes fast-forward fix in push
Tyler Cipriani posted v5 of the series checking the pushed ref for `--force-if-includes`. The new third patch defers the reflog verification until after determining whether a force is needed, so valid fast-forward pushes are no longer rejected when the local reflog is expired -- an existing bug the earlier patches made worse. The series also makes `check_if_includes_upstream()` consult the reflog of the ref actually being pushed, explicitly rejecting non-branch pushes while allowing deletions, and adds a `pushRefUnverifiable` advice message suggesting `--force-with-lease=<ref>:<expect>` or `--no-force-if-includes`.

### repack losing objects to a mid-run ".keep"
The series fixes a race where `git repack -d` can delete a pack whose objects were never copied anywhere, because a concurrent push links a `.keep` onto an existing pack between repack's two scans. Today Justin Tobler reviewed the first patch, agreeing that a process should only delete a `.keep` it created itself, and suggested not registering `.keep` tempfiles before the ODB transaction commit -- instead letting the commit phase create the `.keep` explicitly and aborting if it already exists -- offering to send adjusted local patches from another series.

### stash.index autostash leaves a redundant stash entry
The bug report describes how, with `stash.index=true`, a successfully applied merge autostash is still stored as a stash entry and deleting `MERGE_AUTOSTASH` fails, because `git stash apply --index` reaches a reset mode that discards branch state via `remove_merge_branch_state()`. Today D. Ben Knoble reproduced the bug, confirmed the analysis lines up with the code, and said he is experimenting with replacing the `git reset` subprocess call with a direct `reset_working_tree()` call, while rejecting disabling `stash.index` for merge autostashes as papering over the underlying problem.

## In brief
- René Scharfe bisected the `git status --ignored` partial-pathspec match bug to commit 95c11ecc73 and proposed removing the `!excluded` optimization check in `treat_directory()`.
- Royce Remer posted a fix swapping the order of `send_wanted_ref_info()` and `send_shallow_info()` in `upload_pack_v2()` so shallow fetches with `uploadpack.allowRefInWant=true` work, arguing there is no backwards-compatibility concern because the combination never worked.
- Kaartic Sivaraam posted v4 fixing a commit-buffer leak in `builtin/history.c` by pairing `repo_logmsg_reencode()` with `repo_unuse_commit_buffer()`.
- Justin Tobler's two-patch series fixes packfile flush during ODB transaction commit with `core.fsyncMethod=batch`; Karthik Nayak reviewed both patches positively.
- Junio C Hamano reviewed Jiri Kuncar's NULL-guard patch keeping `git pull` from crashing on an unresolvable merge head, calling it a good improvement, and suggested simplifying the test's corrupt-object step.
- SZEDER Gábor posted v2 of the series precompiling `git-compat-util.h`, reporting about 35% faster builds.
