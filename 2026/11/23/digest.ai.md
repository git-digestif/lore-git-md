# Git mailing list daily digest for 2026/11/23

## Long threads

### branch: let --delete-merged find squash merged branches

*Long thread (root 2026/09/29/07-30-30): 71 replies so far from 7 participants over 56 days; 1 message that day.*

This series adds `git branch --delete-integrated <pattern>` to the branch builtin, which works like `--delete-merged` but also deletes branches whose changes all landed upstream via "Squash and merge" or "Rebase and merge", naming the upstream commit in the output. Version 10 made the file merge a plain text merge that ignores merge drivers set in `.gitattributes`, fixed a branch that replaced a file or submodule with a directory never being found as landed, made `--dry-run` work with `-d` and `-D`, printed the "Deleted"/"Would delete" messages from one table of templates, and renamed the opt-out helper to `opted_out_of_delete_merged()`. Today Junio C Hamano acknowledged that v10's `--dry-run` support for `-d` and `-D` is much improved, but noted that branch creation and renaming still lack `--dry-run` support, suggesting the design mindset should shift from "dry-run is incompatible with non-delete modes" to "all operation modes ought to support it, but some do not yet", accepting these as gaps to be filled later.

### utf8: bounds-related fixes and a doc fix

*Long thread (root 2026/11/16/18-56-59): 35 replies so far from 3 participants over 8 days; 6 messages that day.*

This series fixes bounds handling in `utf8.c`: `utf8_strnwidth()` disregarded its `len` argument, and its comment incorrectly claimed it returns `strlen()` where it actually returns `len`. Version 5 keeps Junio C Hamano's functions but uses an array index in the `_counted` variant so it never computes `s + len` or touches `s[len]` or beyond, avoiding O(n^2) degradation, and passes `SIZE_MAX` in the wrapper for NUL-terminated strings. Today Junio C Hamano reviewed each v5 patch: he approved the core idea of `display_mode_esc_sequence_len_counted()` but required fixes to the sign-off chain, test assertions (`cl_assert_equal_u()` for `size_t` values), and wording; he suggested "bounded" instead of "terminated" in the `utf8_strnwidth()` comment; asked patch 3/4's message to state the externally observable symptom of an escape sequence reaching `orig + len`; and asked for a more specific title for 4/4. He said that apart from those issues the v5 series looks in very good shape and they are getting closer to the finish line. Dirk Gouders replied that he would address the items, including possibly avoiding the `_counted` variant in `strbuf_add_wrapped_text()`.

## The day in brief

Junio C Hamano moved two long-running series forward, acknowledging the improved `--dry-run` handling in v10 of the `--delete-integrated` branch work and saying the utf8 bounds-fix series is getting closer to the finish line. New review activity covered making `git history` refuse to rewrite branches checked out in other worktrees, a proposed `^/` server-root syntax for submodule URLs, and hardening bundle-uri against unresolvable relative URIs.

## Notable threads

### history: refuse to rewrite branches used by other worktrees

This patch makes `git history` refuse to rewrite local branches that contain the target commit and are checked out in another worktree, matching what `git branch -f` and `git rebase --update-refs` already do, since otherwise the other worktree shows staged changes nobody made or a later `git rebase --continue` fails. Phillip Wood reviewed it as a useful improvement and asked about a rebase running in the same worktree and about `--update-refs=head`. Junio C Hamano identified a bare-repository hole: when HEAD points at a branch checked out in another worktree, that branch is skipped, so `git history drop` moves it and leaves the worktree's index and working tree out of sync. Aniket Gupta confirmed the repro and said v2 will ask the other worktrees whether they use the current branch instead of skipping it by name, add a test for each subcommand, and fix the log message about per-subcommand behavior.

### submodule: make '^/' work like '../' but with an absolute path

This RFC proposes resolving a submodule URL starting with `^/` against the superproject's remote from the root of the server, so `../../org/lib.git` style URLs survive forking the superproject to a different depth; protocol, user, host and port stay the same. D. Ben Knoble asked about later local-path support, documentation restructuring, and the hand-rolled string parsing in `root_relative_url()`. Junio C Hamano objected to calling `^/` an "absolute path" as misleading for scp-style URLs like `git@host:group/super.git`, suggesting "paths relative to the server-root", and identified a parsing flaw in `skip_bracketed_host()` where scp-style remotes containing `@[` in the path can make `root_relative_url()` fail or return a corrupt URL.

### bundle-uri: do not die on unresolvable relative URIs

This v4 series adds `relative_url_gently()` so callers can handle resolution failures without `die()`, names the original URL when `relative_url()` fails, and changes `bundle_list_update()` in `bundle-uri.c` to report the error and stop parsing the list instead of killing the clone. Derrick Stolee said the v4 version looks good to him. Junio C Hamano pointed out that because `bundle_list_update()` places the bundle in the `.bundles` hashmap before resolving the URI, a failing entry stays with a NULL uri and can emit a redundant "error: bundle '<id>' has no uri"; Aniket Gupta confirmed the extra error appears with a server-advertised bad entry and said he will drop the bundle from the list in v5 and add a test.

### setup: fix hash algorithm detection

This v4 patch restructures `repository_format_configure()` in `setup.c` so a command-line hash wins, an existing repository keeps its hash, and a new repository falls back through environment, config, then `GIT_HASH_DEFAULT`, with re-initializing using a different hash rejected. Junio C Hamano first said everything looks good, then argued the extra `repo_fmt->hash_algo != GIT_HASH_UNKNOWN` check is pointless since `check_repo_format()` already rejects `GIT_HASH_UNKNOWN`, and reported that the "re-init does not modify hash from config" test fails because `test_config_global` does nothing useful in a subshell. brian m. carlson said he will do a reroll, noting the failure only triggers on bash.

### submodule: support linked worktrees in recursive checkout

This v2 five-patch series fixes recursive checkout when a submodule is a linked worktree of its repository, which previously crashed with a `BUG()` or left the submodule broken. Adrian Ratiu's v2 changes `validate_submodule_legacy_git_dir()` to return an error instead of `BUG()`, adds a `die_refusing_submodule_git_dir()` helper that only suggests `extensions.submodulePathConfig` when it is disabled, moves git dir setup out of `submodule_move_head()` into `setup_submodule_git_dir()`, validates the git dir after absorbing, and makes recursive checkout accept a linked worktree by validating its common dir while rejecting any git dir whose commondir file is not at `<common-dir>/worktrees/<id>`.

## In brief

- Phillip Wood reviewed Wonseok Oh's fix clearing the saved HEAD OID when cherry-picking onto an unborn branch, calling it a good immediate fix for the uninitialized variable while tracing the odd unborn-branch behavior back to 2016.
- Johannes Schindelin posted v2 of the five-patch Coverity overflow series, simplifying the `approxidate()` overflow check and switching `write_in_full()` to return int status.
- **Credential OPT_SUBCOMMAND review**: Laveesh Bansal gave a Reviewed-by on Wasswa Edson's credential OPT_SUBCOMMAND() series, then sent a detailed per-commit write-up that Junio C Hamano thanked him for.

- Christian Couder reviewed Elia Pinto's v3 `size_t` series as "almost there", suggesting small commit-message improvements.
- brian m. carlsson posted v2 of the docs patch warning that `http.extraheader` is not for credentials; Junio C Hamano suggested camelCase `http.extraHeader` and a `gitcredentials(7)` link.
- Laveesh Bansal added a Reviewed-by to Aniket Gupta's fsck patch reporting the real object count in the connectivity message.
- Okhuomon Ajayi pinged on the v2 patch removing `the_repository` from the default negotiator.
