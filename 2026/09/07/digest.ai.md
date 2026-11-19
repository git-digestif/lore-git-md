# Git mailing list daily digest for 2026/09/07

## Long threads

### receive-report hook in builtin/receive-pack.c

*Long thread (root 2026/08/18/07-55-55): 41 replies so far from 5 participants over 21 days; 4 messages that day.*

Karthik Nayak's series introduces a `receive-report` hook that runs after the reference transaction completes but before the report is sent to the client: the hook receives the pkt-line encoded report on stdin, its stdout replaces the report sent to the user, and a non-zero exit marks all references as rejected. Earlier revisions reworked report generation into a shared `generate_report()` and an `enum report_status_version`. Today Patrick Steinhardt said he is happy with the v7 version. Junio C Hamano raised three small issues: patch 4/4 passes `false` to `generate_report()` instead of the `version` variable used in the preceding call, the second sentence of the comment in patch 3/4 is stale and no longer matches the `generate_report()` interface, and `REPORT_STATUS_UNKOWN` in patch 2/4 is missing the letter 'N'.

### restricting hex object IDs to lowercase in Git 3.0

*Long thread (root 2026/07/29/23-32-09): 26 replies so far from 6 participants over 41 days; 2 messages that day.*

brian m. carlsson's series removes the ability to use uppercase hex in object IDs in Git 3.0's breaking-changes mode, because software that assumes a unique lowercase hex form can break or be exploited when uppercase IDs are accepted. Today he posted [PATCH v2 0/7], whose changes from v1 include fixing an incorrect hex range in a comment, updating commit messages, moving the t5324 fixes to a separate commit with an expanded message, and restoring lowercasing in `parse_oid_prefix`. Phillip Wood said the explanation of the object-ID-versus-ref distinction security case made more sense to him and wondered whether Git should forbid refnames whose last component looks like an object ID.

### running auto maintenance at the end of a sequencer sequence

*Long thread (root 2026/09/04/07-53-44): 18 replies so far from 4 participants over 4 days; 13 messages that day.*

Thomas Bachem's v2 series makes rebase, cherry-pick and revert each run auto maintenance once when a sequence finishes, and keeps it out of the `git commit`, `git merge` and exec commands a sequence spawns, via a new `git_config_append_parameter()` helper in config.c. Today Patrick Steinhardt and Phillip Wood reviewed all three patches: Steinhardt asked what patch 2/3 actually changes and hoped for a single exit path shared by the backends, and questioned setting both `maintenance.auto` and `gc.auto`; Wood asked for clearer commit messages and test changes. Bachem confirmed the change means every finishing sequence now runs `git maintenance run --auto`, agreed to drop `gc.auto=0`, and proposed moving the `run_auto_maintenance()` call out of the sequencer into builtin/rebase.c and builtin/revert.c, which Wood endorsed.

## The day in brief
Patrick Steinhardt said he is happy with the v7 receive-report hook series, with only small nits raised by Junio C Hamano. brian m. carlsson posted v2 of the series restricting hex object IDs to lowercase in Git 3.0. Thomas Bachem's sequencer auto-maintenance series drew review and an endorsed proposal to move the maintenance call into the builtins.

## Notable threads

### lifecycle hooks for git worktree
Domen Kožar's series proposes `post-worktree-add`, `post-worktree-remove` and `post-worktree-move` hooks, motivated by tools like devenv that provision per-worktree environments when AI agents create and discard worktrees, since `post-checkout` does not fire for `--no-checkout` or `--orphan` and nothing fires on move or remove. Today Kožar pinged the list asking whether the interface addresses the concerns. Kristoffer Haugsbakk proposed an alternative: a `--id=<string>` option on `git worktree add` recording ownership so commands without that ID do nothing to owned worktrees. Kožar replied that devenv neither owns the worktrees nor controls the invoking commands, so it needs to observe the lifecycle regardless, and asked how it would learn of an add or remove under the ownership-ID scheme.

