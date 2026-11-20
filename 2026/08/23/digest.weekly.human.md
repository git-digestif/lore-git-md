## Long threads

### `git history squash` subcommand series

*Long thread: 86 replies so far from 9 participants over 68 days.*

Harald Nordgren posted v14 of the eight-patch series adding `git history squash <revision-range>` to fold commit ranges into one, updating trailer handling and simplifying test usage. The week saw Phillip Wood note missing `Signed-off-by` lines from his fixups and Kristoffer Haugsbakk advise on trailer ordering, while Junio C Hamano corrected the use of `istarts_with()` for matching hex object IDs in `fixup!` subjects, insisting the match must be case-sensitive. The series remains in active review with these details being refined.

### Moving libgit.a sources into a `lib/` directory

*Long thread: 38 replies so far from 12 participants over 124 days.*

Michael Montalbo's RFC to move roughly 700 libgit.a source files into a `lib/` directory drew Junio C Hamano's response to his mechanical grouping experiment. Junio observed that the earliest group forms a coherent "transport" cluster matching intuition, but suggested carving conceptually clean groups into their own directories -- as was done with `builtin/`, `refs/`, or `odb/` -- without waiting to classify the remainder, and raised whether bisect and revision machinery could be separated.

### Pluggable packfile writes for `receive-pack`

*Long thread: 35 replies so far from 3 participants over 16 days.*

Justin Tobler advanced the series to v5, introducing the generic `odb_transaction_write_pack()` interface so that `git-receive-pack(1)` writes incoming packfiles through the ODB transaction layer rather than child processes tied to the "files" backend. Patrick Steinhardt approved using transaction flags such as `ODB_TRANSACTION_RECEIVE` to differentiate callers, while Junio C Hamano flagged reading `receive.unpackLimit` in generic object-layer code as a layering violation and suggested `odb-file.unpackLimit` as a more accurate name.

### Pluggable packfile generation via the object database

*Long thread: 31 replies so far from 6 participants over 15 days.*

Patrick Steinhardt posted v4 of the series making packfile generation pluggable for `git-upload-pack`, `git-send-pack`, and `git-bundle` via a new `odb_generate_pack()` interface with a "files" backend. The version fixed a use-after-scope bug Elijah Newren identified in `odb_source_files_generate_pack()`, and Karthik Nayak gave a clean review, though Elijah proposed an alternative for the `the_repository` cleanup in `bundle.c` which Patrick defended.

### Eagerly loading ODB alternates

*Long thread: 31 replies so far from 5 participants over 12 days.*

Patrick Steinhardt's v3 initializes alternates up front when the object database is created, adding a first patch that creates the ref and object databases after the repository config is written in `init_db()`. Karthik Nayak affirmed the rationale that `core.sharedRepository` must be known when the ODB is created, while Junio C Hamano questioned `eq()` semantics for filesystems with differing case sensitivity, leading to agreement that the existing hashing can stay as a best-effort mechanism since inconsistent casing only causes a redundant duplicate search.

### Ambiguous remote branch message in `worktree add`

*Long thread: 27 replies so far from 6 participants over 15 days.*

Yoichi NAKAYAMA posted v7 of the three-patch series improving the error when `git worktree add` or `git checkout` matches multiple remote tracking branches with the same name, rewording the advice to "Branch name '%s' appears in multiple remotes:" followed by the matched remotes. Junio C Hamano reviewed v7, called the changes sensible, and queued the series, noting that the multiple-match-but-success case in `git worktree add` remains a separate bug to address later.

### New path keys for `git repo info`

*Long thread: 25 replies so far from 4 participants over 37 days.*

K Jayatheerth's GSoC series adds `path.*` keys to `git repo info` for toplevel, superproject root, hooks, index, grafts, and git-prefix, each with absolute and relative suffixes. The week saw agreement that `path.git-prefix` is repository information complementing `cdup`, with Jayatheerth committing to add a `cdup` key in the next version after Junio C Hamano's suggestion that both should be offered together.

### diff hunk provider interface rethought

*Long thread: 25 replies so far from 4 participants over 94 days.*

Michael Montalbo retracted the "diff provider" abstraction introduced in RFC v7, saying it shoehorns several concepts together, and decided to return to the v6 line for future re-rolls while ejecting the diff-hunks store patches. He clarified goals for a narrower pkt-line interface to xdiff for user-configured alternatives, OID-only operation for callers needing only hunk header info, and consistent composition with surrounding diff functionality.

