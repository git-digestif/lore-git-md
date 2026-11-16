# The Git Project Weekly Digest
**Period: 2026/11/09 -- 2026/11/15**

## The period in brief
This week saw **14 active days** of mailing list traffic with **heavy development activity** across multiple fronts. The Git project made **decisive progress** on long-running feature series, including the `git repo info` path keys and `fetchRefmap` enhancements, while also addressing **critical bugfixes** and **performance regressions**. The most consequential developments were the **final approval of the `git repo info` path keys series**, the **resolution of the `git log --follow` performance regression** with an opt-in toggle, and the **advancement of the `change-id` preservation debate** to a security-focused inflection point. Readers should not miss the **security release (Git 2.56.1)** addressing six CVEs, the **integer-overflow hardening efforts**, and the **ongoing discussion about AI-assisted development partnerships**.

---

## Key developments

### `git repo info` path keys series lands
The **nine-iteration series** adding path-related keys to `git repo info` received **final approval** and is ready for merging. The series introduces four new keys—`path.working_tree`, `path.superproject_working_tree`, `path.index`, and `path.prefix`—that expose filesystem locations of repository components in a scriptable format. Phillip Wood confirmed the **range-diff between v8 and v9 is mechanically correct** and stated, *"I'd be happy if this is merged as-is."* The keys support suffix-based formatting (absolute or relative paths) and return empty strings in bare repositories, except for `path.index`, which aligns with `git rev-parse --git-path` precedent. The series resolves all prior objections, including naming conventions (underscores over hyphens) and test efficiency concerns.

**Key participants**: K Jayatheerth (author), Phillip Wood (reviewer), Junio C Hamano (maintainer)
**Status**: Ready for `next`; uncontroversial after design resolution

---

### `git log --follow` performance regression resolved
A **months-long debate** about a **14× slowdown** in `git log --follow` across merges reached resolution with an **opt-in toggle** approach. The regression, introduced in June 2026, made the feature unusable at scale by enabling rename detection for all merge parents. Miklos Vajna’s patch adds `--follow=merges` (or `log.follow=merges`) to restore pre-regression performance while preserving correctness for users who need it. Junio C Hamano **ejected a prior speedup branch** and queued the toggle as the **project-level resolution**, settling the policy question after extensive discussion about performance vs. correctness trade-offs. The toggle unblocks GitLab’s deployment while maintaining the correctness improvements.

**Key participants**: Miklos Vajna (author), Patrick Steinhardt (reviewer), Junio C Hamano (maintainer)
**Status**: Queued for `next`; toggle patch fast-tracked

---

### `change-id` preservation: security vs. workflow
The **long-running discussion** about preserving `change-id` headers during commit rewriting operations reached a **critical inflection point** this week. Ewout ter Hoeven countered privacy and utility objections with **concrete workflow use cases** from GitHub’s `gh-stack` and GitLab’s `glab stack`, where tools struggle to maintain commit identity across rewrites. However, brian m. carlson raised a **fundamental security concern**: **change-id headers cannot be trusted as stable identifiers** because users can manually edit them, creating a risk for automated review systems. This reframes the problem as a **security boundary question**, with no clear path to reconcile workflow needs with security requirements. The thread now hinges on whether to restrict `change-id` preservation to server-controlled operations or accept that the header cannot be trusted for security-sensitive decisions.

**Key participants**: Ewout ter Hoeven (proponent), brian m. carlson (opponent), Junio C Hamano (maintainer)
**Status**: Design debate ongoing; no patches in flight

---

### Security release: Git 2.56.1 addresses six CVEs
Junio C Hamano announced a **coordinated security release** addressing six CVEs, including:
- **CVE-2026-111760**: Remote code execution via malicious submodule gitfiles
- **CVE-2026-113408**: Path traversal in files ref backend
- **CVE-2026-114467**: Stack corruption from multi-signature commits

