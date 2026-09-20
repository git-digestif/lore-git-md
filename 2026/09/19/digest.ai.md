# Git mailing list daily digest for 2026/09/19

## The day in brief
The Git mailing list saw active discussion on several fronts today. A bugfix for `git stash` autostash handling with staged index entries was submitted, addressing a long-standing issue with `stash.index=true`. Outreachy project planning advanced with the approval of a third project, "Implement promisor remote fetch ordering," and mentor assignments were clarified. A regression in the `reference-transaction` hook for branch renames was diagnosed, with a contributor offering to submit a fix. Meanwhile, a proposed "safe" `strbuf` API faced substantive critique over naming, usability, and a safety-violating bug.

## Notable threads

### Outreachy December 2026 cohort: project approval and mentor assignments
**What changed?**
Christian Couder approved the third Outreachy project, "Implement promisor remote fetch ordering," and offered to co-mentor if needed. The project, tentatively mentored by Kaartic Sivaraam, aims to improve partial-clone functionality by allowing configurable fetch ordering for promisor remotes.

**Background**
Git is participating in the Outreachy December 2026 cohort with three approved projects. The other two projects—"Improve how command arguments and options are scanned and parsed" and "Reduce Git’s global state to enable Git's libification"—already have confirmed mentors. The Outreachy application deadline is October 5, 2026.

**Why it matters**
The addition of the promisor-remote project expands internship opportunities and addresses a performance optimization for partial clones, a growing area of interest in Git’s scalability efforts. The mentor assignments are now complete, though they remain tentative pending intern applications.

---

### `stash.index=true` + `--autostash` bugfix: final polish and submission
**What changed?**
D. Ben Knoble incorporated Phillip Wood’s review feedback and submitted a formal patch series to fix a bug where `git stash` autostash fails to correctly handle staged index entries when `stash.index=true` is set. The series replaces subprocess-based index merging with in-core logic via `merge-ort.c`, eliminating a race condition that could corrupt the `MERGE_AUTOSTASH` ref.

**Background**
The bug manifests during merge operations that trigger autostash, causing incorrect index state preservation. The fix targets `builtin/stash.c` and `merge-ort.c`, with test coverage added to `t/t7600-merge.sh`. Phillip Wood’s earlier feedback identified the root cause (cleared stat data and `CE_UPTODATE` flags) and suggested targeted improvements, which Knoble has now implemented.

**Why it matters**
This is a correctness fix for a real-world annoyance affecting users of `stash.index=true` and `--autostash`. The in-core approach improves performance and reliability, and the added test coverage ensures the bug won’t regress. The series is now ready for review, with no open technical questions beyond a minor design choice about `assert()` retention in `merge-ort.c`.

---

### "Safe" `strbuf` API: substantive critique and safety bug
**What changed?**
Phillip Wood raised three key concerns about Derrick Stolee’s RFC series introducing a "safe" `strbuf` API that avoids `die()` calls:
1. The "safe" framing is vague; the API should instead be described as error-returning, with naming aligned to existing patterns (e.g., `strbuf_*_gently`).
2. Usability is poor due to the need to check errors on every call; a sticky error bit (like C’s `stdio` functions) would simplify error handling.
3. A bug in `sstrbuf_grow()`: it uses `st_add3()`, which calls `die()` on overflow, violating the safety guarantee.

**Background**
The series aims to provide a subset of the `strbuf` API for critical code paths like trace2, where crashes are unacceptable. It introduces new methods (`sstrbuf_init()`, `sstrbuf_release()`, `sstrbuf_grow()`) and refactors `json-writer.c` to use them. The RFC is complete at patch 6/6, with CodeQL used to verify the safety property.

**Why it matters**
The critique undermines the series’ core premise. The naming and usability issues could hinder adoption, while the `st_add3()` bug invalidates the safety guarantee. The discussion now centers on whether the API can be salvaged with targeted fixes or if a broader redesign is needed. The thread highlights the tension between safety, usability, and backward compatibility in Git’s low-level APIs.

---

### `reference-transaction` hook: branch rename regression diagnosed
**What changed?**
Karthik Nayak diagnosed a regression where the `reference-transaction` hook omits the new ref when a branch is renamed (`git branch -m`). The hook only reports the deletion of the old branch name, not the creation of the new one, due to branch renames bypassing the ref transaction mechanism entirely.

