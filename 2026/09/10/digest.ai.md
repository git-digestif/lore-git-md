# Git mailing list daily digest for 2026/09/10

## The day in brief
The Git project saw active development across multiple fronts today. Key highlights include a proposed simplification for the `advice: add scope hint` series, a critical memory leak fix in the `receive-report` hook, and a CI breakage in the Rust reorganization effort. The `git last-modified` Bloom filter optimization received substantive review feedback, and the `git history` memory leak fix is now merged. Junio C Hamano announced Git v2.56.0-rc0, marking a major milestone in the release cycle.

## Notable threads

### Advice subsystem: scope hint simplification
**Thread root**: 2027/08/29/00-49-58

The ongoing discussion about adding scope hints to Git's advice messages took a significant turn today. Vsevolod Myalitsin proposed replacing the previously suggested `scope_hint` enum with a simpler boolean `is_global_hint` field in the `advice_setting` struct. This change would eliminate build failures and focus the implementation on the immediate goal of providing accurate scope hints for `--global` settings like `defaultBranchName`.

Jeff King (Peff) strongly endorsed this simplification, arguing that all advice settings should default to `--global` because advice messages are about silencing chatter for a user rather than a specific repository. He cited historical context from the 2009 introduction of the `advice.*` config system, noting that the original design never intended the scope-less hint to be a literal cut-and-paste command.

Junio C Hamano acknowledged the merit of Peff's position but noted that some users might want to squelch advice per project when working across multiple repositories with different workflows. He clarified that the original design expected users to adapt the suggested command to their needs, leaving the door open for either the boolean simplification or a more flexible mechanism.

The thread also produced a tangential documentation patch from both Junio and Peff to clarify the `<file-option>` placeholder in `git-config.txt`.

**What's at stake**: This change affects how users receive instructions for disabling advice messages. The boolean simplification would make the implementation more robust and focused, but might limit future flexibility if additional scope options become necessary.

### Rust reorganization CI breakage
**Thread root**: 2026/02/04/23-22-08

Junio C Hamano reported a CI breakage in the Rust reorganization effort, specifically in `ci/run-rust-checks.sh`. The script fails because it still expects Rust files in their original `src/` location, while the reorganization moved them to a dedicated `rust/` subdirectory.

Mike Hommey acknowledged the issue as a rebase error and indicated he would provide an update. Junio subsequently removed the topic from the integration branch, pending resolution.

**What's at stake**: This breakage blocks the Rust reorganization effort, which is crucial for Git's ongoing Rustification. The reorganization aims to improve project structure clarity and support downstream projects that vendor Git's C code while excluding its Rust components.

### Memory leak in receive-report hook
**Thread root**: 2026/08/18/07-55-55

Junio C Hamano identified a memory leak in the `override_cmds_error()` helper of the `receive-report` hook implementation. The function overwrites `cmd->error_string` without freeing the existing string if it is owned by the `command` struct.

Karthik Nayak posted v10 of the final patch, introducing a new `error_string_owned` flag to track ownership and conditionally free the existing string before overwriting it. He also committed to adding a test to trigger this error path.

Jeff King confirmed that a previously identified `BUG()` flaw in the series was real but untestable with existing infrastructure, as older clients don't request status reports.

**What's at stake**: This memory leak affects the new `receive-report` hook, which GitLab uses to implement MVCC (multi-version concurrency control) on the server side. The hook allows server administrators to intercept and modify the status report sent to clients after ref updates.

### Git v2.56.0-rc0 release
**Thread root**: 2026/09/10/17-20-21

Junio C Hamano announced the first release candidate for Git v2.56.0. This release includes 635 non-merge commits from 82 contributors, covering new features, performance improvements, bugfixes, and internal refactoring.

Key themes include ODB abstraction (Patrick Steinhardt), reftable backend optimizations (Karthik Nayak, Patrick Steinhardt), repository discovery refactoring (Patrick Steinhardt), Rust integration (brian m. carlson, Taylor Blau), and performance improvements in merge-base computation, ref filtering, and pack-objects.

New user-facing features include `git history drop`, `git refs create/delete/update/rename`, `git replay --linearize`, `git branch --delete-merged`, and `git bisect --reset-when-found`.

**What's at stake**: This release candidate represents a significant step toward Git 2.56, with substantial architectural work that will enable future features like alternative ODB backends and improved scalability.

## In brief

- **[2025/10/10/01-14-05]** Delilah Ashley Wu proposes dropping the Windows path normalization patch entirely and modifying tests to export `XDG_CONFIG_HOME` with forward slashes only, avoiding platform-agnostic code changes.

- **[2026/06/14/14-15-40]** Jeff King confirms the `BUG()` flaw in the `receive-report` hook series was real but untestable with existing infrastructure. Kaartic Sivaraam offers to send a v4 with Peff's proposed commit message wording.

- **[2026/07/17/15-46-58]** Patrick Steinhardt provides substantive review of the `git last-modified` Bloom filter optimization series, identifying correctness issues in `bloom_filter_contains_any_vec()` and naming clarity in `revs_maybe_changed_in_bloom_with_parents()`. He also proposes an alternative design for managing `bloom_filter_settings`.

- **[2026/08/05/14-26-26]** Johannes Schindelin posts v4 of the MinGW/Windows build and runtime adjustments series, a procedural resend with a fly-by style fix split into its own commit.

- **[2026/08/25/14-11-49]** Karthik Nayak provides surface-level and substantive reviews of Patrick Steinhardt's ODB alternates refactoring series, identifying a robustness issue in error handling and raising a question about backend error reporting.

- **[2026/09/03/01-05-47]** Junio C Hamano and Aleksei Sviridkin continue their discussion about the fallback timestamp value in the `--force-if-includes` safety mechanism, with Aleksei providing timing data to support the `0` (epoch start) fallback.

- **[2026/09/03/15-26-44]** Kristoffer Haugsbakk confirms the commit graph as a triggering factor for the memory leak in `git history reword --dry-run`.

- **[2026/09/04/07-53-44]** Thomas Bachem posts v4 of the sequencer auto-maintenance deferral series, tightening commit messages to address Junio's and Patrick Steinhardt's critiques about verbosity.

- **[2026/09/04/21-01-20]** Tyler Cipriani posts v3 of the `--force-if-includes` fix series as a procedural resend to correct threading.

- **[2026/09/07/07-15-57]** Patrick Steinhardt approves v2 of Brigham Campbell's documentation patch fixing AsciiDoc formatting in `git-config(1)`.

- **[2026/09/08/15-24-26]** James Le Cuirot posts v3 of the Rust cross-compilation fix, addressing Junio's feedback from earlier versions.

- **[2026/09/10/06-45-46]** Ariel Keselman posts v2 of the patch avoiding spurious `packed-refs` locks when deleting root refs, incorporating Patrick Steinhardt's feedback about test coverage and the mixed-transaction edge case.

- **[2026/09/10/15-06-07]** Jeff King proposes an alternative solution for merge driver temporary file cleanup using Git's `tempfile` API, eliminating custom signal handling.

- **[2026/09/10/17-07-35]** Junio C Hamano provides surface-level review of Mark C. Chu-Carroll's test modernization patch, pointing to documentation for commit message and patch presentation polish.

- **[2026/09/10/17-39-02]** Junio C Hamano's "What's cooking" report lists new topics needing review, including `vm/advice-config-global-hint`, `jc/rust-cargo-build-target`, and `kn/receive-report-hook`.