The vulnerabilities span **submodules, ref backends, and merge machinery**, with potential for denial-of-service or arbitrary code execution. Fixes are backported to all supported maintenance tracks (2.56.1 down to `maint-2.43`). Key contributors included **Toon Claes** (protocol v2 symref validation), **Karthik Nayak** (multi-signature handling), and **Jeff King** (30 patches for const-correctness and HTTP header redaction). Git for Windows released **v2.56.1** with identical fixes.

**Key participants**: Junio C Hamano (maintainer), Toon Claes, Karthik Nayak, Jeff King
**Status**: Released; fixes integrated into all maintenance tracks

---

### Integer-overflow hardening advances with API changes
Johannes Schindelin’s **integer-overflow hardening series** made **significant progress** this week, addressing Phillip Wood’s review of the wrapper and date subsystems. Key updates include:
1. **API contract change**: `write_in_full()` now returns `0` on success (eliminating overflow risk entirely), and a new `read_in_full_or_error()` helper is introduced for callers needing only success/failure status.
2. **Macro removal**: The `signed_mult_overflows` and `mult_overflows` macros were **dropped entirely** due to GCC-only limitations and operand-width assertion concerns.
3. **Portable date.c hardening**: The original implementation was withdrawn in favor of a simpler approach—capping user-supplied numbers at `INT_MAX` and guarding multiplications manually—to avoid compiler-specific overflow detection.

The series now focuses on **portable, manual overflow checks** and **API-level hardening**, with the wrapper subsystem changes ready for integration.

**Key participants**: Johannes Schindelin (author), Phillip Wood (reviewer)
**Status**: Wrapper subsystem patches ready for `next`; date.c hardening revised

---

### `git apply --check` crash fix polishes for integration
The **two-part bugfix series** for a crash in `git apply --check` when processing malformed patch headers gained **final polish** this week. The series now **rejects corrupt patch headers outright**, aligning with Junio’s guidance to enforce strict parsing rules. Key improvements include:
- **Memory-leak fix**: Siddharth Asthana identified a leak in `parse_git_diff_header()` where `patch->def_name` was overwritten without freeing the old value. The fix adds a `free()` before assignment, with both callers already zeroing the `patch` struct.
- **Error-message clarity**: Messages now reference the problematic header line (e.g., `"---"` before `"new file mode"`) instead of stripped filenames, making them more actionable for users.

The series is **technically sound**, with clear error messages and comprehensive test coverage, and is ready for final review.

**Key participants**: Tanga Joris (author), Siddharth Asthana (reviewer)
**Status**: Ready for `next`; memory-leak and error-message improvements addressed

---

### `fetchRefmap` series advances to v14 with strategic clarity
Harald Nordgren posted **v14 of the `fetchRefmap` series**, introducing `remote.<name>.fetchRefmap` and a matching-inspired fetch mode for shallow repositories. The series is now down to **five patches** after squashing the `@{push}` resolution fix into Patch 3/5. The cover letter **explicitly ties the series to Matt Hunter’s clarification** that Patches 1–3 are sufficient to resolve textual conflicts with Collin’s `ch/remote-defer-follow-remote-head-validation` topic, strengthening the case for a **partial merge**. The architectural objection to Patch 3/5 (dynamic branch enumeration) is weakened by this context, as the patch is required to resolve the conflicts. The series is otherwise ready for final review, with no known blockers beyond the ongoing strategic discussion about whether to merge Patches 1–3 now or insist on the full series.

**Key participants**: Harald Nordgren (author), Junio C Hamano (maintainer), Matt Hunter (reviewer)
**Status**: Ready for final review; strategic discussion ongoing

---

### `git branch --delete-integrated` series reaches v7
Harald Nordgren posted **v7 of the series** teaching `git branch --delete-integrated` to detect branches whose changes were squash-merged or rebase-merged into an upstream branch, even when the upstream later modified the same files. The series is now **technically complete**, with all substantive review issues addressed. Key improvements include:
- **Performance optimization**: The fourth patch now reads the index once before the walk in `find_landed_commits()`, eliminating redundant `repo_read_index()` calls.
- **Test coverage**: Added test cases for edge-case robustness (branches whose changes never landed, landed only partially, or conflicted with upstream edits).
- **Documentation**: The commit message explicitly documents the `.gitattributes` resolution behavior and the fact that `--delete-integrated` may download blobs in partial clones.