### Converting `USE_NSEC` to runtime config

*Long thread: 23 replies so far from 5 participants over 14 days.*

D. Ben Knoble posted v4 converting build-time `USE_NSEC` into the runtime `core.useNanosec` config, using the `use_nanosec` member unconditionally while ignoring the config variable only under `NO_NSEC`. Patrick Steinhardt welcomed removing the preprocessor directives in `is_racy_stat()` and agreed no warning should be emitted on `NO_NSEC` builds, since users sharing `$HOME/.gitconfig` across machines would be annoyed; Junio C Hamano called the struct alignment a "great improvement".

### Packfile loading performance with many packs

*Long thread: 22 replies so far from 5 participants over 6 days.*

Jeff King identified that `packfile_list_prepend()` has essentially the same O(N^2) duplicate-check problem as `append()`, with synthetic tests showing `git rev-list --count foo` growing from 18 ms with 500 packs to 6351 ms with 16000 packs. He concluded that MRU updates should use constant-time movement within a doubly-linked list rather than an explicit duplicate check and removal, and Patrick Steinhardt noted the prepend calls appear in hot code paths including `find_pack_entry()`.

## The period in brief

The week of 2026/08/17 through 2026/08/23 saw steady traffic across seven active days, dominated by Patrick Steinhardt's ODB abstraction efforts and Harald Nordgren's long-running `git history squash` series. Readers should not miss the pluggable packfile interfaces reaching v4 and v5, Elijah Newren's fix for the multi-pack-index race, and the project's refusal to review an AI-generated `utime.h` migration over Developer's Certificate of Origin concerns.

## Key developments

### ODB abstraction: pluggable packfile writes and generation

Patrick Steinhardt and Justin Tobler advanced the object database abstraction with two related series: v5 of pluggable packfile writes for `git-receive-pack(1)` via `odb_transaction_write_pack()`, and v4 of pluggable packfile generation for transport via `odb_generate_pack()`. Both make the "files" backend an implementation detail rather than a hard dependency, with review focusing on transaction flags, `.keep` file lifecycle tests, and whether config names like `receive.unpackLimit` belong in generic object-layer code.

### Generic corrupt-object handling

Patrick Steinhardt posted v2 of the series moving `OBJECT_INFO_DIE_IF_CORRUPT` handling into `do_oid_object_info_extended()` generically using `enum odb_read_status` with negative error codes and an `errmsg` parameter so sources can bubble up human-readable corruption messages. Junio C Hamano accepted the tri-state approach but noted the reasoning relies on the invariant that a single pack contains each object at most once.

### Multi-pack-index race recovery

Elijah Newren's two-patch series addresses a race where geometric repacking creates a new multi-pack-index and removes the pack an older index named as owner, making present objects look missing to running processes. The series makes `git replay` fail gracefully with an error instead of SIGSEGV when a merge input is unreadable, and teaches `find_pack_entry()` to scan the multi-pack-index's covered packs directly for a surviving copy. Junio C Hamano confirmed the diagnosis and will prepare an evil-merge to adapt to API changes from other in-flight topics.

### Reftable stack reload optimization

Karthik Nayak's series removes `REFTABLE_STACK_NEW_ADDITION_RELOAD` and avoids reloading the reftable stack when it is already locked within a reference transaction, cutting the `fstat()` calls that dominated ref creation. Jeff King reported a ~25% speedup for `update-ref --stdin` with 50,000 refs, and Patrick Steinhardt noted libgit2 also uses the flag, suggesting a rename of `reftable_stack_new_addition()`.

### Parallel packfile URI fetches

Patrick Steinhardt submitted a two-patch series letting `git-fetch(1)` and `git-clone(1)` fetch packfile URIs in parallel, refactoring `fetch-pack.c` into preparation, fetch, and aggregation phases and adding the `fetch.packfileURIThreads` config (default 1, 0 for auto-detect). Benchmarks showed speedups up to 7.58x on a throttled connection.

### AI-generated patch refusal

