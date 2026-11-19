# Git mailing list daily digest for 2026/09/07

## Long threads

### receive-report hook for git-receive-pack

*Long thread: 41 replies so far from 5 participants over 21 days; 4 messages that day.*

Karthik Nayak's series adds a `receive-report` hook in `builtin/receive-pack.c` that runs after the reference transaction completes but before the report goes to the client: it receives the pkt-line encoded report on stdin, its stdout replaces the report, and a non-zero exit rejects all references. Earlier revisions had settled the `generate_report()` refactor and `enum report_status_version`, and v7 dropped the now-redundant `report_v2()`. Patrick Steinhardt said he is happy with v7, while Junio C Hamano flagged that patch 4/4 passes `false` to `generate_report()` instead of the `version` variable, that a comment in patch 3/4 no longer matches the interface, and that `REPORT_STATUS_UNKOWN` in patch 2/4 is missing the letter 'N'.

### Git 3.0: restrict hex object IDs to lowercase only

*Long thread: 26 replies so far from 6 participants over 41 days; 2 messages that day.*

brian m. carlsson's series removes Git's acceptance of uppercase hex in object IDs for Git 3.0, since Git has always emitted lowercase and software assuming a unique hex form can break or be exploited by uppercase input; earlier discussion established that policy checks against the lowercase form can be bypassed and regexps like `[0-9a-f]{40}` fail to match uppercase IDs. Phillip Wood said the object-ID-versus-ref case made more sense to him and wondered whether refnames whose last component looks like an object ID should also be forbidden, and brian m. carlsson posted v2, fixing a hex-range comment, updating commit messages, moving the t5324 fixes to a separate commit, and restoring lowercasing in `parse_oid_prefix`.

### sequencer: run auto maintenance once a sequence is done

*Long thread: 18 replies so far from 4 participants over 4 days; 13 messages that day.*

Thomas Bachem's series makes rebase, cherry-pick and revert each end their sequence with one `git maintenance run --auto`, passing `maintenance.auto=false` and `gc.auto=0` to the `git commit`, `git merge` and exec commands a sequence spawns, via a new `git_config_append_parameter()` helper in `config.c`. Patrick Steinhardt asked what `GIT_CONFIG_PARAMETERS` is and whether both config keys are needed, and Phillip Wood suggested rewording commit messages and extending existing tests. Thomas Bachem confirmed every finishing sequence now runs auto maintenance where a conflict-free one previously did not, agreed to drop `gc.auto=0`, document the `GIT_CONFIG_PARAMETERS` format, and move the call out of the sequencer into `builtin/rebase.c` and `builtin/revert.c`, which Phillip Wood endorsed.

## The day in brief
The receive-report hook series reached a v7 Patrick Steinhardt is happy with, modulo three nits from Junio C Hamano. brian m. carlsson rerolled the Git 3.0 lowercase-hex series as v2, and the sequencer auto-maintenance series absorbed feedback on where the maintenance call should live. The post-2.56 version discussion weighed GitHub SHA-256 support as the main open question for a 3.0 bump.

## Notable threads

### git pull overwrites a local directory with a symlink
AIKSXD ax reported that with a `.gitignore` pattern like `dataset/` and a committed symlink of the same name, `git pull` on another machine silently replaces the local directory with the symlink, destroying its contents. brian m. carlsson explained that a trailing-slash pattern ignores everything under the directory rather than the entry itself, so it does not ignore the symlink, and that `git checkout` deletes ignored files and directories by default; he argued both proposed fixes would be substantial behavior changes and recommended not keeping valuable data in untracked or ignored files in the working tree.

### worktree lifecycle hooks
Domen Kožar's v2 series adds `post-worktree-add`, `post-worktree-remove` and `post-worktree-move` hooks in `builtin/worktree.c` so tools like devenv can provision and clean up per-worktree environments as agents create and discard worktrees. Kožar pinged the single-`post-worktree` proposal, and Kristoffer Haugsbakk proposed instead a `--id=<string>` option on `git worktree add` recording ownership so commands without that ID leave owned worktrees alone. Kožar replied that his case has two separate participants -- a tool creating worktrees and devenv observing the lifecycle no matter which tool invokes Git -- and asked how devenv would learn of an add or remove under the ownership-ID scheme.

### What will come after Git 2.56?
Junio C Hamano's question of whether the release after 2.56 should be 3.0, 2.99, or business as usual drew concrete positions. Patrick Steinhardt noted a Git 2.100 is also possible after 2.99, called GitHub's missing SHA-256 support the biggest question mark for a 3.0 bump, and reported that reftable support is upstreamed in libgit2 with SHA-256 default-enabled, so both blockers should clear with the next libgit2 release. Emily Shaffer said GitHub SHA-256 support leaves her most concerned, noted GitOxide has initial SHA-256 support, and said Google would disable SHA-256 for new repos via system config, making her lean toward releasing 3.0.

## In brief
- Paul DE TEMMERMAN reported that `git worktree add "" HEAD` on Windows deletes the entire workspace including `.git` instead of failing safely.
- Patrick Steinhardt posted v3 of "odb: write alternates at creation time", refactoring `init_db()` into `create_repository()` and dropping the unused `odb_source_write_alternate()` callback.
- Eli Barzilay reported that with `stash.index=true` a successfully applied merge autostash is still stored as a redundant stash entry and deleting `MERGE_AUTOSTASH` fails.
- Patrick Steinhardt posted v2 of the ref-storage-format terminology cleanup, renaming options to `--ref-storage-format=` and accepting URIs as a payload.
- Patrick Steinhardt proposed a fix for the `history reword` memory leak in `builtin/history.c`; Kaartic Sivaraam explained why the earlier similar patch was not merged.
- Beat Bolli posted a three-patch `imap-send` series covering OpenSSL 4.1 preparation, ASN1_STRING NUL-termination, and RFC 6125 common-name checking.