The only remaining technical gap is the **author-check mitigation** (Jasper’s proposal), which is not yet implemented but does not block integration.

**Key participants**: Harald Nordgren (author), Phillip Wood (reviewer)
**Status**: Ready for `next`; author-check mitigation deferred

---

## In brief

**`git rerere` subcommand modernization** -- Patrick Steinhardt’s 12-patch series modernizing `git rerere`’s subcommand handling was **merged to `master`**. The series replaces custom subcommand-parsing logic with Git’s standard `OPT_SUBCOMMAND()` mechanism, aligning it with other multi-subcommand built-ins. It also adds a `BUG()` check to catch invalid flag combinations and per-subcommand usage strings. **Key participants**: Patrick Steinhardt, Karthik Nayak.

**Config-related segfaults systematically fixed** -- Thomas Bachem’s 15-patch series eliminates Git segfaults when config settings are given without values (e.g., `git -c foo.bar`). The series audits config callbacks and `repo_config_get_value()` callers to handle missing values gracefully, addressing 23 distinct crash sites. **Key participants**: Thomas Bachem, Junio C Hamano, Derrick Stolee, Phillip Wood.

**Refactor of `t9300-fast-import.sh` advances** -- Karthik Nayak posted v2 of a 20-patch series splitting the monolithic `t9300-fast-import.sh` (4000+ lines) into smaller, self-contained test files. The series makes each test run in a fresh repository with consistent configuration, improving maintainability. **Key participants**: Karthik Nayak, Junio C Hamano.

**Object format interoperability bugs addressed** -- Adrian Ratiu posted a 3-patch series fixing bugs in Git’s `compatObjectFormat` interoperability layer, which handles conversions between SHA-1 and SHA-256 repositories. The series addresses a crash when `compatObjectFormat` and `objectFormat` are set to the same value, silent incorrect OID returns, and inconsistent handling of unknown commit headers. **Key participants**: Adrian Ratiu, Junio C Hamano.

**Promisor-remote protocol v2 capability fixed** -- Christian Couder posted a 3-patch series fixing a bug where the Git client omits the `promisor-remote=...` reply during `fetch` protocol v2 commands. The fix enables reliable partial clone operations with servers that don’t maintain state between requests. **Key participants**: Christian Couder, Junio C Hamano.

**`git ls-remote --show-object-format`** -- Junio C Hamano confirmed the safety of the implementation and endorsed the test coverage as adequate, moving this feature closer to integration. The patch enables tools like Jujutsu to detect a remote repository’s hash algorithm without cloning. **Key participants**: Patrick Steinhardt, Junio C Hamano.

**GitLab CI speedups** -- Johannes Sixt reported successful testing of Patrick Steinhardt’s CI optimization series on Windows hardware, confirming significant runtime reductions (32% for `t0027`, 11% for `t1092`). **Key participants**: Johannes Sixt, Patrick Steinhardt.

**Detached HEAD documentation** -- Julia Evans posted v6 of the documentation rewrite, addressing final style nits and test expectations. The series creates a standalone guide explaining detached HEAD in plain language. **Key participants**: Julia Evans, Junio C Hamano.

**Glossary terminology alignment** -- Andy Koppe’s v4 series addressing the 2024 terminology change that narrowed "pseudoref" to `FETCH_HEAD` and `MERGE_HEAD` only received final sign-off from Junio C Hamano. **Key participants**: Andy Koppe, Junio C Hamano.

**`git credential` conversion to `OPT_SUBCOMMAND`** -- Wasswa Edson posted v2 of the series converting `git credential` to use `OPT_SUBCOMMAND` from the parse-options API. The series is split into three patches: aligning `-h` output with the SYNOPSIS, refactoring subcommands into their own functions, and switching to `OPT_SUBCOMMAND`. **Key participants**: Wasswa Edson, Junio C Hamano.

