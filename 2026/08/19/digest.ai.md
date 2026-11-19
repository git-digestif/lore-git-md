# Git mailing list daily digest for 2026/08/19

## Long threads

### Pluggable packfile writes for git-receive-pack(1) via ODB transactions

*Long thread (root 2026/08/06/21-38-53): 29 replies so far from 3 participants over 14 days; 1 message that day.*

Justin Tobler posted v4 of the series "builtin/receive-pack: support pluggable packfile writes", which introduces a generic `odb_transaction_write_pack()` transaction interface so that git-receive-pack(1) can write incoming packfiles without being tied to the "files" backend; the existing logic that spawns git-index-pack(1) and git-unpack-objects(1) becomes the "files" backend implementation of that interface. Changes since v3 include making the unpack limit consider `odb_transaction_flags`, adding a test assertion that the keep file is generated in the quarantine directory, replacing an include with a forward declaration, and updating commit messages. The nine patches update `index_pack_lockfile()` to derive the ".keep" path from an explicitly provided ODB source, add `odb_transaction_finalize()` with an optional backend finalize callback, lift global state out of `unpack()`, resolve the unpack limit lazily, and move the `unpack()` logic into the "files" backend, which removes ".keep" files during finalize after commit. Earlier, Patrick Steinhardt questioned the ".keep" handling, and Justin Tobler proposed using transaction flags such as `ODB_TRANSACTION_RECEIVE` to differentiate callers of `odb_transaction_write_pack()`, which Patrick Steinhardt agreed with and v4 now implements.

## The day in brief

Justin Tobler posted v4 of the pluggable packfile-writes series for git-receive-pack(1), and Patrick Steinhardt posted v2 of a series making `OBJECT_INFO_DIE_IF_CORRUPT` handling backend-agnostic. Karthik Nayak's reftable stack series avoids redundant stack reloads during reference transactions, and his report hook for git-receive-pack(1) drew detailed review.

## Notable threads

### Pluggable packfile generation for the transport layer

Elijah Newren identified a potential use-after-scope bug in `odb_source_files_generate_pack()`: with `clean_on_exit = 1`, `mark_child_for_cleanup()` retains a pointer to the function-local `cp`, which `cleanup_children()` would dereference on abnormal exit. Justin Tobler asked for confirmation that having git-bundle(1) generate packfiles via the object database interface is meant to enable bundle creation on repositories using a different ODB backend, while the bundle still uses a packfile as the common format. The series makes the transport-layer parts of packfile generation pluggable.

### Recovering when a multi-pack-index names a removed pack

Junio C Hamano confirmed the diagnosis behind the replay fix -- any value other than `0 < result->clean` is a failure, but the code treated any non-zero value as OK -- and gave the verdict "Well spotted and fixed." He explained that a proposed `test_grep ! "[Ss]egmentation"` check is unnecessary because `test_must_fail` already requires controlled failure. On the packfile recovery patch, he said he will prepare an evil-merge to rewrite the `packfile_fill_entry(p, oid, e)` call to match an API change from another in-flight topic when merging to 'seen'. The series addresses objects treated as missing due to a race with geometric repacking, plus a git-replay(1) SIGSEGV on unreadable merge input.

### Report hook for git-receive-pack(1)

Karthik Nayak's patch introduces a 'report' hook that receives the pkt-line encoded status report on stdin and whose stdout replaces the report sent to the client; a non-zero exit makes receive-pack die. Reviewers Kristoffer Haugsbakk and Patrick Steinhardt suggested moving the GitLab MVCC context into the commit message, listing the hook in git-receive-pack(1) with githooks(5) as the canonical documentation, dropping "remote" from the description, and renaming a `code` variable to `ret`. Karthik Nayak agreed to most of these, and Patrick Steinhardt accepted a non-zero hook exit as a "nuclear option" for cases where failing the operation is the only remaining choice.

### Generic OBJECT_INFO_DIE_IF_CORRUPT handling in the ODB

Patrick Steinhardt posted v2 of "odb: handle `OBJECT_INFO_DIE_IF_CORRUPT` generically", restructuring the series around `enum odb_read_status` with negative error codes exclusively (`ODB_READ_OK`, `ODB_READ_ERROR`, `ODB_READ_NOT_FOUND`), adding an `errmsg` parameter to `read_object_info()` so sources can bubble up corruption messages, and making the die-if-corrupt check backend-agnostic instead of relying on `has_packed_and_bad()`. Junio C Hamano accepted the explanation that only the objects in a bad pack are marked, not the whole pack, but noted the reasoning relies on a single pack containing each object at most once.

### Avoiding reftable stack reloads while locked

Karthik Nayak posted a three-patch series for reftable/stack that skips reloading the stack when it is already locked inside a reference transaction, reporting that `newfstatat()` calls stay constant at about 55 regardless of ref count, versus linear growth before. Justin Tobler reviewed the patches, calling the lock move to `struct reftable_stack` "Looks good", but Junio C Hamano raised an unresolved concern: with the lock now per-`reftable_stack`, a second `reftable_addition` on an already-locked stack would fail `flock_acquire()`, and closing it would release the lock shared with other addition holders.

## In brief

- D. Ben Knoble and Patrick Steinhardt continued reviewing the runtime `core.useNanosec` conversion; Ben proposed compiling the `cfg->use_nanosec` checks unconditionally while ignoring the config under `NO_NSEC`, and Junio C Hamano decided against warning when the config is enabled under `NO_NSEC` because users sharing a `$HOME/.gitconfig` across machines would be annoyed.
- Patrick Steinhardt approved Vincent Mailhol's v3 Bash completion for `git history`, calling it a good base to build on; Junio C Hamano said he will evil-merge a test adjustment for a new subcommand from another in-flight topic.
- Yoichi NAKAYAMA posted v5 of the series improving messages for ambiguous remote branch names in `git checkout` and `git worktree add`, now listing matched remote names via a new `advise_ambiguous_remote()`; D. Ben Knoble suggested renaming a helper and splitting the refactoring from the message changes.
- Lutz Lengemann posted v2 of the zsh completion fix for `git -C <path>`, now walking all global options to locate the command, which also fixes completion after options like `git -p checkout`.
- Patrick Steinhardt asked for more accurate wording in Grayson Gordon's `http.sslVerifyStatus` OCSP patch and supplied comprehensive OCSP stapling test infrastructure, while questioning whether the complexity is worth it.
- Patrick Steinhardt posted v2 patch 1/5 flagging known-bad packed objects as corrupt rather than missing by threading a `struct packed_git **bad_pack` out-parameter through the pack lookup path.
- Yoichi Nakayama proposed restructuring a conditional in `git worktree add` to reduce nesting in the fix that stops DWIM when `-b` or `-B` is given.
- Junio C Hamano declined Kenneth Lorber's patch surfacing editor failure in `git config -e` exit codes, noting an equivalent change is already in flight, and advised against `test_env` in the tests.
- Patrick Steinhardt and Junio C Hamano approved Nikolaus Schuetz's t1402 tests for forbidden characters in refnames, agreeing they protect authors of new backends.
- Friel posted v2 of a patch adding a `write_pack_file/wrote_bytes` Trace2 datum to `builtin/pack-objects.c` reporting total pack bytes written.
- Patrick Steinhardt traced the 6700 loose-object threshold for geometric repacking to commit 2c3c439947 (2007), chosen so git-gc(1) would not trigger for a fully unpacked Git v0.99 but would for v1.0.
- Yuri reported that `git stash push` is not atomic: on failure over a file exceeding the diff size limit it leaves both an invalid stash and working-directory changes, and asked for more informative error messages.
