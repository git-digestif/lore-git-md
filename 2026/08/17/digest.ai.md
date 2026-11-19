# Git mailing list daily digest for 2026/08/17

## Long threads

### `git history squash` subcommand series

*Long thread (root 2026/06/14/19-25-39): 85 replies so far from 9 participants over 65 days; 7 messages that day.*

Harald Nordgren's series adds `git history squash <revision-range>` to fold a range of commits into one, splitting the implementation into patches for parsing, range validation, branch protection, commit creation, and message editing. Today Phillip Wood pointed out that several patches are missing his Signed-off-by from the fixup patches he sent and that the sign-off should sit above his Helped-by to reflect the chain of custody. A discussion of the `istarts_with()` use in a test followed: Junio C Hamano corrected his earlier framing, saying the case-insensitive comparison is new code introduced by this series and there is no reason to be case-insensitive there, and agreed the `| tr` must go for the topic to play nicely with the uppercase-hex restriction. Harald Nordgren asked whether to use both headers, and Kristoffer Haugsbakk advised the order Signed-off-by: Phillip Wood, Helped-by: Phillip Wood, Signed-off-by: Harald Nordgren.

### Moving libgit.a sources into a `lib/` directory

*Long thread (root 2026/04/16/13-24-29): 38 replies so far from 12 participants over 124 days; 1 message that day.*

Patrick Steinhardt's RFC series moves the libgit.a sources into a separate `lib/` directory so the top level exposes entry points like README.md and build files more clearly. Earlier, Michael Montalbo proposed a script-based method to group the 231 libgit.a source files into directories using weighted commit-history labels and a cscope call-graph check. Today Junio C Hamano observed that the earliest group in that list (connect.c, fetch-pack.c, remote-curl.c, send-pack.c, transport.c, upload-pack.c) forms the "transport" group and matches intuition, while the other groups may not be clear-cut; he called the thought experiment interesting, suggested carving out a new directory for a conceptually clean group without waiting to classify the remainder, and raised the tangential question of whether bisect and revision machinery can be separated so bisect stands on its own.

### Pluggable packfile writes in `receive-pack`

*Long thread (root 2026/08/06/21-38-53): 29 replies so far from 3 participants over 12 days; 1 message that day.*

Justin Tobler's series introduces a generic `odb_transaction_write_pack()` interface so `git-receive-pack(1)` can write incoming packfiles through ODB transactions, with the existing `git-index-pack(1)`/`git-unpack-objects(1)` logic becoming the "files" backend implementation. Today Patrick Steinhardt agreed with Justin Tobler's earlier suggestion to start using transaction flags such as `ODB_TRANSACTION_RECEIVE` in the current series to differentiate callers of `odb_transaction_write_pack()`, noting that doing so makes it more obvious how future series will look.

### Eagerly loading ODB alternates

*Long thread (root 2026/08/10/13-33-27): 26 replies so far from 5 participants over 8 days; 8 messages that day.*

Patrick Steinhardt's series makes the object database eagerly initialize alternates instead of deferring them, removing scattered `odb_prepare_alternates()` call sites and dropping the `loaded_alternates` and `alternates_db` fields. Today he revised his position on case-insensitive deduplication of alternate paths: the metric does not resolve symlinks or mountpoints, and the stat()-based check is ineffective on Windows, so he suggested leaving the case-sensitivity change out of this patch and documenting it as a NEEDSWORK area. Jeff King agreed unconditional case-sensitive matching is his preference too, but flagged a timing hazard in caching `core.ignoreCase` before the repository is fully configured and suggested `repo_ignore_case()`; Patrick Steinhardt acknowledged the gotcha, said he will cherry-pick that fix, and posted v3, which adds a first patch creating the ref and object databases after the complete repository configuration is written in `init_db()`. Jeff King confirmed `repo_ignore_case()` is not yet usable here because `repo_config_values()` BUG()s on anything but `the_repository`.

### Packfile loading performance with many packs

*Long thread (root 2026/08/12/19-11-09): 22 replies so far from 5 participants over 6 days; 2 messages that day.*

Johannes Schindelin's patch fixes the O(N^2) regression in loading many packfiles by adding a `skip_dup_check` parameter to `packfile_list_append()` so `packfile_store_add_pack()` can skip the linear scan for known-new packs. Today Patrick Steinhardt noted that `packfile_list_prepend()` has essentially the same duplicate-check problem, and Jeff King identified its calls in hot code paths, including the MRU adjustment from `find_pack_entry()`, as a possible candidate for the clone slowdown. Jeff King supplied a synthetic many-pack test showing `git rev-list --count foo` timings growing quadratically with pack count, cautioned that lookup across packs is itself linear, and concluded it still seems prudent for MRU updates to use constant-time movement within a doubly-linked list rather than an explicit duplicate check and removal.