brian m. carlsson declined to review a treewide migration from `utime.h` to `utimensat()` because its cover letter disclosed generation by Gemini Flash, arguing this cannot satisfy the Developer's Certificate of Origin requirement. Junio C Hamano endorsed the stance as prudent, explaining that the project bears the cost if DCO-covered work later proves legally unusable, and brian m. carlsson affirmed that knowingly distributing suspect code creates legal exposure.

### `git organize` RFC

Michael Montalbo posted a 14-patch RFC introducing `git organize`, a command that records a project's file-placement guideline in a tracked `.gitorganize` file and moves out-of-place files, repointing references and staging the result. It has `status` and `apply` subcommands, configurable `organize.labeler`/`organize.organizer` commands, and eight planned carves for subsystems, with the final patch moving transport sources into `transport/`.

### Shallow clone push optimization

Elijah Newren submitted a `send-pack` patch that adds reachable shallow grafts as negative pack tips so pushing from a shallow clone no longer sends the entire toplevel tree, controlled by a new `push.shallowExcludeBoundary` config (default true). Patrick Steinhardt reviewed it with questions about graft amendment, tree negotiation, and `push.negotiate` defaults, which Elijah answered while agreeing to mark parameters `const`.

## In brief

**Use-after-free fix** -- Shlok Kulshreshtha's v3 fix collapsing two exits in `get_oid_with_context_1()` so `free(new_path)` happens after the last use of `cp` was reviewed as "nicely described" and queued by Junio C Hamano.

**`worktree add` DWIM fix** -- Yoichi NAKAYAMA's v4 preventing DWIM to remote-tracking branches when `-b` or `-B` is given was reviewed and queued by Junio C Hamano.

**Worktree repair relative paths** -- Yoichi NAKAYAMA posted v3 of the fix adopting a caller-prepared `strbuf` signature for `read_gitfile_raw()` so `git worktree repair` can convert relative `.git` cross-references to absolute paths.

**`http.sslVerifyStatus`** -- Grayson Gordon posted v6 of the stapled OCSP response check with documentation rephrased per Junio C Hamano's suggestion.

**`git pull --hard` vs `--reset`** -- Discussion of Artur Bieniek's `git pull --hard` proposal shifted toward Phillip Wood's `git pull --reset [--discard-changes | --autostash]` alternative, which Artur now prefers since it refuses to discard uncommitted changes by default.

**Report hook for receive-pack** -- Karthik Nayak posted v2 of the `report` hook; Junio C Hamano agreed a failed report hook should explicitly tell the client the push was declined, modeled on pre-receive.

**Trailer URL fix** -- Kristoffer Haugsbakk's v3 making trailer parsing fail on `://` separators so HTTPS URLs are not mistaken for trailers was marked for `next`.

**Credential libsecret fix** -- Daniel Martí defended his explicit secret-loading fix in `credential/libsecret.c` against a NULL-check fallback, arguing the flag discards errors from the second round trip and skips locked items.

**`git config --global` XDG** -- Delilah Ashley Wu posted v2 making `git config --global` read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config`, fixing the inconsistency with plain `git config list`.

**Stash ref configuration** -- Vladimir Sitnikov proposed a `stash.ref` knob (defaulting to `refs/stash`) so a worktree can use an isolated stash stack such as `refs/worktree/stash`.

**Subtree rebase bug** -- Gabriel Ford reported that after `git subtree add --squash`, a later `git rebase` conflicts between main repo and subtree files with identical relative paths.

**gitk README** -- Johannes Sixt submitted a patch discouraging AI contributions, prompting discussion with Weijie Yuan about how "Generated" differs from "Assisted" content.

**gitdatamodel advertising** -- Kristoffer Haugsbakk posted v2 of the series advertising gitdatamodel(7) across documentation.

**Unicode box-drawing opt-out** -- Oswald Buddenhagen proposed an opt-out option for the Unicode output in `git repo structure`; Junio C Hamano agreed an opt-out is needed since locale-only detection leaves no way to disable it.

## Looking ahead

The `git history squash` series awaits resolution of the `istarts_with()` case-sensitivity question and final sign-off ordering before it can advance. The pluggable packfile writes series needs a test verifying the full `.keep` file lifecycle from quarantine to main ODB. K Jayatheerth will add a `cdup` key to the `git repo info` path series in its next iteration. The `git organize` RFC and Vladimir Sitnikov's `stash.ref` proposal will see initial review discussion in the coming days.