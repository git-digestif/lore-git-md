# The Git Project -- Weekly Digest for 2026/11/02 -- 2026/11/08

## The period in brief

The week of 2026/11/02--2026/11/08 saw **heavy, eventful traffic** across seven active days. The mailing list grappled with **stalled patch series at risk of abandonment**, **critical regressions in promisor remote lazy-fetch and commit-graph maintenance**, and **design debates over `git repo info` path keys and HTTP retry logic**. Three developments stand out: the **`git repo info` path keys series reached critical design decisions** but faces abandonment due to lack of reviewer engagement; **Junio C Hamano formalized a new `seen` branch ejection policy** to balance reviewer productivity with quality control; and **a 15-patch series fixed segfaults on unvalued config keys** across 17 crash sites, hardening Git’s config parsing.

---

## Key developments

### `git repo info` path keys series at risk of abandonment
The v7--v8 series adding seven new path-related keys (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.git-prefix`, `path.cdup`) to `git repo info` saw **key design resolutions** but remains stalled due to persistent lack of reviewer engagement. K Jayatheerth and Justin Tobler agreed to special-case `/dev/null` handling for `path.hooks.absolute` and `path.hooks.relative`, returning `/dev/null` verbatim when `core.hooksPath` is set to that value. However, **Phillip Wood raised high-weight design objections** to the v8 series, questioning the necessity of `path.hooks` (arguing scripts should use `git hook run` or config-based hooks instead) and the behavioral consistency of `path.cdup` at repository root. The series is now at **critical risk of abandonment** unless reviewers step up, despite being mechanically complete and addressing all prior feedback.

The series targets the `git repo info` command (a GSoC project) and aims to expose filesystem locations of repository components in a scriptable format. The practical stakes are **script portability and usability**: the keys would allow scripts to reliably locate repository components without fragile path manipulation. The unresolved design objections—particularly around `path.hooks` and `path.cdup`—must be addressed before merging, as they affect core functionality and script behavior.

---

### Integration pipeline policy update: `seen` branch ejection formalized
Junio C Hamano adopted **Phillip Wood’s suggestion** to explicitly frame ejection from the `seen` branch as a **reversible, routine event** rather than a final rejection. The new policy, announced in "What's cooking" reports, states that topics ejected from `seen` can be resubmitted if they gain enough interest and demonstrate good fit. The change addresses **contributor anxiety** by signaling that ejection is temporary and contingent on renewed engagement, rather than a permanent rejection.

The policy targets the **integration pipeline** and aims to balance **reviewer productivity** with **quality control**. The practical stakes are **contributor morale and patch throughput**: the new wording reduces the stigma of ejection and encourages resubmission of stalled topics. The decision is technically important because it formalizes a long-standing informal practice, making the integration process more transparent and predictable.

---

### Segfaults on unvalued config keys fixed across 17 crash sites
Thomas Bachem posted a **15-patch series** fixing segfaults when config settings are given without values (e.g., `git -c foo.bar` or bare `bar` under `[foo]`), covering 17 crash sites across commands like `git rebase -i`, `git fetch`, `git add -p`, and `git clone --bundle-uri`. The series **systematically eliminates crashes** triggered by unvalued config keys, hardening Git’s config parsing against NULL dereferences. Most fixes reject missing values with `config_error_nonbool()`, following the precedent set by ba176db511 (2023-12-07). Three exceptions handle the absence gracefully: `fetch.bundleCreationToken` defaults to 0, remote URLs skip `includeIf` matching, and `git config list` omits the key entirely.

The series targets **config parsing** (`config.c`, `repo_config_get_value()` callers) and addresses a **systemic issue** affecting 17 distinct crash sites. The practical stakes are **stability and security**: the fixes prevent crashes in common workflows and eliminate a server-triggered crash via bundle lists (patch 14/15). The series is based on `master` and merges cleanly, though it conflicts with two topics in `seen` due to overlapping test additions. No reviews have been posted yet, but the patches are **mechanically sound** and follow established patterns.

---

### Promisor remote lazy-fetch regression fixed
Patrick Steinhardt posted a **two-patch series** fixing a critical regression in promisor remote lazy-fetch that was causing `git merge` to fail with "not our ref" errors. The issue stemmed from **stale loose object cache entries** not being updated when new objects were written during merges, a regression introduced during ODB abstraction work. The fix ensures the loose object cache is properly invalidated after writing merged blobs, addressing a blocker for CI workflows in projects like Kubernetes that rely on blobless partial clones.

The series targets the **ODB abstraction layer**, **promisor remote lazy-fetch mechanism**, and **loose object cache**. The practical stakes are **high for users of partial clones**: the regression manifested only when merging an object whose first two hex characters matched those of a newly written object in the same subdirectory, causing unnecessary and failing backfill fetches. The fix is **minimal and targeted**, updating the loose object cache’s oidtree after writing objects to listed subdirectories. Junio Hamano endorsed landing the current patch as a **targeted regression fix** while documenting broader `QUICK` flag limitations via a `NEEDSWORK` comment in `odb.h`.

---

### Commit-graph corruption in Git 2.54’s geometric maintenance strategy
Christoph Weiss reported a **critical bug** in Git 2.54’s geometric maintenance strategy where `git maintenance run --auto` prunes unreachable commits but fails to update the commit-graph, leaving it in an inconsistent state. The bug affects operations that rely on the commit-graph, such as `git log --graph` and `git merge-base`. The reproduction script is self-contained and demonstrates the corruption via `git fsck`.

The bug targets the **commit-graph** and **maintenance** subsystems and is a **regression** introduced by the geometric maintenance strategy (now the default). The practical stakes are **high for users upgrading to Git 2.54**: the corruption affects core functionality and could impact performance and correctness. The root cause is that the `geometric` strategy prunes unreachable commits without updating the commit-graph, violating the invariant that the commit-graph must reflect the current state of the object database. No fix has been posted yet, but the report is **actionable** and includes a minimal reproduction script.

---

### HTTP retry logic configuration debate
Brian M. Carlson and Ravi Mistry debated the **default behavior of Git’s HTTP retry logic**, with Carlson arguing that even 429 responses should trigger a non-zero initial delay to avoid overwhelming rate-limited servers. The discussion settled on a **10-second default delay** (`http.retryAfter=10`) with exponential backoff (10s, 20s, 40s, ...), striking a balance between server protection and interactive usability. The series adds exponential backoff for HTTP retries, makes retryable status codes configurable, and extends retries to HTTP POST requests.

The series targets the **HTTP transport** (`http.c`, `remote-curl.c`) and addresses a **measurable performance issue** (tight retry loops) while extending retry logic to POST requests, which carry the bulk of Git’s data transfer. The practical stakes are **resilience and server protection**: the 10-second default protects rate-limited hosting backends while maintaining usability. Junio C Hamano raised a **procedural concern** about the two-phase default value introduction (patches 1–4 assume 0, patch 5 sets 10s), questioning whether it is the right way to structure the series for maintainability. The v3 series is **technically complete** and awaits Junio’s final review.

---

### `git describe` regression fix converges on opt-out `--exact` flag
Linus Torvalds’ patch fixing `git describe`’s commit distance calculation when commit timestamps are non-monotonic gained consensus on its user-facing design. The discussion settled on an **opt-out `--exact` flag** (defaulting to on) with a warning mechanism when the commit graph is unavailable. The patch addresses a **real-world regression** in the Linux kernel’s version reporting during the v7.4 merge window, where `git describe` incorrectly reported commit distances due to skewed commit dates caused by GitLab’s Marge-bot workflow.

The patch targets the **commit graph** and **revision walking** subsystems and is a **regression fix** for repositories with non-monotonic commit dates. The practical stakes are **high for users of `git describe`**: the regression affects version reporting in scripts and CI workflows. The opt-out `--exact` flag ensures backward compatibility while warning users when the commit graph is unavailable. The patch is now **queued for integration**, addressing a long-standing issue in Git’s version reporting.

---

## In brief

**`parse-options` early scanning API** -- Christian Couder’s series introducing a minimal, reusable `parse-options` sub-API (`early_scan_options()`) was marked for `next` after addressing all prior feedback, including validation checks, ephemeral `option` pointer documentation, and test coverage for the `git fast-import` bugfix. The series replaces fragile hand-rolled early argument scans with a robust, reusable API.

**`git grep --recurse-submodules` fix** -- Patrick Steinhardt’s three-patch series fixing `git grep --recurse-submodules` for nested submodule trees was marked for `next` after addressing all review feedback. The fix targets a regression introduced in Git v2.56.0 and is explicitly intended for backporting to stable branches.

**`git annotate` completion fix** -- Tanga Joris’s patch fixing command-line completion for `git annotate` was approved by Junio C Hamano and marked ready for `next`. The fix removes redundant `-c` prepending in `cmd_annotate()`, ensuring `git annotate --<TAB>` offers the correct options.

**`git apply` crash regression fix** -- Aniket Gupta’s fix for a crash in `git apply` when `core.fileMode=false` addressed a regression where patch series containing a pure rename followed by an edit failed with `error: renamed: wrong type`. The fix prevents tracking a start point that has no ref and ensures consistent behavior across workflows.

**`git unpack-objects` type mismatch fix** -- Kristofer Karlsson’s fix for a type mismatch in `unpack-objects` addressed a regression where blob sizes larger than 4GB were truncated on 32-bit platforms (notably Windows). The fix changes `stream_blob()` to accept `size_t`, ensuring correct handling of large objects.

**`git rev-list` option parsing fix** -- Azuama Odinaka Maximus’s fix for `git rev-list`’s `--grep` and related option parsing expanded to cover all 18 `parse_long_opt()`-style options, ensuring consistent argument parsing when followed by another option-like string.

**`git rerere` subcommand modernization** -- Patrick Steinhardt’s 12-patch series converting `git rerere` to use `OPT_SUBCOMMAND()` received surface-level review, with Karthik Nayak suggesting minor improvements to enum documentation and code structure.

**`git log --follow` performance regression** -- Patrick Steinhardt identified a potential regression in Miklos Vajna’s performance series, where `follow_needs_renames()` disables rename detection for commits with unrelated histories, contradicting the intent of commit 304812ed33.

**`git maintenance` lock-handling consistency** -- Patrick Steinhardt’s series improving `git maintenance` lock-handling agreed to split into two patches (behavioral change and Trace2 event) after review, ensuring consistent error-message formatting and simplifying the Trace2 event’s `reason` field.

**`git credential-cache` daemon race condition fix** -- Farid Zakaria’s fix for a race condition in the Git credential-cache daemon acknowledged Junio’s architectural concern about mixing `unix_ss_*` and `tempfile` APIs, but did not engage with the concern, making a v3 or maintainer intervention likely.

**`git bundle-uri` URI handling regression fix** -- Ravi Mistry’s patch fixing a crash in `git bundle-uri` when processing relative URIs with excessive `../` components addressed Patrick Steinhardt’s review feedback and was marked ready for `next`.

**`git reflog expire` refactoring** -- Laveesh Bansal’s patch refactoring `git reflog expire` to consolidate two loops into one addressed Junio’s suggestion from d1536ac3e2, improving code clarity and maintainability.

**`git column` argument parsing** -- Fien Dora proposed refactoring `git column` to use `early_scan_options()`, seeking maintainer input on whether to preserve current restrictions or expand to full `parse_options()` flexibility.

**Git v3 compatibility** -- Adam Dinwoodie reported that `git init` incorrectly defaults to SHA-1 even when Git is built with `WITH_BREAKING_CHANGES=1`. brian m. carlson confirmed the bug and is preparing a fix, which must handle reinitialization safely to avoid silently overwriting object formats.

**Reftable backend HEAD reflog bug** -- 井上司 reported a correctness regression in the reftable backend where HEAD reflog entries are either written with the wrong OID or omitted entirely when HEAD is updated in the same transaction as its target branch. The bug affects reflog auditing and recovery, and a targeted fix is in progress.

**Outreachy proposals** -- Cynthia Uche and Tanga Joris posted strong technical proposals for Outreachy, focusing on **Git command argument scanning** and **global state reduction in pathspec skip-worktree matching logic**. Both proposals demonstrated thorough preparation and clear technical goals.

**CI optimizations** -- Patrick Steinhardt’s seven-patch series targeting GitLab CI bottlenecks received maintainer acknowledgment, with Junio finding all patches "interesting" and "fully made sense." The series aims to improve CI performance and reliability.

---

## Looking ahead

The next week is likely to see **progress on stalled patch series**, particularly the `git repo info` path keys series and the HTTP retry logic configuration. The **commit-graph corruption bug** in Git 2.54’s geometric maintenance strategy will likely see a fix, given its high impact and clear reproduction. **Outreachy proposals** will continue to generate discussion, with several focusing on **global state reduction** and **promisor remote improvements**. The **reftable backend HEAD reflog bug** is also likely to see a fix, given its impact on reflog auditing and recovery. Finally, **Junio C Hamano’s integration pipeline policy** will continue to shape how stalled topics are handled, with the new `seen` branch ejection wording likely to appear in upcoming "What's cooking" reports.