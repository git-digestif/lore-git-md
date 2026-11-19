# Git mailing list daily digest for 2026/10/03

## Long threads

### New gitmergeconflicts documentation guide

*Long thread: 32 replies so far from 6 participants over 10 days; 3 messages that day.*

Julia Evans's series adds a new `gitmergeconflicts` manual page explaining merge conflicts with examples, linked from `git-merge`, `git-pull`, `git-rebase`, `git-revert`, and `git-cherry-pick` instead of repeating the process in each command's documentation. Earlier reviewers shaped a new "WHAT IS A MERGE CONFLICT?" section for v2. Today D. Ben Knoble said he quite likes that section and would not have read "unstaged" as meaning removed from the index, and gave his position that `git <thing> --continue` is the right recommendation for most users even though for merge it may coincide with `git commit`. Junio C Hamano agreed Ben's reading of "unstaged" is possible but explained that "to unstage" is ambiguous and reiterated his suggestion to avoid the verb.

### Repack corner cases for cruft-less MIDXs

*Long thread: 27 replies so far from 5 participants over 4 days; 6 messages that day.*

Taylor Blau's v2 series fixes corner cases where `repack.midxMustContainCruft=false` can produce a multi-pack-index whose objects are not closed under reachability, which can make bitmap generation fail. Earlier review covered the oidset conversion, kept-pack traversal, and the `--write-midx=incremental` gap. Today Taylor Blau agreed with Jeff King that the double-negative condition in patch 5/8 is awkward, tracing it to `midx_must_contain_cruft` arguably deserving the name `midx_may_exclude_cruft`; defended calling `prepare_revision_walk()` twice on the same `rev_info` by citing existing callers; explained that the `item->util` preferred-pack marking would not survive sorting because the last marked entry wins; and confirmed that `midx_included_packs()` performs the pack selection in patch 8/8. Jeff King accepted both the double-`prepare_revision_walk()` evidence and the sorting explanation.

### RFC for git-brebase

*Long thread: 12 replies so far from 2 participants over 1 day; 13 messages that day.*

Alejandro Colomar posted an RFC for `git-brebase`, a 164-line shell script that rebases onto the first conflicting commit of a target branch using `git bisect` internally, supporting `--first-parent`, `--pre-exec`, and `--post-exec`. Nico Williams said he still preferred a `--first-conflict` option on `git rebase` itself, proposed that `git rebase` bisect to the first conflicting upstream commit by default, and suggested script improvements including a `trap` and avoiding a temp callback file. Colomar argued `--abort` should return only to the start of the current iteration so trees of branches can be rebased, patched the script to work with detached HEAD, and said he now defaults to brebase for every rebase while still using plain `git rebase` for precise operations.

## The day in brief
The merge-conflicts documentation guide and the cruft-less MIDX repack series both absorbed detailed review feedback, while a new RFC proposed a `git-brebase` tool that rebases onto the first conflicting commit. Separately, discussion of a "limbo" object-format state for empty repositories produced a concrete protocol proposal, and a filter-branch state-branch bugfix reached v3.

## Notable threads

### "Limbo" object-format state for empty repositories
Jeff King proposed letting an empty server-side repository advertise a "limbo" object format so first pushes from either sha1 or sha256 clients succeed. Today brian m. carlsson argued `object-format=limbo` would break every existing object-format-aware client and raised policy questions; Jeff King proposed a new ignorable capability ("magic-limbo") paired with a valid `object-format=` value; Junio C Hamano lamented that multiple `object-format` capability lines would have been the right design; and carlsson countered that an `alt-object-format` directive would also be needed and corrected an earlier bug recollection to an infinite loop fixed in 2.41.

### Optional sha1dc Rust backend for SHA-1
Johannes Schindelin's series adds an option to use the fast SIMD-based sha1dc Rust crate, with a runtime escape hatch to the C backend. Today Schindelin conceded "compile-time" is misleading since `sha1dc_choose()` selects the backend at runtime, and reported that the Rust and C implementations cover the same collision-detection space. brian m. carlsson objected that the crate requires Rust 1.87 while Git supports older versions, proposing that any minimum-version bump be handled in a separate series with a policy document.

### filter-branch state-branch map fix
Grant Moyer posted [PATCH v3] fixing `git filter-branch --state-branch` map initialization, which since f6d855091e interpreted object pairs in reverse, causing the state branch to be effectively ignored; the patch restores `$from_commit:$to_commit` order and adds a test that incremental rewrites use rewritten parents.

## In brief
- **history: sign rewritten commits v5**: Souma posted v5, adding Git completion coverage for the existing `-S`/`--gpg-sign` and `--no-gpg-sign` options on history subcommands.

- Colin Hinton posted v4 deferring `fetch.followRemoteHEAD` validation to the point of use in `do_fetch()`, leaving `remote.<name>.followRemoteHEAD` marked NEEDSWORK.
- Julia Evans published v2 removing required SYNOPSIS sections from section 7 man pages, with Tuomas Ahola's declarative lint-script rewrite.
- Fionn Fitzmaurice patched the Zsh completion to exclude files already present in the argument list.
- Kristoffer Haugsbakk posted v2 rephrasing the `trailer.<key-alias>.cmd` examples in the interpret-trailers documentation.
- SZEDER Gábor noted the precompiled `git-compat-util.h` changes `__FILE__` expansion in error messages, which he judged acceptable.
