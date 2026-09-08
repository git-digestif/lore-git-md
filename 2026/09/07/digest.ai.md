# Git mailing list daily digest for 2026/09/07

## The day in brief
The Git mailing list saw active discussion on several fronts today. Domen Kožar and Kristoffer Haugsbakk continued their debate over the `post-worktree` hook design, with Domen clarifying that hooks and IDs serve complementary purposes. Ted Nyman pinged Junio C Hamano about Trace2 instrumentation for `git fetch-pack`, which is ready for `next`. Phillip Wood expanded the discussion on the lowercase-only object ID series by proposing to forbid ref names that look like object IDs. Patrick Steinhardt posted v3 of his ODB alternates refactoring series, which consolidates alternates handling into ODB creation. Thomas Bachem and reviewers refined the auto-maintenance deferral series, agreeing to move maintenance calls to built-in commands. A critical bug was reported where `git worktree add "" HEAD` deletes the `.git` directory on Windows. Beat Bolli posted a series fixing OpenSSL compatibility and correctness issues in `git imap-send`.

## Notable threads

### Unified `post-worktree` hook design discussion
**What’s changing?**
The thread explores two competing approaches to worktree lifecycle notifications: Domen Kožar’s unified `post-worktree` hook (subcommand-style: `add`, `remove`, `move`) and Kristoffer Haugsbakk’s `--id=<string>` option for `git worktree add`.

**Why it matters**
As coding agents automate worktree operations, reliable lifecycle notifications become essential. The discussion now centers on whether hooks (for observability) and IDs (for ownership) can coexist.

**Today’s updates**
- [2026/09/07/14-30-13] Domen Kožar pinged Junio C Hamano to confirm whether the unified `post-worktree` hook proposal meets the bar for a Git-native hook, emphasizing the growing need for reliable lifecycle notifications.
- [2026/09/07/18-18-47] Kristoffer Haugsbakk proposed replacing the hook with a `--id=<string>` option, arguing that ownership (not observability) is the core problem.
- [2026/09/07/19-34-43] Domen Kožar clarified that hooks and IDs serve complementary purposes—hooks for observability (tools reacting to uncontrolled events) and IDs for ownership (tools preventing interference)—and suggested they could coexist.

**Key technical details**
- Domen’s proposal: unified `post-worktree` hook with subcommand-style interface (`add`, `remove`, `move`). Hook signature: `post-worktree <subcommand> <worktree-id> <worktree-path> [<old-path>]`.
- Kristoffer’s counterproposal: `--id=<string>` option for `git worktree add`, storing the ID in the worktree and enforcing ownership.

**Status**
Under active design discussion; no merge status change. Domen’s version 2 compromise remains in `seen` pending resolution of Junio’s fundamental objection.

---

### Trace2 instrumentation for `git fetch-pack`
**What’s changing?**
Ted Nyman’s patch adds Trace2 telemetry to the packfile URI download process in `git fetch-pack` to expose the number of URIs advertised and the download loop duration in protocol v2 fetches.

**Why it matters**
This instrumentation helps monitor and optimize the performance of protocol v2 fetches, particularly in environments with multiple packfile URIs.

**Today’s updates**
- [2026/09/07/21-24-54] Ted Nyman pinged Junio C Hamano to check for remaining concerns, noting Patrick Steinhardt’s positive review and the patch’s readiness for `next`.

**Key technical details**
- Files: `fetch-pack.c`, `t/t5702-protocol-v2.sh`
- New symbols: none (uses existing Trace2 APIs: `trace2_region_enter`, `trace2_region_leave`, `trace2_data_intmax`)
- Behavior: emits region events around the URI download loop and a data event with the URI count; no per-pack spam

**Status**
Ready for `next` unless Junio raises new objections.

---

### Lowercase-only object ID series
**What’s changing?**
brian m. carlson’s RFC series proposes a breaking change for Git 3.0: restricting hex object IDs to lowercase only to eliminate compatibility and security risks for external tools that assume hex uniqueness.

**Why it matters**
Git currently emits lowercase hex object IDs but accepts both uppercase and lowercase in parsing, creating potential security issues for external tools.

**Today’s updates**
- [2026/09/07/13-37-04] Phillip Wood acknowledged brian’s security rationale but expanded the discussion to adjacent policy questions: if uppercase hex can defeat a security check, why can’t a ref name pointing to the same commit do the same? He proposed forbidding ref names that *look like* object IDs (e.g., `refs/heads/abcdef1234...`), suggesting the thread may broaden to broader input validation.

**Key technical details**
- New files: `hex-ll.c`/`hex-ll.h` (lower-level hex parsing utilities).
- New symbols: `hex_kind` enum (`HEX_KIND_MIXED`, `HEX_KIND_LOWER`, `HEX_KIND_OID`), `hexval_lc_table`, updated `hexval()`, `hex2chr()`, and `hex_to_bytes()` functions.
- Modified subsystems: hex parsing in packet-length parsing, quoted-printable decoding, percent-encoding, ref formatting, URL handling, object ID parsing from disk/network/diagnostics, and object name resolution.

