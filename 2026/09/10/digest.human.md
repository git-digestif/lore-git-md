# Git mailing list daily digest for 2026/09/10

## The day in brief
The Git project saw active development across multiple fronts today. Key highlights include a proposed simplification for the `advice: add scope hint` series, a critical memory leak fix in the `receive-report` hook, and ongoing refinements to Rust infrastructure and ODB abstraction. Several bugfixes and documentation improvements also progressed, with maintainers signaling readiness to queue several series.

## Notable threads

### Advice scope hint simplification
**Thread root**: 2027/08/29/00-49-58

The ongoing discussion about adding scope hints to Git's advice system reached a potential turning point. Vsevolod Myalitsin proposed replacing the `scope_hint` enum with a simple boolean `is_global_hint`, arguing that no realistic scenario requires `--system` or other scopes for advice settings.

Jeff King (Peff) strongly endorsed this simplification, stating: "I think this is the right direction. The YAGNI principle applies here - we don't have a use case for anything but --global, so let's not complicate the code with it." He cited historical context from the 2009 introduction of the `advice.*` config system, noting that scope was never debated in prior discussions.

Junio C Hamano acknowledged the merit of Peff's position but noted that some users might want per-project advice settings: "I can see that some users may want to squelch advice per project, even though such cases may be rare." The maintainer clarified that the original design never intended the scope-less hint to be a literal command, leaving the door open for either approach.

The proposed boolean simplification would resolve build failures and eliminate the need to handle unhandled `CONFIG_SCOPE` values, while still addressing the original goal of accurate scope hints for `--global` settings like `defaultBranchName`.

### Receive-report hook memory leak fix
**Thread root**: 2026/08/18/07-55-55

Karthik Nayak posted v10 of the `receive-report` hook series, addressing a memory leak in the `override_cmds_error()` helper identified by Junio C Hamano. The patch introduces a new `error_string_owned` flag to track ownership of `cmd->error_string` and conditionally free the existing string before overwriting it.

The series is now feature-complete with all prior feedback incorporated. Jeff King confirmed that a previously identified `BUG()` flaw was real but untestable with existing infrastructure, as older clients don't request status reports. Karthik declined to add an interop test, citing practical limitations.

Junio had previously queued the series in `next`, and with this final polish, it appears ready for graduation to `master`. The hook enables server administrators to filter or modify the status report sent to clients after ref updates, with GitLab's MVCC use case as the primary motivation.

### Rust infrastructure updates
**Thread root**: 2026/02/04/23-22-08

The Rust reorganization effort saw continued progress and some setbacks. Junio C Hamano reported a CI breakage in `ci/run-rust-checks.sh` due to unupdated path expectations after the Rust code was moved to a `rust/` subdirectory. The maintainer kicked the series out of `seen` to await updates.

Mike Hommey acknowledged the issue as a rebase error and noted that the v4 patch addresses all prior mechanical feedback. The series aims to improve project structure clarity in Git's mixed C/Rust codebase while preserving build functionality.

The discussion also touched on forward-compatibility for downstream projects that vendor Git's C code while excluding its Rust components, with Mike proposing to publish the `gitcore` crate to crates.io as a long-term solution.

### ODB alternates refactoring
**Thread root**: 2026/08/25/14-11-49

Patrick Steinhardt posted v5 of the ODB alternates refactoring series, incorporating minor naming and documentation improvements from Karthik Nayak's feedback. The series removes the ability to write ODB alternates after repository creation, simplifying the ODB interface.

Karthik's substantive review of patch 7/9 identified a potential robustness issue in the alternates-writing loop, suggesting to check `ferror()` after each `fprintf()` call rather than once at the end. Patrick acknowledged this feedback but noted it doesn't block integration.

The series is now well-documented and addresses all substantive review feedback, making it a strong candidate for integration. Junio previously signaled intent to replace the v3 series in `next` with this version.

## In brief

- **Global config listing inconsistency**: Delilah Ashley Wu proposed dropping the Windows path normalization patch entirely and instead modifying tests to export `XDG_CONFIG_HOME` with forward slashes only, avoiding platform-agnostic code changes.
- **Memory leak in git history**: Kaartic Sivaraam offered to send a v4 with Jeff King's proposed commit message wording as a strict improvement, though no functional changes are needed.
- **--force-if-includes bugfix**: Tyler Cipriani posted v3 of the series as a procedural resend to correct threading, with no technical changes from the ready-to-merge v2.
- **Documentation improvements**: Brigham Campbell posted v2 of a documentation patch fixing AsciiDoc formatting, incorporating Junio's requested quoting fixes and subject-line tweak.
- **Merge driver temporary files**: Jeff King proposed an alternative solution using Git's `tempfile` API to handle signal-safe cleanup of merge driver temporary files, eliminating custom signal handling.
- **Test modernization**: Mark C. Chu-Carroll posted a patch updating `t/t4010-diff-pathspec.sh` to use modern test style, reducing the file from 88 to 31 lines.
- **Git v2.56.0-rc0**: Junio C Hamano announced the first release candidate for Git v2.56.0, summarizing 635 non-merge commits from 82 contributors.
- **Refs/files-backend bugfix**: Ariel Keselman posted v2 of a patch avoiding spurious `packed-refs` locks when deleting root refs, incorporating Patrick Steinhardt's feedback about test coverage and edge cases.