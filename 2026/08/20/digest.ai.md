# Git mailing list daily digest for 2026/08/20

## Long threads

### `git history squash` subcommand for folding commit ranges

*Long thread (root 2026/06/14/19-25-39): 86 replies so far from 9 participants over 68 days; 2 messages that day.*

Harald Nordgren posted v14 of the eight-patch series adding `git history squash <revision-range>` to `builtin/history.c`, which folds a range of commits into one while preserving the oldest commit's authorship and parents and taking the tip's tree. The v14 changelog updates commit message trailers and simplifies `rev-parse` and `tr` usage in the tests. Earlier, v13 had split the implementation into five patches covering parsing, range validation, branch protection, commit creation, and message editing, after review-process friction over rapid rerolls. Today Junio C Hamano questioned why patch 7/8 uses `istarts_with()` to match hex object IDs in `fixup!` subjects, noting that `fixup! ABCdef` should not be accepted.

### Pluggable packfile writes for `git-receive-pack`

*Long thread (root 2026/08/06/21-38-53): 33 replies so far from 3 participants over 15 days; 5 messages that day.*

Justin Tobler posted v5 of the nine-patch series "builtin/receive-pack: support pluggable packfile writes", which introduces the generic `odb_transaction_write_pack()` transaction interface so the incoming packfile can be written by any ODB backend, with the existing child-process logic in `git-receive-pack(1)` becoming the "files" backend implementation. The v5 changes add a test assertion that `.keep` files are migrated to the main ODB before removal at finalize and update a commit message. Earlier, Patrick Steinhardt had asked for a `reference-transaction` hook assertion covering the full `.keep` lifecycle and approved passing the unpack limit via `odb_transaction_flags` rather than assuming `git-receive-pack(1)`; Tobler agreed to add the lifecycle test in the next version.

### Eagerly loading ODB alternates

*Long thread (root 2026/08/10/13-33-27): 29 replies so far from 5 participants over 11 days; 3 messages that day.*

Patrick Steinhardt's v3 series "odb: eagerly load alternates" moves alternate initialization into object database setup, since deferring it no longer buys much now that alternates are parsed on nearly every object lookup. The new first patch creates the ref and object databases after the repository config is written in `init_db()`, because `core.sharedRepository` must be known when the odb is created. Today Karthik Nayak affirmed that rationale and said the new patch and range-diff look good, while Junio C Hamano questioned the `eq()` semantics in patch 2/5 for repositories whose object databases live on separate filesystems with differing case sensitivity, asking whether `core.ignoreCase` belongs in repository configuration at all and suggesting each object database source might report its own filesystem's case sensitivity.

### Pluggable packfile generation via the object database

*Long thread (root 2026/08/07/10-45-06): 24 replies so far from 6 participants over 14 days; 14 messages that day.*

Patrick Steinhardt posted v3 of "odb: make packfile generation pluggable", which adds `odb_generate_pack()` in `odb.h`/`odb.c` with a "files" backend implementation in `odb/source-files.c` that spawns `git-pack-objects`, and converts `git-upload-pack(1)`, `git-send-pack(1)`, and `git-bundle(1)` to generate packfiles through the interface. The v3 changelog states it fixes the use-after-scope bug in `odb_source_files_generate_pack()` that Elijah Newren reported, where a stale pointer to the function-local `cp` could be dereferenced on abnormal exit. Karthik Nayak reviewed the patches, asking about the unused `source` parameter and error wording, which Steinhardt answered; Junio C Hamano asked whether `opts->pack_fd` should be closed when the primary source does not support `generate_pack`.

### Converting `USE_NSEC` to runtime config

*Long thread (root 2026/08/07/11-56-23): 23 replies so far from 5 participants over 14 days; 5 messages that day.*

D. Ben Knoble posted v4 of "Convert USE_NSEC to runtime config" (topic `dk/use-nsec-runtime`), which exposes the build-time `USE_NSEC` knob as the runtime `core.useNanosec` configuration so index-related commands are less likely to hit racy-git problems on supported filesystems. In v4, `is_racy_stat()` and `match_stat_data()` select the nanosecond racy checks via the runtime value without preprocessor directives, and the docs mention potential platform unsafety. Junio C Hamano reviewed patch 2/3, which aligns `repo_config_values_init` with the struct declaration, suggesting a wording tweak and otherwise calling it a "great improvement".

## The day in brief

Harald Nordgren posted v14 of the `git history squash` series, Justin Tobler posted v5 of the pluggable packfile-writes series for `git-receive-pack`, and Patrick Steinhardt posted v3 of the pluggable packfile-generation series after fixing a use-after-scope bug. D. Ben Knoble posted v4 converting `USE_NSEC` to the runtime `core.useNanosec` config, and Junio C Hamano said he would queue Yoichi NAKAYAMA's `worktree add` DWIM fix.

## Notable threads

