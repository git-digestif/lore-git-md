# Git mailing list daily digest for 2026/08/15

## Long threads

### GSoC series adding path keys to `git repo info`

*Long thread (root 2026/07/16/01-21-31): 20 replies so far from 4 participants over 31 days; 1 message that day.*

K Jayatheerth's GSoC series extends `git repo info` with keys that output the paths of repository components: `path.toplevel` for the repository tree, `path.superproject-root` for the superproject tree from submodules, `path.hooks`, `path.index`, `path.grafts`, and `path.git-prefix`, each supporting absolute and relative suffixes for formatting, along with documentation and tests. The latest revision renamed `superproject-working-tree` to `superproject-root`, shifted the prefix member from `startup_info` into `struct repository`, dropped the `path.objects` key, and removed unnecessary `strbuf_addstr(buf, "")` calls. Today Lucas Seiki Oshiro asked why the test for `path.toplevel` absolute and relative suffix formatting does not use the `test_repo_info_path` helper.

## The day in brief

Tian Yuchen posted v6 of the patch moving `fetch_if_missing` into `struct repository`, which Junio C Hamano reviewed favorably. Johannes Sixt reported that the twelve-patch Git for Windows upstreaming series builds cleanly with no negative effects. Yoichi NAKAYAMA submitted a fix for worktree repair's handling of relative paths in `.git` files.

## Notable threads

### Moving `fetch_if_missing` into `struct repository`

This libification patch moves the global `fetch_if_missing` variable, which controls whether a missing-object check prompts a lazy fetch from a promisor remote, into `struct repository` with a default of 1 to preserve behavior. Tian Yuchen posted v6, in which `option_parse_missing_action()` in `builtin/pack-objects.c` now receives a `repo` through the option's `value` field, dropping the earlier commit that gave `fetch_if_missing` call sites access to `repo`. Junio C Hamano reviewed the iteration, asked as a curiosity whether running `git index-pack` outside a repository is legitimate or a user error, and said that otherwise it looks good to him.

### `format-rev` learns `--abbrev`, `--color`, and `--date`

Kristoffer Haugsbakk's four-patch series adds three formatting options to `git format-rev` along with preparatory refactoring and documentation changes, factoring the `--date` alternatives out of `rev-list-options.adoc` into a shared include. Today Junio C Hamano pointed out that in the `date_cb` callback of patch 4/4, the `BUG_ON_OPT_NEG(unset)` assertion sits after `parse_date_format(arg, &data->date_mode)`, which dereferences `option->value`; if `unset` were 1, `option->value` would be NULL and the code would segfault before the assertion could trigger, so he suggested moving the assertion before the dereference.

### Upstreaming more Git for Windows patches

Johannes Schindelin's twelve-patch v2 series upstreams assorted mingw and Windows build fixes from Git for Windows, covering the Python parts in the build, `CC` hard-coding, `-D_USE_32BIT_TIME_T`, `--large-address-aware`, `MINGW_PREFIX` handling in meson, MSYS2 metadata, and related build-system adjustments. Today Johannes Sixt observed that in patch 08/12 `MINGW_PREFIX` is only used to set `prefix`, so the CMake and meson hunks introducing `ENSURE_MSYSTEM_IS_SET` and `MINGW_PREFIX` as compile definitions do not belong in that patch yet, since only patch 12/12 uses the variable to drive C code. Sixt also reported that he has been building his own Windows version with a subset of the series for some time and that building with all twelve patches showed no negative effects after a quick check, with CI passing.

### Worktree repair relative-path detection in `.git` files

Yoichi NAKAYAMA's patch fixes `git worktree repair` for worktrees whose `.git` file contains a relative path: since `read_gitfile_gently()` always returns an absolute path, the relative-to-absolute conversion in `repair_gitfile()` and `repair_worktree_at_path()` was dead code. Today he submitted the patch, which adds a `bool *absolute` output parameter to `read_gitfile_gently()` and removes the dead conversion code, touching `builtin/init-db.c`, `setup.c`, `setup.h`, `worktree.c`, and `t/t2406-worktree-repair.sh`.

### Warning on valueless `submodule.active` config

tilak-raaz's patch addresses a hard error from the config parser when `submodule.active` is given without a value, which aborts commands. It swaps `repo_config_get_string_multi()` for `repo_config_get_value_multi()` to parse valueless true safely and emits a warning via `config_error_nonbool()` instead of crashing, resolving a NEEDSWORK comment in `submodule.c`. Today tilak-raaz posted v4, correcting the v3 commit message that a botched `--amend` had left in the past tense. He explained that returning an error code was investigated but rejected because `is_tree_submodule_active()` is evaluated as a boolean predicate by callers and `-1` is truthy in C, so the implementation keeps the `continue` behavior: malformed valueless entries are skipped after the warning while valid entries are processed.

## In brief

- In the RFC on using filesystem cloning (copy-on-write or block cloning) for `git worktree add`, brian m. carlson cautioned against ReFS support, citing serious implementation defects where unflushed copies can be corrupt, while Peter Morris countered that the Windows CopyFile API opens the source with shared-read deny-write access and Linux FICLONE is atomic, concluding support would be limited to Windows and Linux.
- On the patch improving the `git worktree add` error for an ambiguous remote branch name, Yoichi Nakayama agreed that modifying `unique_tracking_name()` to optionally report matching remotes is good, and noted that a fully qualified name like `origin/bar-topic` is resolved before that function is called.
- On slow bash completion in large repositories, D. Ben Knoble suggested enabling `feature.manyFiles`; Matthew Hughes reported that his repo is configured via `git-scalar(1)` with `feature.manyFiles=false`, that a fresh repo with it enabled showed no significant change, and that he was not sure whether `git ls-files --others --directory` can use fsmonitor or the untracked cache.
- Jeff King affirmed that Colin Hinton's patch dropping the unused `name` parameter from the chdir-notify APIs "summarizes the situation nicely."
