# Git mailing list daily digest for 2026/10/03

## Long threads

### New gitmergeconflicts(7) manual page linked from merge commands

*Long thread (root 2026/09/24/14-44-15): 32 replies so far from 6 participants over 10 days; 3 messages that day.*

This documentation series adds a new `Documentation/gitmergeconflicts.adoc` manual page explaining merge-conflict resolution with examples, linked from `git-merge`, `git-pull`, `git-rebase`, `git-revert`, and `git-cherry-pick` instead of repeating the process in each page. Earlier rounds refined a new "WHAT IS A MERGE CONFLICT?" section for v2, with Patrick Steinhardt, Junio C Hamano, and Julia Evans trading wording suggestions. Today D. Ben Knoble said he quite likes the new section and would not have read "unstaged" as deleted from the index, only changed and not `git add`-ed; he also stated that `git <thing> --continue` is the right recommendation for most users, preferring the uniform interface even where `git commit` and `git merge --continue` may coincide. Junio C Hamano agreed Ben's reading is possible, explained that "to unstage" is ambiguous (`git reset HEAD -- path` or `git rm --cached -- path`, neither matching what a conflicted merge records in the index), and reiterated his suggestion to avoid the verb.

### Repack corner cases for cruft-less MIDXs

*Long thread (root 2026/09/30/01-28-34): 27 replies so far from 5 participants over 4 days; 6 messages that day.*

This eight-patch v2 series by Taylor Blau fixes corner cases where `repack.midxMustContainCruft=false` can produce a multi-pack index not closed under reachability, which can make bitmap generation fail even though all reachable objects remain available; it adds tree and tag roots from included packs, retains cruft for incremental repacks, and includes required packs in ordinary and incremental MIDX writes. Earlier, Elijah Newren objected that the v2 oidset conversion for extra roots can regress pack quality because walk order affects namehash-based delta selection, and Jeff King reviewed several patches while questioning comment placement and his confidence in patch 8/8. Today Taylor Blau answered: he agreed the double-negative guarding `--keep-pack` args in patch 5/8 is awkward, tracing it to `midx_must_contain_cruft` probably deserving the name `midx_may_exclude_cruft`; defended calling `prepare_revision_walk()` twice on one `rev_info` by citing `get_object_list()` and `mark_reachable_objects()` as existing callers; explained the `item->util` preferred-pack marking would not survive sorting because the last mark wins; and confirmed `midx_included_packs()` does the pack selection in patch 8/8. Jeff King accepted the `prepare_revision_walk()` evidence as "good company" and said the preferred-pack explanation now made sense to him.

### RFC: git-brebase, rebasing onto the first conflicting commit

*Long thread (root 2026/10/03/20-11-47): 12 replies so far from 2 participants over 1 day; 13 messages that day.*

Alejandro Colomar posted an RFC for `git-brebase`, a 164-line shell script that rebases onto the first conflicting commit of a target branch using `git-bisect(1)` internally, supporting `--first-parent`, `--pre-exec`, and `--post-exec`. Nico Williams replied that he still preferred a `--first-conflict`/`--onto-first-conflict` option on `git-rebase(1)` itself, proposing that `git rebase` by default bisect to the first upstream commit introducing conflicts and stop there, with `--continue` re-running toward the target and `--direct` to disable this; he also suggested `getopts`, a `trap` for temp files, and a callback without a temp file, and noted the script breaks detached HEAD mode. Colomar agreed a `trap` makes sense, explained `getopts(1)` is unusable for git scripts because it consumes `--`, argued `--abort` should return only to the start of the current iteration so the behavior belongs in a separate porcelain tool, and later reported a tested patch making `git-brebase` work with detached HEAD by storing the branch or commit in a temp file and using `git checkout` instead of `git switch`. Williams concluded with an untested rework avoiding the temp callback file via a `--bisect-run-callback` sub-command mode, and acknowledged plain `git-rebase` is still wanted when not changing the base.

## The day in brief

Taylor Blau's v2 repack series for cruft-less MIDXs drew responses; Jeff King accepted the `prepare_revision_walk()` evidence as "good company" and said the preferred-pack explanation made sense to him. Alejandro Colomar's `git-brebase` RFC sparked debate with Nico Williams over bisect-to-first-conflict in `git-rebase`. brian m. carlson objected that the sha1dc Rust crate needs Rust 1.87; Johannes Schindelin conceded the "compile-time" title was misleading. Jeff King's "limbo" object-format proposal for empty repos drew protocol alternatives from carlson and Junio C Hamano.

## Notable threads

### filter-branch: fix --state-branch commit map initialization

