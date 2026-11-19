# Git mailing list daily digest for 2026/08/19

## Long threads

### Pluggable packfile writes for git-receive-pack(1)

*Long thread: 29 replies so far from 3 participants over 14 days; 1 message that day.*

Justin Tobler posted v4 of the series "builtin/receive-pack: support pluggable packfile writes", which introduces a generic `odb_transaction_write_pack()` transaction interface so that writing an incoming packfile is no longer tied to the "files" backend's child processes `git-index-pack(1)` and `git-unpack-objects(1)`. Changes since v3 include making the unpack limit consider `odb_transaction_flags`, adding a test assertion that the keep file is generated in the quarantine directory, replacing an include with a forward declaration, and updating commit messages. Earlier, Patrick Steinhardt had questioned the ".keep" file handling and urged using transaction flags such as `ODB_TRANSACTION_RECEIVE` to differentiate callers of the new interface, which this version now does.

## The day in brief
Justin Tobler's v4 makes receive-pack's packfile writes pluggable through a new ODB transaction interface. Patrick Steinhardt reworked corrupt-object handling to be backend-agnostic with a proper status enum. Karthik Nayak's report hook for receive-pack drew documentation and design refinements.

## Notable threads

### Generic OBJECT_INFO_DIE_IF_CORRUPT handling in the ODB
Patrick Steinhardt posted v2 of "odb: handle `OBJECT_INFO_DIE_IF_CORRUPT` generically", restructuring the series around `enum odb_read_status` with negative error codes (`ODB_READ_OK`, `ODB_READ_ERROR`, `ODB_READ_NOT_FOUND`), adding an `errmsg` parameter to `read_object_info()` so sources can bubble up human-readable corruption messages, and making the die-if-corrupt check backend-agnostic instead of relying on `has_packed_and_bad()`. Junio C Hamano accepted the explanation but noted the reasoning relies on the invariant that a single pack contains each object at most once.

### Recovering when a multi-pack-index names a removed pack
Junio C Hamano reviewed Elijah Newren's two-patch series on objects treated as missing due to a race with geometric repacking. On the replay fix he confirmed the diagnosis -- any value other than `0 < result->clean` is a failure, but the code mistakenly treated any non-zero value as OK -- and gave the verdict "Well spotted and fixed." On the packfile patch he said he will prepare an evil-merge to adapt a `packfile_fill_entry()` call to an API change from another in-flight topic.

### Report hook for git-receive-pack(1)
Karthik Nayak's patch introduces a `report` hook that receives the pkt-line encoded status report on stdin and whose stdout replaces the report sent to the client, with a non-zero exit causing receive-pack to die. Kristoffer Haugsbakk and Patrick Steinhardt suggested documenting the hook in `githooks(5)` and `git-receive-pack(1)`, adding the GitLab MVCC use case to the commit message, and explaining why existing hooks do not fit; Nayak agreed to most of these.

## In brief
- Elijah Newren identified a potential use-after-scope bug in `odb_source_files_generate_pack()` where `clean_on_exit` retains a pointer to a function-local struct; Justin Tobler asked for confirmation that git-bundle(1) generating packfiles via the ODB interface is meant to support non-"files" backends.
- Karthik Nayak posted a reftable series avoiding stack reloads while locked, keeping `newfstatat()` calls constant regardless of ref count; Junio C Hamano raised an unresolved concern that a second `reftable_addition` on a locked stack would release a shared lock.
- **Patrick backs Mailhol's v3; Junio plans evil-merge**: Patrick Steinhardt is "happy with this version" of Vincent Mailhol's `git history` Bash completion, calling it "a good base to build on"; Junio C Hamano will evil-merge a test adding 'squash' for a subcommand from another in-flight topic.

- D. Ben Knoble proposed compiling `cfg->use_nanosec` checks unconditionally under `NO_NSEC`; Junio C Hamano advised against a warning for setups sharing `$HOME/.gitconfig` across machines.
- Patrick Steinhardt posted v2 1/5 flagging known-bad packed objects as corrupt rather than missing via a `bad_pack` out-parameter.
- Yoichi Nakayama proposed restructuring the `ac == 2` conditional in `git worktree add` to stop DWIM when `-b` or `-B` is given.
- Yoichi NAKAYAMA posted v5 improving ambiguous remote branch messages in checkout and worktree add, now listing the matched remotes.
- Junio C Hamano declined Kenneth Lorber's config editor-failure patch as already covered by an in-flight change, and advised against `test_env` in the tests.
