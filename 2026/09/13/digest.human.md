# Git mailing list daily digest for 2026/09/13

## Long threads

### Advice scoping for the defaultBranchName hint

*Long thread: 20 replies so far from 4 participants over 5 days; 1 message that day.*

Vsevolod Myalitsin's three-patch series in `advice.c` makes the `defaultBranchName` advice recommend disabling itself in the global configuration, because the current hint (`git config set advice.defaultBranchName false`) writes to the local repository, which is useless: the advice appears while initializing a repository, and the initialization scenario does not repeat for that repository, so the advice reappears on the next `git init`. Earlier reviews shaped the v4 series into passing the whole `advice_setting` to `vadvise()`, adding a scope hint, and setting the global scope for this advice. Today Junio C Hamano agreed that adding "--global" to the advice hint is acceptable since users can still cut-and-paste and tweak it, but clarified that the original intent was for users to decide the scope themselves, and suggested it may not be necessary to mark each advice individually for scope since the message could simply always say "--global".

## The day in brief

Junio C Hamano judged the Windows CI Rust series ready and decided to merge it in `next` and fast-track it to `master` immediately. Two new bugfix series landed on the list: one validating `git worktree repair` targets before modifying cross-references, and one fixing ODB transaction commits that mix loose objects with large blobs under batch fsync.

## Notable threads

### Using Rust in the Windows CI jobs

Johannes Schindelin's series lets the Windows CI jobs build with Rust, which Git requires by default since v2.55.0; those jobs had opted out because the minimal Git for Windows SDK they download lacks Rust, but the GitHub runners come with Rust preinstalled, configured for the Visual C toolchain, so the series picks a GCC-compatible Cargo target under MSYS2/MinGW. Today Schindelin confirmed that `CARGO_BUILD_TARGET` works for this, noting the Makefile does not export it automatically so it must be exported explicitly; he posted v3 switching from `CARGO_TARGET`/`--target` to `CARGO_BUILD_TARGET`, then v4 whose only change is removing the now-incorrect commit-message paragraph about `--target`. Junio C Hamano judged v4 ready and decided to merge it in `next` and fast-track it to `master` immediately, reasoning that Windows has an unusually high ratio of binary users relative to source builders.

### Validating `git worktree repair` targets

Yoichi NAKAYAMA submitted a two-patch series fixing `git worktree repair` in `worktree.c` and `t/t2406-worktree-repair.sh`: the command did not sufficiently validate the cross-references between a linked working tree and its administrative data, so it could modify the wrong `.git` file or gitdir. The first patch refactors `repair_worktree_at_path()` to read the `.git` file once via `read_gitfile_raw()` and extract the worktree ID with a new `get_worktree_id()`; the second verifies that the recorded worktree ID matches the worktree being repaired and stops when the gitdir path cannot be inferred, preventing corruption when worktree directories are swapped or a linked worktree is copied.

### Flushing the transaction packfile before migrating objects

Justin Tobler posted a two-patch series fixing a commit failure in `object-file.c` when an ODB transaction contains both a loose object and a "large" blob with `core.fsyncMethod` set to `batch`: the transaction's temporary directory was migrated to the main ODB before the packfile was flushed, producing a rename error. The first patch lifts the `odb_reprepare()` call out of `flush_packfile_transaction()` and requires callers to invoke it explicitly; the second moves the packfile flush ahead of the object migration in `odb_transaction_files_commit()` and adds a `t/t1050-large.sh` test verifying no objects remain in temporary locations.

## In brief

- Johannes Schindelin changed `contrib/buildsystems/CMakeLists.txt` so the CMake Windows x64 build defines `ENSURE_MSYSTEM_IS_SET="UCRT64"` and `MINGW_PREFIX="ucrt64"`, accommodating Git for Windows' migration to UCRT64.
- Johannes Schindelin fixed two Perl-related Windows test failures: `t9700` now checks `$ENV{MSYSTEM}` because MSYS2 Perl reports `$^O` as `cygwin`, and `t9129` skips its UTF-8 cases on `MINGW`.
- Jean-Noël AVILA argued against extending the documentation lint regex to commands, citing false positives in definition lists, and Todd Zullinger said he would drop the lint patch and wait before re-rolling the pack-refs and refs doc fixes.
- René Scharfe noted that GCC accepts `if (!E);` while Clang warns, and agreed the polite course on the cocci `if (!E) free(E)` conversion is to leave the flawed code in place.
- Orgad Shaneh pinged Junio C Hamano about tag content verification in the `fetch --prune-tags` tests; Junio replied he needs to swap the old discussion back into context and would not respond today.
