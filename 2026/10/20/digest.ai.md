# Git mailing list daily digest for 2026/10/20

## The day in brief
The Git mailing list saw **156 emails** today, with key developments including a **proposed architectural shift** in the CRLF conversion bugfix, **critical platform-specific concerns** for the copy-on-write worktree feature, **performance optimizations** for the ODB alternates reloading fix, and **ongoing discussions** about AI-generated contributions, test modernization, and documentation improvements. The most consequential new developments were Torsten Bögershausen’s proposal to relocate the CRLF conversion check to `ie_match_stat()`, brian m. carlson’s warning about Windows ReFS silent corruption, and Junio C Hamano’s feedback on the object storage extension series.

---

## Notable threads

### 1. read-cache.c: Architectural debate over CRLF conversion check placement
**What changed?**
Torsten Bögershausen proposed relocating the `would_convert_to_git()` check from `ie_modified()` to `ie_match_stat()` to avoid interfering with Git’s racy Git logic. This would simplify the code by removing the conditional shortcut in `ie_modified()` but introduce a minor inefficiency: `would_convert_to_git()` would be called twice (once in `ie_match_stat()` and once in `ce_modified_check_fs()`).

**Why it matters**
The proposal addresses a long-standing architectural debate in the CRLF conversion bugfix series. Curtis Allen Smith’s v5 patch placed the check in `ie_modified()`, but Torsten’s alternative would affect a broader set of commands (e.g., `git diff-files`, `git diff-index`) and reintroduce performance overhead. Junio C Hamano’s recent review emphasized preserving the lstat() optimization, suggesting neither approach may fully meet the project’s standards.

**Key details**
- **Files**: `read-cache.c`, `t/t0020-crlf.sh`
- **Commands affected**: `git status`, `git rebase`, `git pull --rebase`, `git diff-files`, `git diff-index`
- **Performance trade-off**: 12 microseconds per modified 2.6 KB file (realistic) vs. 138ms for 10,000 files (extreme)
- **Racy Git concern**: Torsten’s proposal avoids interfering with Git’s racy Git logic for Windows symlinks
- **Unresolved**: Johannes Sixt’s objection that the fix should target `git add` rather than index refresh operations

---