**Status**
v2 of the RFC series is complete and addresses all feedback from the initial RFC. The series remains in RFC limbo, with no final decision on whether the project will accept the breaking change.

---

### ODB alternates refactoring
**What’s changing?**
Patrick Steinhardt’s 9-patch series refactors how Git writes object database (ODB) alternates during repository creation, removing the ability to write alternates after repository creation and consolidating alternates handling into ODB creation.

**Why it matters**
This simplifies the ODB interface and prepares for alternates to become an implementation detail of the ODB backend, giving backends more control over their configuration.

**Today’s updates**
- [2026/09/07/07-23-21] Patrick Steinhardt responded to Justin Tobler’s review, clarifying that the awkwardness of `init_db()`’s flag-based interface could be resolved by lifting repository initialization messages out of the function entirely and renaming it to `create_repository()`.
- [2026/09/07/08-25-37] Patrick Steinhardt posted v3 of the series, introducing `create_repository()` to split repository initialization into skeleton creation, ODB setup, and refdb setup, eliminating skip flags and consolidating alternates logic.
- Patches 1-9/9 in v3 were posted, covering the refactoring of `init_db()`, deferring ODB initialization in `git clone`, and consolidating alternates handling into ODB creation.

**Key technical details**
- Files touched: `builtin/clone.c`, `setup.c`/`setup.h`, `odb/source-files.c`, and other ODB source files.
- New or renamed symbols: `create_repository()` (replaces `init_db()` for skeleton creation), `collect_alternates()` (renamed from `setup_reference()`), `odb_create_on_disk_options` (carries alternates for ODB creation).
- Subsystems: ODB, repository creation, alternates handling, and worktree support.

**Status**
Under review. The series is built on `2c3adbb2c4` (The 18th batch, 2026-08-24) with `ps/odb-eagerly-load-alternates` merged in.

---

### Auto-maintenance deferral during sequencer operations
**What’s changing?**
Thomas Bachem’s series defers auto maintenance until the end of sequencer-driven operations (`git rebase`, `git cherry-pick`, `git revert`) to prevent maintenance from interfering with intermediate steps and ensure consistent behavior across all three commands.

**Why it matters**
This prevents lock contention and ensures that maintenance runs exactly once at sequence completion, matching the apply backend’s behavior.

**Today’s updates**
- [2026/09/07/08-14-07] Patrick Steinhardt suggested improving the commit message for `git_config_append_parameter()` to explain `GIT_CONFIG_PARAMETERS` and its quoting format.
- [2026/09/07/08-14-12] Patrick Steinhardt raised concerns about the commit message clarity and the scattering of `run_auto_maintenance()` calls across multiple exit paths.
- [2026/09/07/13-24-13] Phillip Wood suggested rewording the commit message to avoid implying active interference and to clarify edge cases.
- [2026/09/07/16-35-20] Thomas Bachem proposed moving the `run_auto_maintenance()` call from the sequencer to the built-in commands (`run_specific_rebase()` and `run_sequencer()`), consolidating maintenance into a single exit path per command.
- [2026/09/07/16-40-12] Phillip Wood approved the proposal to move maintenance calls to built-in commands, aligning with the apply backend’s design.

**Key technical details**
- Files touched: `sequencer.c`, `config.c`, `config.h`, `builtin/rebase.c`, `builtin/revert.c`, test files `t/t3418-rebase-continue.sh` and `t/t3510-cherry-pick-sequence.sh`
- New behavior: auto maintenance runs exactly once at sequence completion (after autostash is applied) instead of during intermediate commits/merges
- CLI/config impact: disables auto maintenance for individual commands (`git commit`, `git merge`, exec commands) during any sequencer-driven operation by passing `maintenance.auto=false` via `GIT_CONFIG_PARAMETERS`

**Status**
Under review. The series is technically complete, with all prior feedback addressed. The latest refinement resolves the final design question by moving the `run_auto_maintenance()` call out of the sequencer and into the built-in commands.

---

### Critical bug: `git worktree add "" HEAD` deletes `.git` on Windows
**What’s changing?**
Paul DE TEMMERMAN reported a critical bug on Windows where `git worktree add "" HEAD` deletes the `.git` directory, describing a high-severity, platform-specific data-loss scenario.

**Why it matters**
This is a critical data-loss bug that affects Windows users and undermines the reliability of the `git worktree` command.

**Today’s updates**
- [2026/09/07/22-12-50] Paul DE TEMMERMAN reported the bug, providing clear reproduction steps and system information.

**Key technical details**
- Subsystem: `git worktree` (specifically the `add` subcommand).
- Files likely involved: `builtin/worktree.c`, Windows-specific path handling in `compat/`.
- Platform: Windows (64-bit, OpenSSL 3.5.7, SHA-1/SHA-256).
- Trigger: empty string argument (`""`) passed as the worktree path.
- Impact: deletion of the `.git` directory (not just the worktree).

**Status**
Newly reported; no prior discussion or resolution.

---

