# Git mailing list daily digest for 2026/11/02

## Long threads

### `git repo info` path keys

*Long thread (root 2026/07/16/01-21-31): 45 replies so far from 5 participants over 110 days; 6 messages that day.*

K Jayatheerth's GSoC series adds `path.*` keys to `git repo info` (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, `path.cdup`), including a fix so `get_superproject_working_tree()` takes a repository parameter instead of relying on the cwd. Previously Junio C Hamano warned the v7 series had seen no review for a month and would be discarded without volunteers. Today Justin Tobler reviewed several patches: he agreed patch 2/8 makes sense passing the repository to `get_superproject_working_tree()` but noted its new test fails at that commit; on 3/8 he suggested collapsing early returns into one check; on 4/8 he objected to relative output like "../../../../dev/null" when `core.hooksPath=/dev/null`; on 6/8 he noted a redundant NULL check and questioned dropping the grafts patch since grafts are deprecated; on 8/8 he flagged a style nit and a test whose `cd repo` leaks into later tests.

### reference-transaction hook old values

*Long thread (root 2026/09/19/13-34-15): 31 replies so far from 5 participants over 45 days; 2 messages that day.*

Maciej Ciemborowicz's v6 patch resolves old object IDs in the common transaction-hook layer so the reference-transaction hook reports real old values for unconditional deletions, kept separate from caller-supplied `old_oid` so it cannot constrain updates. Earlier Patrick Steinhardt had argued for exactly this common-layer approach. Today Junio C Hamano said the patch had gone a month without review and would be discarded from his tree unless volunteers step up; Maciej acknowledged, hoping someone else will review it.

### community voting process proposal

*Long thread (root 2026/10/06/18-05-36): 30 replies so far from 13 participants over 28 days; 1 message that day.*

Emily Shaffer's documentation proposal for a community voting process, following the Contributor's Summit, covers voter eligibility and platform choice; earlier discussion ranged over commit-count thresholds, activity windows, abstention and secret votes. Today Patrick Steinhardt argued that excluding long-inactive people is not necessarily unexpected, since an informed vote requires context on how a topic affected the community, and said any human-override process (Junio or the PLC adding specific voters) must be fully transparent--who added whom and why--and public well ahead of time.

### Git 3.0 lowercase-only hex object IDs

*Long thread (root 2026/07/29/23-32-09): 28 replies so far from 7 participants over 97 days; 2 messages that day.*

brian m. carlsson's v2 series removes acceptance of uppercase hex in object IDs in Git 3.0, motivated by software that assumes lowercase and security risks from non-unique hex forms. Today Junio C Hamano noted the series had drawn no reactions and asked for review to confirm it matches earlier discussion so it can be marked for 'next'; Patrick Steinhardt added it to his todo list for the week.

### shallow-clone history hint and review-process debate

*Long thread (root 2026/09/20/09-53-33): 22 replies so far from 6 participants over 44 days; 17 messages that day.*

Harald Nordgren's v3 patch adds a `shallowHistory` advice when `<rev>~N` resolution stops at a shallow boundary, suggesting the exact `--deepen` and naming the remote and branch via refspec reverse-mapping. Today the thread turned to review capacity: Harald asked whether the review process itself needs to change; Kristoffer Haugsbakk and Patrick Steinhardt discussed scarce reviewer capacity, automation and distrust of AI rubber-stamping; Phillip Wood urged self-review and floated requiring review contributions; Junio C Hamano replied that acceptability is shown only by positive reviews, recalled reverting AI-reviewed topics in a past release, and rejected letting contributors throw AI-generated "reviews" at others' threads.

## The day in brief

Junio C Hamano proposed ejecting topics unreviewed for weeks from 'seen' and warned that Maciej Ciemborowicz's reference-transaction patch would be discarded without review. Justin Tobler reviewed several patches in K Jayatheerth's `git repo info` path-keys series, and Kristofer Karlsson posted fixes so quarantine and rev-list connectivity checks verify the right objects.

## Notable threads

### ejecting unreviewed topics from 'seen'
Junio C Hamano proposed a rule that topics unreviewed for N weeks are ejected from 'seen', announcing notices for everything lingering over a month. Patrick Steinhardt called the rule reasonable since evicted topics can be re-added, and flagged the confusing "Will merge to 'next'?" state; Junio explained the trailing question mark means "I plan to do this unless someone stops me," and Phillip Wood suggested clarifying that ejected topics can return with renewed interest.

### sequencer `--empty` with fixup/squash
Farid Zakaria's v5 patch makes `git rebase --autosquash --empty=drop` work when a `fixup!` empties its target. Phillip Wood called it "pretty good" but requested many changes: clearer comments, moving an `is_rebase_i()` check earlier, `BUG()` guards for merge commits, restructuring the emptiness check, and error checking when reading the fixup_target file; he also noted the patch does not apply to master.

### `describe` distance with skewed commit dates
Linus Torvalds's patch fixes `git describe` miscounting distance when commit dates are skewed by using generation numbers. Today Patrick Steinhardt and Derrick Stolee backed calling `ensure_generations_valid()` in `describe_commit()` despite the cost without a commit-graph; Junio asked whether that cost could be amortized by writing a commit-graph; Linus explained the kernel's localversion is historical and argued consistently wrong numbers are worse than odd ones.

### quarantine alternate refs
Kristofer Karlsson posted v2 of a patch so `--alternate-refs` no longer lists the repository's own refs (including hidden ones) during quarantine connectivity checks, using a "borrowed" flag on alternate object directories; a perf test showed a push into a repo with ~200,000 hidden refs improving from 0.31s to 0.09s.

### connectivity check ignores replacements
Kristofer Karlsson posted a patch making rev-list's connectivity check ignore replace refs, matching fsck, prune, pack-objects and upload-pack; Junio C Hamano endorsed it as "absolutely the right thing to do."

## In brief
- **Jasper Mayone posted v4 worktree series**: `git worktree remove` refuses worktrees mid-operation, now also single cherry-pick/revert via CHERRY_PICK_HEAD/REVERT_HEAD; `%(worktreepath)` names the rebasing worktree; `git branch` drops "+" only in that worktree. *Long thread (root 2026/10/21/17-33-36): 22 replies so far from 7 participants over 13 days; 5 messages that day.*

- Patrick Steinhardt posted v2 of "ci: consolidate test variables"; Phillip Wood reviewed it as ready for next.
- Junio C Hamano found Lorenz Leutgeb's doc patch adding 'if-asked' to the pushcert option good enough and marked it ready for 'next'.
- Patrick Steinhardt criticized Elia Pinto for repeating wording he had asked to change, warning he will ignore the series; Junio C Hamano asked what response would have satisfied him.
- Phillip Wood pointed to his own patches in 'seen' addressing the same stash duplicate-work issue differently and asked Jasmeet Bhatia to review them.
- Christian Couder posted v4 of "Standardize early option scanning", adding `early_scan_options()` to parse-options and converting fast-import's `--allow-unsafe-features` scan.
- Andy Koppe explained to Julia Evans that root refs like ORIG_HEAD can be stored in reftables and updated via update-ref, while pseudorefs (FETCH_HEAD, MERGE_HEAD) are plain files that may hold multiple hashes.
- Antonin Delpeuch submitted a Governance.adoc describing current project structures; Junio C Hamano asked for line-wrapping and questioned some characterizations.
- On Harald Nordgren's `--matched-only` range-diff patch, Ben Knoble asked for motivation, Harald gave and then retracted a rebasing use case, and Junio noted the unanswered question of whether combining left_only and right_only makes sense.