### 2. worktree: Platform-specific correctness risks for copy-on-write feature
**What changed?**
brian m. carlson raised **critical correctness concerns** about the Windows ReFS reflink implementation (`FSCTL_DUPLICATE_EXTENTS_TO_FILE`), citing a long-standing Git LFS bug (issue #6312) that can cause silent corruption when cloning extents. He advocated disabling Windows reflink entirely until Microsoft fixes the underlying bug. For macOS, he noted the CI testability of the clonefile() path remains unvalidated.

**Why it matters**
The v2 worktree improvements series introduced cross-platform copy-on-write support, but these concerns threaten its viability on Windows and macOS. The Windows ReFS corruption risk is particularly severe, as it could lead to data loss without warning. The macOS CI issue, while less critical, could leave the APFS reflink path untested in automated pipelines.

**Key details**
- **Files**: `copy.c`, `copy.h`, `builtin/worktree.c`
- **Windows risk**: Silent corruption with `FSCTL_DUPLICATE_EXTENTS_TO_FILE` (Git LFS #6312)
- **macOS risk**: CI environment may not support `cp --reflink=always`, leaving clonefile() path untested
- **Author response**: Jason Newton has not yet addressed these concerns
- **Impact**: May require disabling Windows reflink or adding platform-specific safeguards

---

### 3. odb: Performance optimizations for alternates reloading fix
**What changed?**
Patrick Steinhardt’s four-patch series to restore dynamic reloading of alternate object databases received **performance optimizations** in patches 3 and 4. Patch 3 reordered logic in `odb_files_dir_is_usable()` to check the hashmap before hitting the filesystem, reducing lookup time for 20k missing objects from 309ms to 263ms. Patch 4 used `struct stat_validity` to track the alternates file’s mtime and inode, further reducing lookup time to 197ms (nearly matching the pre-regression baseline of 180ms).

**Why it matters**
The regression (introduced in March 2026) caused multi-minute delays in shallow fetches from large alternates (e.g., 7M objects). The optimizations make the fix production-ready by mitigating its performance overhead. Junio C Hamano’s review identified a minor edge case in patch 4 (updating `stat_validity` even on read failures), but the series is otherwise uncontroversial.

**Key details**
- **Files**: `odb/source-files.c`, `odb/source-files.h`, `t/t5547-push-quarantine.sh`
- **Performance**: 309ms → 263ms (patch 3) → 197ms (patch 4)
- **Edge case**: `stat_validity` updated even if alternates file read fails
- **Test coverage**: New test verifies concurrent alternates updates

---

### 4. extensions.objectStorage: Design feedback and open questions
**What changed?**
Karthik Nayak’s review of Patrick Steinhardt’s object storage extension series raised **four design-level concerns**:
1. **Path normalization**: Should the `files` backend use `strbuf_realpath()` for consistency with the ref storage extension?
2. **Naming subtlety**: The distinction between `object.format` (hash algorithm) and `objects.storageFormat` (storage backend) in `git repo` output may confuse users.
3. **Encapsulation**: The hard-coded default (`files`) is exposed in inspection tools, creating a maintainability risk.
4. **Test coverage**: Should invalid path payloads for the `files` backend be tested?

**Why it matters**
The series introduces a pluggable object database backend mechanism, a major architectural change. The feedback highlights potential inconsistencies and maintainability risks that could affect long-term usability. Patrick acknowledged the concerns and committed to exploring fixes (e.g., initializing the `object_storage_format` field), but the naming issue was resolved as a pragmatic compromise.

**Key details**
- **Files**: `odb/source-files.c`, `remote.c`, `builtin/repo.c`
- **New config**: `extensions.objectStorage`, `init.defaultObjectStorageFormat`
- **CLI options**: `--object-storage-format=`, `--show-object-storage-format`
- **Unresolved**: Path normalization, default encapsulation, test coverage for invalid paths

---

### 5. AI-generated contributions: Policy discussion
**What changed?**
Junio C Hamano and Patrick Steinhardt engaged in a **meta-discussion** about AI-generated contributions, sparked by a patch with an overly verbose commit message. Junio agreed contributors must demonstrate understanding when using AI tools but suggested AI can be useful for polishing (e.g., grammar, typos) if guided by domain knowledge. Patrick critiqued AI-generated commit messages as "exhaustive but hollow," advocating for contributor-written messages to maintain quality standards.

**Why it matters**
The discussion reflects broader concerns about maintaining code quality and contributor accountability as AI-assisted contributions increase. The project is still developing a formal AI policy, with Emily Shaffer identified as the likely organizer of a contributors’ summit session.

**Key details**
- **Context**: Patch from Matthew Suozzo fixing a reftable buffer overflow
- **Junio’s stance**: AI can polish but not generate content without domain knowledge
- **Patrick’s stance**: Contributor-written messages are essential for quality
- **Next steps**: Emily Shaffer to organize a contributors’ summit session

---

## In brief

### Bugfixes and regressions
- **ZIP archive**: René Scharfe’s v2 patch to omit the Extended Timestamp (UT) field for out-of-range values is **ready for integration** after incorporating Matthew E. Luallen’s test infrastructure fix (`TIME_T_IS_64BIT`).
- **`git init`**: Matt Hunter’s patch to restore absolute path output in the "Initialized empty Git repository" message is **ready for merging** after addressing Junio’s feedback. The regression was introduced in Git 2.56.
- **`git fsck`**: Aniket Gupta’s patch to correct the misleading object count in `--verbose` mode is **under review**, with no objections raised.
- **`git imap-send`**: Harald Nordgren’s v2 patch to fix non-canonical email header order now uses `istarts_with()` for RFC 5322 compliance.

### Refactoring and cleanup
- **`the_repository` removal**: Okhuomon Ajayi’s patch to eliminate `the_repository` from `merge-blobs.c` is **ready for integration** after resolving a parameter naming debate (`r` vs. `repo`).
- **`prune-packed`**: Shashank Srikumar’s patch to move `pack.packsizelimit` into `repo_config_values` is **under review**, with Kristoffer Haugsbakk commending the commit message’s clarity.
- **`git stash`**: Phillip Wood’s seven-patch refactoring series to eliminate redundant index operations is **ready for integration**, with Junio praising the most interesting patch (replacing `reset_tree()` with `create_index_from_tree()`).

### Documentation
- **`gittutorial`**: Julia Evans’ v2 series to replace the outdated tutorial is **ready for merging** after addressing AsciiDoc formatting warnings. The series clarifies the tutorial’s living-draft nature and improves SSH setup instructions.
- **`git rebase`**: Omri Sarig’s v3 patch to fix stale mentions of `--preserve-merges` and `--allow-empty-message` is **queued for `maint`** (Git 2.56.0) after Junio’s editorial refinements.
- **`MyFirstContribution`**: Junio’s v2 series to reorder the tutorial and emphasize review discussion is **under review**, with Patrick Steinhardt suggesting splitting structural changes from wording updates.

### Test modernization
- **`t/t0410-partial-clone.sh`**: Dorine Tipo’s v2 patch to replace brittle shell pipelines with `git rev-parse` is **under review**, with no objections raised.
- **`t/t5301-sliding-window.sh`**: Roger Koranteng’s patch to replace `test -f` with `test_path_is_file` is **under review**, with no feedback yet.

### Outreachy and mentorship
- **Promisor remote fetch ordering**: Wasswa Edson inquired about the project’s scope, seeking clarification on whether it involves adding user-configurable control over fetch order.
- **Argument parsing**: Tanga Joris proposed a microproject to convert a translatable string in `builtin/checkout.c` from `_()` to `Q_()` as part of the "Improve how command arguments and options are scanned and parsed" project.

### CI and build system
- **GitLab CI**: Patrick Steinhardt’s patch to fix shallow clone failures in "check-whitespace" and "check-style" jobs is **under review**, with Junio questioning the jobs’ overall value.
- **GitHub Actions**: Junio’s v4 patch to split macOS runners between `macos-15` and `macos-26` was **retracted due to a mechanical error**, with a v5 expected.

### Miscellaneous
- **`git bisect`**: Steven Rostedt reported that `ktest.pl` (Linux kernel tooling) relies on parsing `git bisect` output, which was broken by a recent formatting change. Theodore Ts'o proposed adding a `--format=json` option for structured output.
- **Hook configuration**: Thomas Bachem’s patch to make `hook.<event>.enabled` apply to all hook types (not just config-defined hooks) is **under review**, with Adrian Ratiu confirming the current behavior was intentional.