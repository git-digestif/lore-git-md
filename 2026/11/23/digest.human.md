# Git mailing list daily digest for 2026/11/23

## Long threads

### branch: let --delete-merged find squash merged branches

*Long thread: 71 replies so far from 7 participants over 56 days; 1 message that day.*

This series adds `git branch --delete-integrated <pattern>` to the branch builtin, which works like `--delete-merged` but also deletes branches merged through "Squash and merge" or "Rebase and merge", naming the upstream commit that landed them. Earlier revisions fixed a crash when a submodule path was replaced by different regular files on both sides, made the file merge a plain text merge that ignores `.gitattributes` drivers, and added `--dry-run` support for `-d` and `-D`. Today Junio C Hamano acknowledged that v10's `--dry-run` support for `-d` and `-D` is much improved, while noting that branch creation and renaming still lack `--dry-run` support and suggesting the design should eventually treat dry-run as something all operation modes support.

### utf8: bounds-related fixes and a doc fix

*Long thread: 35 replies so far from 3 participants over 8 days; 6 messages that day.*

Dirk Gouders's series fixes `utf8_strnwidth()` in `utf8.c`, which disregarded its `len` argument, and corrects its comment; v5 keeps Junio C Hamano's functions while using an array index in the `_counted` variant to avoid computing `s + len` and O(n^2) degradation, passing `SIZE_MAX` in the wrapper for NUL-terminated strings. Today Junio C Hamano reviewed each v5 patch: he approved the core idea of `display_mode_esc_sequence_len_counted()` but required a chronological sign-off chain, removal of an unneeded include, `cl_assert_equal_u()` for size_t values, and wording changes, asked patch 3/4's message to state the externally observable symptom of an escape sequence reaching past `len`, and said that apart from those issues the series looks in very good shape. Dirk Gouders replied that he would fix the sign-off order, shorten the long test lines, and test whether the `_counted` variant can be avoided in `strbuf_add_wrapped_text()`.

## The day in brief

Junio C Hamano's reviews moved two long-running series closer to completion: the utf8 bounds fixes look in very good shape apart from listed nits, and the `--delete-integrated` dry-run handling was acknowledged as much improved. Aniket Gupta's `git history` worktree-safety patch drew a concrete bare-repository hole that he plans to fix in v2, and his bundle-uri series earned a look-good from Derrick Stolee plus one redundant-error nit.

## Notable threads

### history: refuse to rewrite branches used by other worktrees

This patch makes `git history` refuse to rewrite local branches that another worktree has checked out, matching what `git branch -f` and `git rebase --update-refs` already do, since otherwise the other worktree is left showing staged changes nobody made. Today Phillip Wood reviewed it as a useful improvement and asked about same-worktree rebase interactions and documentation; Junio C Hamano identified a bare-repository hole where HEAD points at a branch checked out in another worktree; Aniket Gupta confirmed the repro and said v2 will ask the other worktrees whether they use the current branch instead of skipping it by name, with tests for each subcommand.

### submodule: make '^/' work like '../' but with an absolute path

This RFC resolves a submodule URL starting with `^/` against the superproject's remote from the server root, so `../../org/lib.git` style URLs survive forking to a different depth. Today D. Ben Knoble asked about documentation structure, local-path support, and the hand-rolled parsing in `root_relative_url()`; Junio C Hamano objected to calling `^/` an "absolute path" for scp-style URLs and identified a parsing flaw in `skip_bracketed_host()` where `@[` in an scp-style path can advance past the first colon and corrupt the resolved URL.

### bundle-uri: do not die on unresolvable relative URIs

Aniket Gupta's v4 adds `relative_url_gently()` so a bundle list entry with an unresolvable relative URI is reported and skipped instead of killing the whole clone, naming the original URL in the error. Today Derrick Stolee said the version looks good to him, while Junio C Hamano pointed out that a failing entry stays in the `.bundles` hashmap with a NULL uri and can emit a redundant second error; Aniket Gupta confirmed and said v5 will drop the bundle from the list when its URI cannot be resolved.

## In brief

- brian m. carlsson posted v4 of "setup: fix hash algorithm detection", restructuring `repository_format_configure()` so a command-line hash wins and re-initializing with a different hash is rejected; Junio C Hamano said the changes look good, then flagged a pointless `hash_algo` check and a test-script failure caused by `test_config_global` in a subshell, which brian m. carlsson will address in a reroll.
- Adrian Ratiu posted v2 of the five-patch series supporting linked worktrees in recursive submodule checkout, replacing a `BUG()` with an error, centralizing the `extensions.submodulePathConfig` advice, and validating the git dir after absorbing.
- Phillip Wood reviewed Wonseok Oh's fix clearing the saved HEAD OID when cherry-picking onto an unborn branch as a good immediate fix, tracing the uninitialized-variable use to a 2016 commit and suggesting future work on abort semantics.
- Johannes Schindelin posted v2 of the five-patch Coverity overflow series, including rejecting quantities above `INT_MAX` in `approxidate` relative-time multiplication.
- Laveesh Bansal gave a `Reviewed-by` on Wasswa Edson's `OPT_SUBCOMMAND()` conversion of `git credential` and posted a detailed review write-up that Junio C Hamano called really helpful.