### what the release after Git 2.56 should be called
Junio C Hamano asked whether the next release should be Git 3.0, 2.99, 2.98 or 2.57, noting the BreakingChanges document should be reviewed for readiness. Today Patrick Steinhardt said GitHub's missing SHA-256 support is the biggest question mark and he would not want to trigger a 3.0 bump before GitHub supports it, though he noted reftable and SHA-256 support have been upstreamed into libgit2. Emily Shaffer said missing GitHub SHA-256 support is her main concern about landing 3.0, but that Google's distribution would disable SHA-256 for new repos via system config, making her lean toward releasing 3.0.

### git pull replacing a local directory with a symlink
AIKSXD ax reported that when a `.gitignore` pattern like `dataset/` is used and a symlink of the same name is committed, `git pull` on another machine silently replaces the local directory with the symlink, destroying its contents. brian m. carlsson explained that trailing-slash patterns ignore everything under a directory rather than the entry itself, so they do not ignore symlinks, and that `git checkout` deletes ignored files and directories by default; he argued both proposed fixes would be substantial behavior changes and recommended not keeping valuable data in untracked or ignored files in the working tree.

### git worktree add with an empty path deletes the workspace
Paul DE TEMMERMAN reported that `git worktree add "" HEAD` on Windows with git 2.55.0.windows.5 fails but deletes the entire workspace including the `.git` directory, instead of failing safely.

### writing alternates at object database creation time
Patrick Steinhardt's series refactors alternates handling so they are written only when a repository is created, removing the `odb_source_write_alternate()` callback. Today he posted [PATCH v3 0/9], which refactors `init_db()` into `create_repository()` with the skip flags dropped, defers object database creation in git-clone until after the clone URI is resolved, unifies alternate setup in `collect_alternates()`, and removes the now-unused write-alternate callback; he also answered Justin Tobler's questions, saying clone options such as `--shared` should be backend-specific.

## In brief
- Patrick Steinhardt posted [PATCH v2 00/11] renaming ref storage format options consistently (`--ref-storage-format=`, `GIT_REF_STORAGE_FORMAT`, `init.defaultRefStorageFormat`, old names kept as hidden aliases) and extending the option to accept URI payloads.
- Eli Barzilay reported that with `stash.index=true` a successfully applied merge autostash is still stored as a redundant stash entry and deleting `MERGE_AUTOSTASH` fails; a staged change is required to trigger it.
- On the rerere lock race, Patrick Steinhardt noted it also occurs between two normal writers, not just against `git rerere gc`, and Phillip Wood corrected his earlier assumption that the lock is shared across worktrees.
- **history reword memory leak fix proposed**: Patrick Steinhardt proposed a fix for the `history reword` memory leak in builtin/history.c; Kaartic Sivaraam explained the earlier similar patch was held back because Peff suggested its commit message was misleading.

- Patrick Steinhardt criticized Mark C. Chu-Carroll's shell-test conversion series: commit message conventions, series too large, and several `test_path_is_file` conversions wrong where the file may legitimately not exist.
- Beat Bolli posted a three-patch imap-send series: OpenSSL 4.1 preparation, NUL-terminated ASN1_STRING copies, and RFC 6125-compliant CN checking only when no SAN DNS names are present.
- Patrick Steinhardt framed the rerere lock issue as two distinct problems -- the long-standing lock race and the overly trigger-friendly gc heuristic his v2 series targets.
- **CI: Alpine for linux32, oldstable for Debian**: Patrick Steinhardt suggested using Alpine Linux for the 32-bit job and the "oldstable" tag for Debian to cut churn.

- Ted Nyman pinged about his fetch-pack Trace2 patch for packfile URI downloads, which Patrick Steinhardt had earlier said looked good.
- Brigham Campbell posted a one-line doc fix separating a conjoined bullet in Documentation/config/maintenance.adoc; Patrick Steinhardt confirmed the misrendering in git-config(1).
