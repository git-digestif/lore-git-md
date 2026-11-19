# Git mailing list daily digest for 2026/08/15

## Long threads

### `git repo info` path keys from the GSoC series

*Long thread: 20 replies so far from 4 participants over 31 days; 1 message that day.*

K Jayatheerth's GSoC series adds new keys to `git repo info` that output the paths of repository components: `path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, and `path.git-prefix`, each supporting absolute and relative suffixes for formatting, with documentation and tests. Earlier revisions renamed `superproject-working-tree` to `superproject-root`, dropped `path.objects` until pluggable ODB backends arrive, removed unnecessary `strbuf_addstr(buf, "")` calls, and added relative-path test cases. Today Lucas Seiki Oshiro asked why the test for `path.toplevel` absolute and relative suffix formatting does not use the `test_repo_info_path` helper.

## The day in brief

Tian Yuchen posted v6 of the patch moving `fetch_if_missing` into `struct repository`, which Junio C Hamano reviewed favorably. Junio also caught an assertion-order bug in the new `git format-rev` date callback that could segfault before the assertion fires. On the GSoC `git repo info` path-keys series, Lucas Seiki Oshiro questioned the test structure for `path.toplevel`.

## Notable threads

### Moving `fetch_if_missing` into `struct repository`

Tian Yuchen posted v6 of the libification patch that moves the global `fetch_if_missing` variable, which controls whether a missing-object check prompts a lazy fetch from a promisor remote, into `struct repository`, defaulting to 1 to preserve behavior. In v6, `option_parse_missing_action()` in `builtin/pack-objects.c` receives the `repo` pointer through the option's `value` field, so the earlier commit that gave `fetch_if_missing` call sites access to `repo` is dropped. Junio C Hamano reviewed the iteration, asked as a curiosity whether running `git index-pack` outside a repository is legitimate or a user error, and said that otherwise it looks good to him.

### `format-rev` learns `--abbrev`, `--color`, and `--date`

Kristoffer Haugsbakk's four-patch series adds `--abbrev`, `--color`, and `--date` options to `git format-rev`, with preparatory refactoring of option variables into a struct and documentation factoring. Today Junio C Hamano pointed out that in the `date_cb` callback of patch 4/4, the `BUG_ON_OPT_NEG(unset)` assertion is placed after `parse_date_format(arg, &data->date_mode)`, which dereferences `option->value`; if `unset` were 1, `option->value` would be NULL and the code would segfault before the assertion could trigger, so he suggested moving the assertion before the dereference.

### Upstreaming more Git for Windows mingw patches

Johannes Schindelin's twelve-patch v2 series upstreams Git for Windows build fixes touching `config.mak.uname`, `meson.build`, and mingw compilation flags, including Python inclusion in the build, MSYS2 metadata handling, and `ETC_*` definitions. Today Johannes Sixt observed that in patch 08/12 `MINGW_PREFIX` is only used to set `prefix`, so the CMake and meson hunks introducing `ENSURE_MSYSTEM_IS_SET` and `MINGW_PREFIX` as compile definitions do not belong in that patch yet, since only patch 12/12 uses the variable to drive C code. Sixt also reported that he has been building his own Windows version with a subset of the series and that building with all twelve patches showed no negative effects after a quick check, with CI passing.

## In brief

- Yoichi NAKAYAMA submitted a patch fixing `worktree repair` to handle relative paths in `.git` files correctly, adding a `bool *absolute` output parameter to `read_gitfile_gently()` and removing dead relative-to-absolute conversion code in `repair_gitfile()` and `repair_worktree_at_path()`, since `read_gitfile_gently()` always returns an absolute path.
- tilak-raaz posted v4 of a patch making a valueless `submodule.active` config emit a warning via `config_error_nonbool()` instead of a hard error, swapping `repo_config_get_string_multi()` for `repo_config_get_value_multi()`; returning an error code was rejected because `-1` is truthy in C and callers evaluate `is_tree_submodule_active()` as a boolean predicate.
- In the RFC on using filesystem cloning for `git worktree add`, brian m. carlson cautioned against ReFS support citing corruption of unflushed copies, while Peter Morris countered that Windows `CopyFile` and Linux `FICLONE` avoid the issue and said the approach is viable but beyond his own expertise to implement.
- Yoichi Nakayama agreed that modifying `unique_tracking_name()` to optionally report which remotes have a branch with the given name is a good approach for the improved ambiguous-remote-branch message in `git worktree add`.
- On slow bash completion in large repositories, D. Ben Knoble suggested enabling `feature.manyFiles`; Matthew Hughes noted `git-scalar(1)` sets it to false and that enabling it in a fresh repository showed no significant performance change.
- Jeff King affirmed that Colin Hinton's patch dropping the unused `name` parameter from the `chdir-notify` register/unregister APIs and callback signature "summarizes the situation nicely."