**Background**
Maciej Ciemborowicz reported the bug, noting it affects both the files and reftable backends. Karthik traced the issue to backend-specific behavior: the files backend uses `refs_delete_ref()`, which triggers the hook for the deletion, while the reftable backend writes a TOMBSTONE entry directly, skipping the hook. The expected behavior—confirmed via `git update-ref --stdin`—is that both deletion and creation events should be included in the same transaction.

**Why it matters**
The `reference-transaction` hook is relied on by external tools (e.g., Gerrit, GitLab) to monitor ref changes. The regression breaks workflows that depend on tracking branch renames, and the backend inconsistency complicates the fix. Karthik’s diagnosis provides a clear path forward: update the branch-rename logic to use transactions, ensuring the hook receives both events. He has offered to submit a patch, signaling this is likely to be resolved soon.

---

### GitLab CI breakage: Windows Rust support validated
**What changed?**
Karthik Nayak set up an internal GitLab merge request (MR #671) and pipeline (#2863888081) to validate Johannes Schindelin’s patch series fixing GitLab CI breakage for Windows builds after Rust support was enabled. The series addresses missing Rust toolchain provisioning (`cargo`), silent data loss in dependency setup, and linker unavailability for MinGW builds.

**Background**
The breakage occurred after Rust was enabled in Windows CI jobs, causing build failures due to `cargo: command not found` errors and missing GNU Rust host-linker support. Schindelin’s series introduces a `-Mingw` switch in `ci/install-dependencies.ps1`, preserves `.git/info/exclude` contents, and adjusts `PATH` in `.gitlab-ci.yml` to ensure `cargo` is accessible.

**Why it matters**
The MR and pipeline provide a realistic test environment for the fix, addressing the "lightly tested" caveat in the original submission. The series is narrowly scoped to GitLab’s MinGW environment and does not affect other CI platforms. Karthik’s validation increases confidence in the fix, which is critical for maintaining CI stability as Git’s Rust support matures.

---

### `reference-transaction` hook: regression fix and race condition concern
**What changed?**
Maciej Ciemborowicz submitted a three-patch series to fix a regression where the `reference-transaction` hook receives all-zero OIDs for both old and new fields when deleting branches or tags. The series restores the correct behavior (old OID followed by all-zeros) while preserving the performance gains of the original optimization. However, Karthik Nayak identified a race condition in the first patch: `refs_delete_refs()` now performs conditional deletion based on supplied old OIDs, which could cause transaction failures if the ref’s value changes between resolution and execution.

**Background**
The regression was introduced in Git 2.31 by commit `8198907795`, which optimized bulk tag deletions but inadvertently omitted the old OID for safety-checked deletions. The fix modifies `refs_delete_refs()` to accept optional old OIDs and updates callers (`git branch -d`, `git tag -d`, `git fetch --prune`, `git remote prune`) to pass the resolved values. Performance measurements confirm no regression.

**Why it matters**
The series addresses a correctness issue affecting tools that rely on the hook to track ref deletions. The race condition, however, is a serious concern: it could lead to spurious transaction failures in production workflows. The author will need to resolve this before the series can proceed, likely by reverting to unconditional deletion or implementing a retry mechanism.

---

## In brief
- **`git reflog expire` regression**: r.norouzi reported a regression in Git 2.50 where the default expiry times for reachable and unreachable reflog entries were swapped. The bug was introduced by commit `85658275702b`, which reversed the order of `.default_expire_total` and `.default_expire_unreachable` in `reflog.h`. No fix has been posted yet.
- **`fetch.shallow` config**: Harald Nordgren submitted a patch introducing `fetch.shallow` to restrict refspec-less fetches to the current branch’s upstream in shallow repositories. The feature addresses performance issues with `git fetch`/`git pull` in large repositories and is opt-in (default: `false`). The patch includes thorough test coverage and is ready for review.
- **`git commit` date warning withdrawn**: Yashwanth Sai withdrew an RFC patch proposing a warning in `git commit` for commits dated before their parents, due to AI co-authorship in the commit message violating Git’s contribution guidelines. The feature, which aimed to address usability issues with `git log --since`, was endorsed by brian m. carlson as "useful" but will not proceed in its current form.
- **`git stash` autostash fix**: D. Ben Knoble submitted a two-patch series to fix a bug in `git stash` autostash handling of staged index entries. The series replaces subprocess-based index merging with in-core logic via `merge-ort.c` and adds test coverage. The patches are under review, with no substantive feedback yet.