Grant Moyer posted [PATCH v3] fixing `git-filter-branch`'s `--state-branch` map initialization to interpret object pairs as `$from_commit:$to_commit`, correcting the reversed interpretation introduced in f6d855091e when filter-branch stopped depending on Perl; the patch updates the existing test to use a tree-filter that exits non-zero so it fails if state-branch commits are not skipped, and adds a test verifying incremental rewrites use rewritten parents from the state branch.

### history: sign rewritten commits (v5)

Souma posted v5 of the series adding a signing-key option to the replay API and exposing `-S`/`--gpg-sign[=<key-id>]` and `--no-gpg-sign` on `drop`, `fixup`, `reword`, `split`, and `squash`, so history rewriting honors `commit.gpgSign` and explicit signing requests on both directly rewritten commits and replayed descendants; v5's change since v4 is adding completion coverage for the new options, and Souma confirmed the completion output matches the parse-options behavior shared with `git rebase` and `git commit`.

### Optional sha1dc Rust crate as a SHA-1 backend

Johannes Schindelin's series adds the fast sha1dc Rust crate as an optional SHA-1 backend with a runtime `core.sha1dcBackend` escape hatch defaulting to Rust. Today Schindelin conceded "compile-time" in the title is misleading because `sha1dc_choose()` selects at runtime, explained the Rust and C implementations use the same collision-detection idea but different UBC filters and check groupings over the same per-DV solution space, and reported his AI-assisted analysis and tests found the Rust implementation robust, though his full cross-validation test suite was unfinished. brian m. carlson objected that the crate requires Rust 1.87 while Git declares compatibility with much older Rust, noted gccrs is not yet viable, and proposed any minimum-version increase be handled in a separate series with a policy document.

### fetch.c: defer fetch.followRemoteHEAD validation (v4)

Colin Hinton posted [PATCH v4] deferring validation of `fetch.followRemoteHEAD` in `builtin/fetch.c`: the raw config string is stored and resolved lazily via `get_follow_remote_head()` at the single point in `do_fetch()` that consults it, so a mistyped value only warns and a missing value only dies when this fetch would have used it; `remote.<name>.followRemoteHEAD` in `remote.c`'s `handle_config()` remains unaddressed, marked with a NEEDSWORK comment for a follow-up.

### A "limbo" object-format state for empty repositories

Jeff King proposed letting an empty server-side repository advertise a "limbo" object-format state so a first push from a sha1 or sha256 client sets the server's format, avoiding the mismatched-hash fatal on first push to forge-created repos. brian m. carlson argued `object-format=limbo` is incompatible with every existing object-format-aware client, while Jeff King proposed a new ignorable capability (tentatively "magic-limbo") paired with a valid `object-format=` value; Junio C Hamano remarked that multiple capability lines would have been the obviously right approach. carlson countered that the alternate format must also be declared (e.g. `alt-object-format=sha256`), corrected the earlier bug recollection to an infinite loop fixed by aa962fef27 in 2.41, and argued the policy mechanism (config preferred, since non-push writes also need a hash algorithm) is within Git's scope.

## In brief

- Harald Nordgren posted v5 of the CI series linking failure and leak annotations to the test file and line; Phillip Wood corrected his earlier assessment, noting the link does not navigate when the file was not changed in the PR.
- Maciej Ciemborowicz reported that IDEs such as Codex in VS Code create worktrees directly, bypassing wrappers, so the proposed worktree lifecycle hooks are needed as a repository-local boundary.
- Julia Evans published v2 of "doc: don't require a SYNOPSIS in section 7", incorporating Tuomas Ahola's declarative lint-script rewrite and removing SYNOPSIS from 11 section 7 pages.
- Oscar Mira announced git-carrier 0.1.0, a single bash script implementing `git park`/`git unpark`/`git land` to park a conflicted merge as ordinary commits and land it later as a real merge.
- Fionn Fitzmaurice submitted a patch to `contrib/completion/git-completion.zsh` excluding files already in the argument list from completion candidates via `compadd -F`.
- Alejandro Colomar proposed that `git-bisect(1)` set BISECT_HEAD even without `--no-checkout`, so the commit under test can be referenced after switching branches.
- On a plumbing equivalent of `git bisect visualize`, D. Ben Knoble suggested `git rev-list` over `bisect/bad` with negated good refs, Kristoffer Haugsbakk suggested `git rev-list --bisect`, and Alejandro Colomar leaned toward piping a heredoc into `git bisect run` for skipped commits.
- Kristoffer Haugsbakk reported that `&commat;` in the gitbreaking-changes link syntax renders a working HTML link without a `mailto` href, but suggested dropping the dual-link patch if plain URLs are preferred.