### `worktree add` should not DWIM with `-b`/`-B`
Yoichi NAKAYAMA posted v4 of the fix in `builtin/worktree.c` for a regression from Git 2.42 where `git worktree add -b <new-branch> <path> <branch>` could still DWIM `<branch>` to a remote-tracking branch, silently ignoring `<new-branch>`. The patch adds an `else if (ac == 2 && new_branch)` branch that skips DWIM, plus a test that `git worktree add -b branch ../wt foo` fails with "fatal: invalid reference: foo". Junio C Hamano reviewed it, noting it is much smaller than the initial iteration, and said "Will queue."

### Trailers: stop recognizing URLs as trailers
Kristoffer Haugsbakk posted v2 of the patch in `trailer.c`, rebased on `master` now that its base topic has graduated. `find_separator()` now fails when the separator and following characters form `://` before any whitespace, so HTTPS URLs at the end of commit messages are no longer parsed as trailers; v2 uses `starts_with` for readability and notes the heuristic can produce benign false positives like `1://`. Junio C Hamano had rebooted the topic after its base graduated and asked for a summary of outstanding issues.

### Reftable stack: avoid reloading when locked
Karthik Nayak's three-patch series in `reftable/stack.c` removes `REFTABLE_STACK_NEW_ADDITION_RELOAD` and avoids reloading the stack inside a reference transaction, where the stack is already locked and reloading is a no-op; previously most of the time creating many refs in one transaction was spent on `fstat()`. Jeff King reported a ~25% speedup for `update-ref --stdin` with 50,000 refs. Patrick Steinhardt noted libgit2 also uses the flag and suggested a rename, which Nayak agreed to add as another commit.

### `git pull --hard` mode
Artur Bieniek's patch adds `--hard` to `git pull` as an explicit alternative to merge and rebase that resets branch, index, and working tree to the fetched candidate. Phillip Wood proposed an alternative `git pull --reset [--discard-changes | --autostash]` that would refuse by default with uncommitted changes; Bieniek said he now prefers that distinction, since `--ff-only` does not cover upstream branches that were rebased or force-updated. Kristoffer Haugsbakk questioned the need for a local branch in that scenario, using the remote-tracking branch instead.

### Recovering when a multi-pack-index names a removed pack
Elijah Newren's two-patch series fixes a race where a running process holding an old multi-pack-index cannot access objects whose owning pack was removed by a geometric repack, plus a `git replay` SIGSEGV on missing objects. Patrick Steinhardt said the patch looks good overall, asked whether a variant affects callers not using `OBJECT_INFO_QUICK` (concluding they reload packfiles and find the object), and preferred the proposed lazy second search over eagerly loading every MIDX-referenced packfile.

## In brief

- Yoichi NAKAYAMA posted v2 of "worktree repair: detect relative path in .git file correctly", adding `read_gitfile_raw()` so `git worktree repair` can convert relative cross-references to absolute when `worktree.useRelativePaths` is false.
- Patrick Steinhardt's v2 series handles `OBJECT_INFO_DIE_IF_CORRUPT` generically across ODB sources; Karthik Nayak reviewed it and conceded the implicit return-code conversion is fine, though he proposed rewording a comment about error preference.
- Yoichi Nakayama posted v6 of "worktree add: improve message for ambiguous remote branch name", splitting the work into three commits and renaming the advice function to `advise_disambiguating_remotes`, applied to both checkout and worktree add. *Long thread (root 2026/08/08/08-21-41): 21 replies so far from 6 participants over 13 days; 3 messages that day.*
- Aaron Plattner posted v2 of "http: preserve wwwauth_headers across redirects", adding `credential_update_url()` in `credential.c` so the WWW-Authenticate list survives a redirect's URL update.
- Patrick Steinhardt agreed `git-history(1)` should refuse updating branches checked out in another worktree and created GitLab work item 772.
- Friel's v2 adds a `write_pack_file/wrote_bytes` Trace2 datum to `builtin/pack-objects.c`; Patrick Steinhardt and Jeff King discussed the hash algorithm used for the trailing checksum.
- Karthik Nayak's patch introduces a `report` hook for `git-receive-pack(1)`; Phillip Wood corrected the commit message's reasoning about config-based hooks.
- **Zsh completion v2 for `git -C <path>`**: Lutz Lengemann's v2 walks global options to set `__git_cmd_idx`; D. Ben Knoble approved conditionally, leaving only zsh-vs-bash index math uncertain.

- Daniel Martí nudged for review of the credential/libsecret patch loading secrets explicitly; Junio C Hamano proposed keeping `SECRET_SEARCH_LOAD_SECRETS` and loading only on NULL.
- Nikolaus Schuetz posted v2 of t1402 testing all documented forbidden refname characters in embedded form.
- Nikolaus Schuetz posted v2 of t1401 pinning `git symbolic-ref` exit codes (128 non-quiet, 1 with `--quiet`) and empty stdout.
- **Four-patch series links gitdatamodel(7) across docs**: Kristoffer Haugsbakk listed gitdatamodel(7) as a guide in git(1), linked four glossary terms to it, and added a link from the datamodel page to the glossary.
