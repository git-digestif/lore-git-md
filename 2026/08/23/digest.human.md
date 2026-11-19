# Git mailing list daily digest for 2026/08/23

## Long threads

### diff hunk provider interface rethought

*Long thread: 25 replies so far from 4 participants over 94 days; 1 message that day.*

Michael Montalbo retracted the "diff provider" abstraction introduced in RFC v7, saying it shoehorns several concepts together that should be addressed separately, and clarified that "chain order is authority" was an unclear way to describe earlier providers overriding xdiff's default. The underlying work concerns the single point where every in-process diff computes its hunks: given two blobs and diff settings, consumers such as blame, the stat formats, patch output, and `log -L` all currently load both blobs and run xdiff. Montalbo confirmed the original intent was to configure a provider for a file type via `.gitattributes` and `diff.<driver>.process`, using xdiff when no provider is configured, and stated his goals: a "cousin" interface to xdiff-interface for user-configured alternatives that compose with surrounding diff functionality while xdiff still renders content, OID-only operation for callers that need only hunk header info, and a consistent pkt-line interface for plugging in xdiff alternatives. He decided to return to the v6 line for future re-rolls and to eject the diff-hunks store patches for now. Earlier, the v7 series had restructured around a provider chain with a diff-hunks store at `$GIT_DIR/objects/info/diff-hunks`, and reviewers had questioned the terminology, cache invalidation, and a build break in `blob_line_cap()`.

## The day in brief
Michael Montalbo posted a 14-patch RFC introducing `git organize` to record and apply file placement, and separately retracted the diff provider abstraction in favor of a narrower pkt-line xdiff-interface. Delilah Ashley Wu rerolled the fix making `git config --global` read both home and XDG files, and Vladimir Sitnikov proposed a `stash.ref` knob for per-worktree stash stacks.

## Notable threads

### `git organize` RFC records and applies file placement
Michael Montalbo posted RFC PATCH 00/14 introducing `git organize`, a command that records a project's file-placement guideline in a tracked `.gitorganize` file and moves out-of-place files, repointing references and staging the result. It has `status` and `apply` subcommands, configurable `organize.labeler`/`organize.organizer` commands, and eight planned carves for odb, refs, pack, diff, revision, index, setup, and transport; the final patch moves 22 transport C sources and 21 headers into `transport/`.

### `git config --global` to read home and XDG files
Delilah Ashley Wu posted v2 of a three-patch series making `git config --global` read both `$HOME/.gitconfig` and `$XDG_CONFIG_HOME/git/config`, fixing the inconsistency where `--global` read only the former while plain `git config list` read both. The patches enforce forward slashes in XDG paths on Windows, let the config sequence optionally report an error when no files were read, and make `--global` reuse `do_git_config_sequence()`. Chris Torek asked which of the two files `git config --global --edit` edits.

### configurable ref for the stash stack
Vladimir Sitnikov proposed a `stash.ref` configuration knob (defaulting to `refs/stash`) so a worktree can use an isolated stash stack such as `refs/worktree/stash`, citing silent data movement between worktrees sharing `refs/stash`. He asked whether the value should be restricted to `refs/`, whether `git stash list` should show other stacks, and whether entries under `refs/worktree/*` are safe from gc. Kristoffer Haugsbakk agreed a configuration option makes the most sense, endorsed opting in to `refs/worktree/stash`, and answered that per gitdatamodel(7) no ref namespace is special for reachability and GC.

## In brief
- Karthik Nayak addressed a concern that moving the list lock into `struct reftable_stack` breaks nested `reftable_addition` instances, proposed tracking per-addition lock ownership, defended keeping the locked check inside `stack_uptodate()`, and reproduced Jeff King's ~25% `update-ref` speedup using `refs/tags/*`.
- Oswald Buddenhagen proposed an opt-out option for the Unicode box-drawing output in `git repo structure`, and Junio C Hamano agreed an opt-out is needed since locale-only detection leaves no way to disable it.
- Johannes Sixt submitted a gitk README patch discouraging AI contributions; Weijie Yuan asked how "Generated" differs from "Assisted" content, citing recent commits with `Assisted-by` trailers.
- Weijie Yuan and Junio C Hamano discussed AI-assistance disclosure in patches, with Junio arguing that disclosed AI use admits contamination while acknowledging Johannes Schindelin is trusted for due diligence.
- Kristoffer Haugsbakk posted v2 of the series advertising gitdatamodel(7) across documentation, correcting commit messages and asking about a ".." in gitglossary(7).
- Nikolaus Schuetz clarified that dot is the only refname character with position-specific rules, so the t1402 patch only needed to test the other forbidden characters embedded in a refname.