### OpenSSL compatibility and correctness fixes for `git imap-send`
**What’s changing?**
Beat Bolli’s three-patch series fixes OpenSSL compatibility and correctness issues in `git imap-send`, including preparing for OpenSSL 4.1, fixing unsafe ASN1_STRING handling, and aligning certificate validation with RFC 6125.

**Why it matters**
These fixes ensure that `git imap-send` remains compatible with future OpenSSL releases and adheres to modern security best practices.

**Today’s updates**
- [2026/09/07/21-12-07] Beat Bolli posted the series, covering OpenSSL 4.1 preparation, unsafe ASN1_STRING handling, and RFC 6125 alignment.
- Patches 1-3/3 were posted, addressing the deprecated ASN1_STRING function, unsafe ASN1_STRING handling, and RFC 6125 alignment.

**Key technical details**
- Files touched: `imap-send.c` only.
- Subsystem: `imap-send` (IMAP TLS certificate verification).
- New/renamed symbols: none (the macro `ASN1_STRING_get_length` is defined only for pre-4.1 OpenSSL and maps to `ASN1_STRING_length`).
- Behavior changes: stricter RFC 6125 compliance, removal of deprecated OpenSSL API usage, forward compatibility with OpenSSL 4.1, and elimination of undefined behavior from unsafe ASN1_STRING handling.

**Status**
Under review; no prior versions.

## In brief
- **[2026/09/07/06-06-47]** Patrick Steinhardt affirmed the `receive-report` hook series’ readiness for integration, noting all prior feedback has been addressed.
- **[2026/09/07/07-49-56]** Patrick Steinhardt confirmed Karthik Nayak’s diagnosis of a bug in the `cache-tree` subsystem, which was incorrectly using `the_repository` for `.gitmodules` lookups.
- **[2026/09/07/07-50-01]** Patrick Steinhardt acknowledged Justin Tobler’s feedback on a commit message’s clarity and offered to rewrite it if needed.
- **[2026/09/07/11-13-23]** Patrick Steinhardt posted a follow-up in the ref-storage terminology unification series, confirming the clear division of labor between his patch series and Thomas Bachem’s concurrent effort.
- **[2026/09/07/11-18-35]** Patrick Steinhardt posted patch 1/11 in v2 of the ref-storage terminology unification series, renaming `--ref-format=` to `--ref-storage-format=` in `git init`.
- **[2026/09/07/11-18-36]** Patrick Steinhardt posted patch 2/11 in v2, renaming `--ref-format=` to `--ref-storage-format=` in `git clone`.
- **[2026/09/07/11-18-37]** Patrick Steinhardt posted patch 3/11 in v2, renaming `--ref-format=` to `--ref-storage-format=` in `git refs migrate`.
- **[2026/09/07/11-18-38]** Patrick Steinhardt posted patch 4/11 in v2, renaming `--ref-format=` to `--ref-storage-format=` in `git submodule`.
- **[2026/09/07/11-18-39]** Patrick Steinhardt posted patch 5/11 in v2, renaming `--show-ref-format` to `--show-ref-storage-format` in `git rev-parse`.
- **[2026/09/07/11-18-40]** Patrick Steinhardt posted patch 6/11 in v2, renaming the build-info output field `default-ref-format` to `default-ref-storage-format`.
- **[2026/09/07/11-18-41]** Patrick Steinhardt posted patch 7/11 in v2, extracting URI-parsing logic into a new reusable function `ref_storage_format_by_uri()`.
- **[2026/09/07/11-18-42]** Patrick Steinhardt posted patch 8/11 in v2, refactoring the logic that determines and validates the ref storage format during repository (re)initialization.
- **[2026/09/07/11-18-43]** Patrick Steinhardt posted patch 9/11 in v2, renaming environment variables while retaining backward-compatible aliases.
- **[2026/09/07/11-18-44]** Patrick Steinhardt posted patch 10/11 in v2, renaming the config key `init.defaultRefFormat` to `init.defaultRefStorageFormat`.
- **[2026/09/07/11-18-45]** Patrick Steinhardt posted patch 11/11 in v2, enabling URI payloads for `--ref-storage-format=`.
- **[2026/09/07/12-56-09]** Vsevolod Myalitsin posted a patch to update the on-screen instructions for disabling the `defaultBranchName` advice message, recommending `--global` instead of `set`.
- **[2026/09/07/18-48-58]** Tuomas Ahola posted a patch to replace the `{,8}` Perl regex quantifier shorthand with `{0,8}` in `Documentation/lint-gitlink.perl` to maintain compatibility with Git’s minimum Perl version (5.26.0).
- **[2026/09/07/04-44-23]** Eli Barzilay reported a bug in `git merge --ff-only --autostash` where `stash.index=true` combined with a staged change leaves a redundant stash entry and fails to clean up the `MERGE_AUTOSTASH` ref.
- **[2026/09/07/07-15-57]** Brigham Campbell posted a one-line documentation fix to separate conjoined bullet items in `Documentation/config/maintenance.adoc`.
- **[2026/09/07/08-19-17]** AIKSXD ax reported a data-loss bug where `.gitignore` trailing-slash patterns fail to ignore symlinks, leading to silent overwrites during `git pull`.