**`git worktree list` improvements** -- Jasper Mayone’s v5 series extending `git for-each-ref`’s `%(worktreepath)` atom and improving worktree safety gained traction. The series now uses a hashmap for O(1) lookups and introduces `%(currentworktree)` for conditional formatting in `git branch`. **Key participants**: Jasper Mayone, Phillip Wood.

**`git history --verbose` output format debate** -- The discussion about adding diff display to `git history reword` shifted to the **output format**, with Patrick Steinhardt requiring it be generalizable to all `git history` subcommands. He suggested `git range-diff`’s structured format as a model. **Key participants**: Patrick Steinhardt, Jelle van der Waa, Karthik Nayak.

**ODB transaction temporary directory creation** -- Justin Tobler’s series to make temporary directory creation eager in ODB transactions faced design scrutiny. Karthik Nayak questioned the motivation for switching from lazy to eager creation, while Patrick Steinhardt endorsed the direction. **Key participants**: Justin Tobler, Karthik Nayak, Patrick Steinhardt.

**Bugfix for `git cat-file --batch` with invalid reflog references** -- Caspar Duregger necroposted a four-patch series systematically applying `GET_OID_GENTLY` to make `git cat-file --batch` error handling uniform for invalid reflog references. The series addresses a regression introduced by commit 393bbb21. **Key participants**: Caspar Duregger, Junio C Hamano.

**`git ls-remote` promisor-remote crash fix** -- Christian Couder posted a patch to fix a crash in `git ls-remote` when invoked outside any Git repository with promisor-remote configuration enabled. Junio C Hamano proposed an alternative approach that makes `promisor_remote_reply()` a no-op when no repository is present. **Key participants**: Christian Couder, Junio C Hamano.

**Partnership proposal for AI-assisted development on Git for Windows** -- Weifang Leiming Cloud Network Technology Co., Ltd. proposed a business partnership to provide AI-driven development support for Git for Windows. The proposal sparked procedural discussion about the appropriateness of such partnerships on the mailing list. **Key participants**: Weifang Leiming, Johannes Schindelin, Junio C Hamano.

**`git diff` shell function context** -- Junio C Hamano confirmed a bug in `git diff`’s context header logic for shell scripts where function definitions span multiple lines. The issue is identified as a configuration problem—Git does not auto-detect shell scripts, so the generic hunk header pattern is used unless `diff=bash` is explicitly configured. **Key participants**: Windl, Ulrich, Johannes Sixt.

**`struct commit` memory optimization** -- Kristofer Karlsson’s patch to reduce `struct commit` size by 10% on 64-bit platforms received final maintainer approval. The patch reorders fields to eliminate padding, reducing memory usage by 10% (from 80 to 72 bytes). **Key participants**: Kristofer Karlsson, Junio C Hamano.

**`git repo structure` performance improvements** -- Patrick Steinhardt posted a two-patch series improving performance and reducing memory usage of `git repo structure`. The patches achieve an 80% memory reduction (5.1× less memory) and 20% wall-time improvement by delaying parsing until necessary and skipping full tree parsing when only the object type is required. **Key participants**: Patrick Steinhardt, Kristofer Karlsson.

---

## Looking ahead
The next week is likely to see **integration of several ready-for-`next` series**, including the `git repo info` path keys, `git log --follow` performance toggle, and `git apply --check` crash fix. The **`change-id` preservation debate** may see new proposals to reconcile workflow needs with security requirements, potentially involving server-side enforcement or cryptographic signing. The **integer-overflow hardening series** is expected to advance further, with the wrapper subsystem patches likely landing and the date.c hardening undergoing final revisions. The **`fetchRefmap` series** may see a partial merge of Patches 1–3 to resolve conflicts with other in-flight topics, while the full series continues to be debated. Finally, the **AI-assisted development partnership proposal** may generate further discussion about the project’s stance on external collaborations.