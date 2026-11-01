# Git mailing list daily digest for 2026/10/31

## The day in brief

Git's HTTP transport gained resilience with a four-patch series adding exponential backoff, configurable retry status codes, and POST request retries. A critical bugfix series addressed 64-to-32-bit truncations in object handling on Windows, while `git stash` performance improved by eliminating redundant index operations. The `@{push}` revision suffix now aligns with `git push` behavior in triangular workflows, and `git describe` fixed commit distance calculations for skewed timestamps.

## Notable threads

### parse-options: new sub-API for early argument scanning
[2026/09/02/16-10-41] Christian Couder's v3 series for `early_scan_options()` received a **tested review** from Cynthia Uche. The reviewer confirmed the `EARLY_SCAN_STOP_AT_NON_OPTION` flag is critical to prevent misinterpreting non-option arguments (e.g., `extra` in `--depth 5 extra --allow-unsafe-features`). Cynthia suggested clarifying this in the comment above the `early_scan_options()` call in `fast-import.c`, noting the flag's purpose is to prevent unsafe features from being wrongly allowed before `parse_options()` can reject invalid syntax.

### sequencer: refuse to revert/cherry-pick a shallow boundary commit
[2026/10/21/10-09-41] Nakul V S's v2 patch now consolidates shallow-boundary detection into a shared helper function (`sequencer_refuse_shallow_boundary()`) and omits the rarely-used `--allow-shallow-boundary` option from the synopsis for clarity. The author acknowledged the manual-todo-list edge case (where a boundary commit's hash is manually added) as an acceptable trade-off, noting the safeguard fails early to avoid leaving behind partial state.

### Fix 64-to-32 truncations on LLP64 platforms (Windows)
[2026/10/21/10-16-36] Elia Pinto's v3 series addressed the last procedural feedback from Patrick Steinhardt. The revised test in `t1007-hash-object.sh` now drops the redundant size comparison and uses `git cat-file -e` to verify object existence, eliminating race-condition risks. The fourth patch's commit message was simplified to avoid overstating the overflow risk in `convert.c` counters. The series is now technically complete and awaits integration.

### completion: remove NO_PARSEOPT for commands using parse-options
[2026/10/24/05-24-49] Tanga Joris agreed to add a comment in `parse-options.h` explaining why `show_usage_with_options_if_asked()` handles completion-helper arguments before command initialization. The change clarifies the centralized refactoring in `parse-options.c` and addresses Kaartic Sivaraam's feedback on maintainability.

### redesigning signal handling in threaded Git code
[2026/10/29/14-15-21] Phillip Wood tentatively suggested using an atomic flag in the signal handler to serialize unwinding of the `sigchain` stack for SIGPIPE. The proposal follows Oswald Buddenhagen's clarification that SIGPIPE (and four other signals) are thread-directed, complicating the original plan to block all signals during thread creation. The thread remains in the design-proposal stage, awaiting feedback from Jeff King and Junio.

### Bug report: include directives in .git/config cause silent failures
[2026/10/31/02-56-44] Chris Torek's proposal to ignore include directives in Git-managed config files faced pushback from brian m. carlson, who cited a real-world workflow relying on `[include]` in `.git/config`. The discussion shifted from a robustness improvement to a compatibility concern, with no clear path forward yet.

### git describe: fix commit distance calculation for skewed timestamps
[2026/10/31/06-55-03] Linus Torvalds submitted a patch correcting `git describe`'s commit distance calculation when commit dates are non-monotonic. The fix prioritizes commit generation numbers (from the commit graph) over raw commit dates, ensuring the walk follows true topological order. Tuomas Ahola offered a rewritten test setup using `test_tick` manipulation to achieve the same commit-date skew, improving test readability.

### Fix truncation issues with large objects on 32-bit platforms
[2026/10/31/11-39-34] Elia Pinto's four-patch series addressed truncation issues in the pack-receiving pipeline (`unpack-objects`, `index-pack`, `git fsck`). The patches convert critical size parameters from `unsigned long` to `size_t`, fixing crashes, false SHA-1 collision reports, and partial object checks on 32-bit platforms. The series is self-contained and merges cleanly with other in-flight work.

### http: exponential backoff, configurable retry codes, POST retries
[2026/10/31/15-57-00] Ravi Mistry's four-patch series improved Git's HTTP transport resilience. Key changes include:
- Fixing an off-by-one retry count bug and a race in Retry-After header parsing.
- Introducing exponential backoff (1s, 2s, 4s, ...) when Retry-After headers are absent.
- Adding the `http.retryStatusCodes` config option to retry transient 5xx errors (opt-in).
- Extending retries to HTTP POST requests, which handle data transfers during fetch/clone and partial clone operations.

The series targets real-world pain points for users operating at scale and includes thorough test coverage.

### git stash: eliminate duplicate work in stash creation
[2026/10/31/20-01-07] Jasmeet Bhatia's patch improved `git stash` performance by reusing the already-prepared index state and untracked file list. The change adds an optional `prepared_untracked_files` parameter to `do_create_stash()`, skipping redundant `repo_read_index_preload()` and `check_changes()` calls. Performance measurements showed a 16-18% improvement for `git stash -p` and 6% for `git stash push` on the Linux kernel repository.

## In brief
- **[2026/10/31/08-25-49]** Srujith A Poojary's v3 patch fixed commit abbreviation in submodule diff headers by using the submodule's object database for uniqueness checks.
- **[2026/10/31/10-06-42]** Thomas Bachem's patch made valueless config settings (`merge.renames`, `diff.renames`, `merge.directoryRenames`) default to `true` in merge-ort.
- **[2026/10/31/10-45-26]** Okhuomon Ajayi removed the last uses of `the_repository` in `fsmonitor.c` as part of the ongoing `the_repository` removal effort.
- **[2026/10/31/11-09-56]** Elia Pinto clarified that the 64-to-32 truncation series targets only in-memory buffers, with a separate patch for `off_t` conversions in streaming reads.
- **[2026/10/31/11-14-50]** Okhuomon Ajayi removed the last uses of `the_repository` in `fsmonitor.c`.
- **[2026/10/31/18-04-10]** Elia Pinto's v2 patch fixed a data corruption bug in `git pack-objects` by using `off_t` for compressed data length on 32-bit platforms.