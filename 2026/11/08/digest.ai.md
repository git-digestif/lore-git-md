# Git mailing list daily digest for 2026/11/08

## Long threads

### Detached HEAD documentation rewrite

*Long thread (root 2026/10/07/14-52-01): 22 replies so far from 4 participants over 33 days; 1 message that day.*

Julia Evans's series rewrites the detached HEAD explanation into a standalone `git help detached-head` guide, wired into `advice.c`, `command-list.txt`, and the build files, addressing complaints that the current text does not explain why detached HEAD is useful, buries the definition, and does not advertise that `git switch` makes accidental detachment harder. Earlier, v4 incorporated Junio C Hamano's wording suggestions, and D. Ben Knoble identified two remaining blockers: applying a squash patch and resolving how to describe HEAD. Today Junio C Hamano expanded on the "parent of the next commit" mental model as HEAD being "where you started from", appreciated Knoble's point about not clobbering an existing branch or creating a temporary one, but objected to phrasing that calls it useful for HEAD to be a commit ID -- the notable fact is that HEAD directly refers to a commit, with the hexadecimal name being an implementation detail -- and suggested starting the paragraph with "directly referring to a commit". He said he hopes the document is approaching a finish.

### test-tool genbytes to avoid slow tr(1) on macOS

*Long thread (root 2026/11/07/14-59-22): 15 replies so far from 3 participants over 2 days; 13 messages that day.*

Kristofer Karlsson's series adds `test-tool genbytes` to replace slow `tr(1)` calls in tests such as t4205, where macOS CI spent most of its time in `tr` on over 2GB of generated data. Today Junio C Hamano retracted his earlier warning that renaming `test-genzeros.c` would break tests, since the dispatcher still recognizes `genzeros`, though he still prefers a single rename-and-modify commit for history readers. Johannes Sixt proposed `printf` padding alternatives, but Karlsson reported that `printf` fails on macOS for large padding, outputting only literal zeros, and Sixt accepted that. Karlsson then posted v2: patch 1 adds `genbytes <hex-byte> [<count>]` taking two lowercase hex digits so NUL is `00`, renames the helper, and replaces `genzeros | tr` call sites; patch 2 converts the remaining `genzeros` callers to `genbytes 00` and removes `genzeros`.

### Moving assume_unchanged into repository configuration

*Long thread (root 2026/11/08/07-19-33): 6 replies so far from 2 participants over 1 day; 9 messages that day.*

Hussain Sitamouwala posted a patch moving `assume_unchanged` from global state into repository-specific configuration via a new `repo_assume_unchanged()` getter, updating callers in `builtin/update-index.c` and `read-cache.c` so the setting is not shared across repositories. Earlier today he also iterated on a separate documentation patch describing promisor remote fetch order: Junio C Hamano objected that v2 was not threaded to v1, called the text well written but questioned one wording choice, and asked for a redundant fetch-fallback sentence to be removed; Sitamouwala agreed to all of it and posted v3 with the sentence and its commit-message clause dropped.

## The day in brief
The day's most consequential work spans documentation and core fixes: Junio C Hamano refined the last wording of Julia Evans's detached HEAD guide while saying he hopes it is approaching a finish, Tanga Joris fixed split-index entry loss under `index.skipHash`, and 井上司 diagnosed a reftable HEAD reflog bug while proposing an all-or-none approach for switch failures.

## Notable threads

### Split index entries lost with index.skipHash
Tanga Joris submitted an Outreachy patch fixing a bug where `core.splitIndex` and `index.skipHash` together lose index entries: `write_shared_index()` names the file after the trailing checksum, which is zeroed under `index.skipHash`, so the split index records a null base and treats the shared index as absent, and `git status` can die with a BUG(). The patch passes `skip_hash = 0` when writing the shared index while the regular index still honors the setting, with two new tests in `t/t1700-split-index.sh`.

### reftable HEAD reflog records stale value
井上司 reported that with the reftable backend, a transaction that updates a branch and points HEAD at it records the branch's old value in the HEAD reflog, or writes no entry at all if the branch is created in the same transaction, because `write_transaction_table()` in `refs/reftable-backend.c` resolves the target with pre-transaction refs. Karthik Nayak agreed the diagnosis is plausible and suggested moving reflog logic into the generic `refs.c` layer so backends receive pre-resolved entries, which would prevent the same bug in any future atomic backend, while saying the specific fix would also resolve the issue and be welcome.

### Defaulting the hash algorithm to GIT_HASH_DEFAULT
brian m. carlsson posted a patch making `setup.c` default the hash algorithm to `GIT_HASH_DEFAULT` when no other is chosen, so breaking-changes mode and Git 3.0 select SHA-256, with a new test in `t/t0001-init.sh`. Junio C Hamano asked whether `HASH_UNKNOWN` should instead mark a not-yet-initialized repository whose first push could decide its hash, and later raised that the patch would clobber the object format when `git init` reinitializes an existing repository, outlining possible reinit behaviors and asking what `git init` does about the reference storage format.

### All-or-none switch/checkout on tracking-config failure
井上司 reported testing two approaches to the halfway switch failure when writing tracking config fails: holding one reference transaction across the worktree update (approach A), which breaks other ref updates in reftable, causes HEAD.lock conflicts in files, and breaks `-m` autostash; and preparing the transaction only as a check before the worktree update (approach B), which leaves nothing locked during the update but does not cover a race with another process creating the branch. He proposes proceeding with approach B, writing the tracking config before the worktree update and removing it if a later step fails.

### credential-cache daemon startup races
Farid Zakaria's v2 series fixes two races when two `git credential-cache store` clients start daemons at once, where the second daemon unlinks the first's socket and its stored credentials are lost. Today Junio C Hamano questioned the cover letter's "upgrades (old git version)" reference, asking how someone unfamiliar with the codebase could know which existing arrangements could be ignored; Zakaria replied that he was unsure how to respond and had tried to capture his scope in the cover letter.

## In brief
- Jeff King reported that `[[name_only]]` anchors before code blocks work with Asciidoctor but not python Asciidoc, which tempts him toward dropping python support, and asked whether an `[[ignore_all_space]]` anchor was misplaced next to `--color-moved-ws`.
- Tanga Joris posted Outreachy proposal draft v2 rebuilding the plan around converting hand-written parsing to parse-options, documenting bugs in `git stash show -G`, `index-pack --max-input-size`, `fetch-pack --depth`, `git diff --no-index`, and `git daemon --port`.
- Cynthia Uche posted proposal draft v3 with `git bisect` as the main target, a "How I work" methodology section, and an updated week-by-week timeline.
- Laveesh Bansal posted a patch collecting `--all` and named refs into one list for a single `refs_reflog_expire()` loop in `builtin/reflog`, keeping the name lookup inside the loop so output and exit status stay the same.
- Tanga Joris reviewed the credential `OPT_SUBCOMMAND` patch as good overall, suggesting a helper for the repeated parse calls and a test that each subcommand rejects extra arguments with exit code 129.
- Torsten Bögershausen reported that explicitly listing test files and using `rm -fv` plus `git checkout` cut t0027 from 71 to 54 seconds on older macOS.
- Colin Hinton said he will adopt Matt Hunter's cleaner `get_follow_remote_head()` approach in his next version and is holding the re-roll until `hn/fetch-refmap` is accepted, asking Junio C Hamano for a better base branch.