## The day in brief
Junio C Hamano said he will queue Shlok Kulshreshtha's use-after-free fix in `object-name.c` and will replace Grayson Gordon's v4 OCSP patch. Patrick Steinhardt posted v3 of the eager-alternates series with a new setup patch, while Jeff King pressed for constant-time MRU updates in the packfile list to address the many-packs slowdown. Junio C Hamano rejected the proposed warn-and-continue behavior for a valueless `submodule.active`.

## Notable threads

### Use-after-free fix in `get_oid_with_context_1()`
Shlok Kulshreshtha's patch fixes a use-after-free in `object-name.c` where a relative `:<path>` argument is resolved into a heap buffer that is freed while `cp` still points at it; v3 collapses the two exits into one so the single `free(new_path)` happens after the last use, and adds a t1092 test for a relative path to a sparse directory. Junio C Hamano reviewed v3, called it nicely described, confirmed the fix matches the expected pattern, and said he will queue it.

### `http.sslVerifyStatus` for stapled OCSP responses
Grayson Gordon's patch adds `http.sslVerifyStatus` so git sets `CURLOPT_SSL_VERIFYSTATUS` and checks stapled OCSP responses instead of ignoring them. Today he submitted v4, moving the four new tests into t5551 keyed on `$HTTPD_PROTO` so they run only in the https pass, changing the prerequisite to `test_might_fail`, dropping a redundant control test, and moving the rationale into the commit log. Junio C Hamano reviewed v4, found the code and documentation unchanged and good, and said he will replace the patch.

### Warning on valueless `submodule.active`
Tilak Raaz's patch would swap to `repo_config_get_value_multi()` and warn instead of hard-erroring on a valueless `submodule.active`. Today Junio C Hamano questioned whether the hard error is a problem, rejected the warn-and-continue behavior because early error messages scroll away unnoticed, and explained that resolving a NEEDSWORK comment can validly mean deciding the suggestion is not sensible; he also flagged the `Signed-off-by` real-name requirement and asked for lines wrapped to roughly 70 columns.

### `[synopsis]` block in `git-format-rev` docs
Kristoffer Haugsbakk's documentation series applies a `[synopsis]` open block to the example in `Documentation/git-format-rev.adoc` so the `<subject>` placeholder gets emphasis styling, and quotes the subject placeholder consistently. Today he posted v4 correcting the example to an open block, reporting clean `make doc` and `make USE_ASCIIDOCTOR=1 doc` runs, and removing the previous Ack. Junio C Hamano said the series is hopefully now ready for `next`.

### Pluggable packfile generation, `bundle.c` cleanup
Patrick Steinhardt's series makes packfile generation pluggable for the transport layer via `odb_generate_pack()`. Today he posted v2, whose changelog notes it mostly removes dependencies on `the_repository` in `bundle.c`. Junio C Hamano objected that adding an `extern` declaration for `get_log_output_encoding()` defeats the purpose of dropping `USE_THE_REPOSITORY_VARIABLE` and asked for those hunks to be omitted.

## In brief
- Yoichi Nakayama's worktree-repair patch removes dead relative-path handling in `worktree.c`; Junio C Hamano criticized the commit message and proposed a `read_raw_gitfile()`-style function, which Yoichi Nakayama agreed to adopt after explaining the concrete mismatch when `worktree.useRelativePaths=false`.
- Lutz Lengemann posted v2 of zsh completion support for `git -C <path>`, walking all words before the command and skipping global options so `git -p checkout <TAB>` also completes; `-c` and `--git-dir <path>` gaps are left for separate patches.
- In the `format-rev` `--abbrev`/`--color`/`--date` series, Kristoffer Haugsbakk acknowledged the `BUG_ON_OPT_NEG` precondition sits after a potential NULL dereference and will add a patch moving validation to the start of the callbacks; Junio C Hamano agreed that is a good pattern.
- Friel submitted a patch adding a `write_pack_file/wrote_bytes` Trace2 datum to `builtin/pack-objects.c` reporting total pack bytes written, summed across split packs, with two new tests in `t/t5300-pack-object.sh`.
- Kenneth Lorber sent an RFC making `git config -e` exit non-zero when the editor fails; Junio C Hamano questioned the value since the file damage is already done, concluding "I dunno".
