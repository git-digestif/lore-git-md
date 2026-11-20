# Git mailing list daily digest for 2026/11/02

## Long threads

### `git repo info` path keys series under review pressure

*Long thread: 45 replies so far from 5 participants over 110 days; 6 messages that day.*

K Jayatheerth's GSoC series adds `path.*` keys to `git repo info` (`path.toplevel`, `path.superproject-root`, `path.hooks`, `path.index`, `path.grafts`, `path.git-prefix`, `path.cdup`), with a fix to `get_superproject_working_tree()` so it takes a repository parameter instead of relying on the cwd. Earlier, Junio C Hamano warned the v7 series would be discarded without review. Today he repeated that warning, and Justin Tobler reviewed several patches: he agreed patch 2/8 makes sense passing the repository to `get_superproject_working_tree()`, suggested exercising the existing `rev-parse --show-superproject-working-tree` caller in t1500 instead, simplified patch 3/8's early returns, objected to relative output like `../../../../dev/null` for `path.hooks` when `core.hooksPath=/dev/null`, questioned whether the deprecated `path.grafts` patch should be dropped, and flagged a missing subshell in the `path.cdup` test.

### Reference-transaction hooks reporting old values

*Long thread: 31 replies so far from 5 participants over 45 days; 2 messages that day.*

Maciej Ciemborowicz's v6 patch resolves old object IDs in the common transaction hook layer so the reference-transaction hook reports real old values for unconditional deletions, keeping them separate from caller-supplied values. Earlier, Patrick Steinhardt endorsed the common-layer approach. Today Junio C Hamano said he will discard the topic from his tree unless volunteers review it within a few weeks, and Maciej Ciemborowicz acknowledged, thanking him and hoping someone else will review it.

### Community voting process proposal

*Long thread: 30 replies so far from 13 participants over 28 days; 1 message that day.*

Emily Shaffer's document proposes a community voting process for decisions where consensus cannot be reached, with eligibility thresholds debated earlier (commit counts, activity windows, human-override exceptions). Today Patrick Steinhardt argued that excluding long-inactive people is not necessarily unexpected, since an informed vote requires community context, and said that any human-override process (Junio or the PLC adding specific voters) must be fully transparent--who added whom, with reasoning, published well ahead of time.

### Git 3.0 lowercase-only hex object IDs

*Long thread: 28 replies so far from 7 participants over 97 days; 2 messages that day.*

brian m. carlson's v2 series removes acceptance of uppercase hex in object IDs in breaking-changes mode, addressing security issues where downstream software assumes a unique lowercase hex form. Today Junio C Hamano noted the series had received no reactions and asked for review to confirm it aligns with earlier discussion so it can be marked for `next`; Patrick Steinhardt added it to his todo list for the week.

### Shallow-clone hint for `<rev>~N` failures

*Long thread: 22 replies so far from 6 participants over 44 days; 17 messages that day.*

Harald Nordgren's v3 patch adds a `shallowHistory` advice when `<rev>~N` or `<rev>^N` resolution stops at a recorded shallow boundary, suggesting the exact `--deepen` needed and naming the remote and branch via refspec reverse-mapping. Today the discussion turned to review capacity: Harald Nordgren asked whether the review process itself needs to change, proposed more automated mechanical checks, and offered to trade reviews; Patrick Steinhardt argued automation cannot replace human review of conceptual direction and welcomed tools like Sashiko; Phillip Wood urged self-review and floated requiring review contributions; Junio C Hamano replied that acceptability is shown only by positive reviews and that he now bases `next` decisions heavily on human review after past AI-review reverts.

## The day in brief

Junio C Hamano proposed ejecting topics unreviewed for N weeks from `seen`, with notices for those lingering over a month, and sent discard warnings for stalled series including the `git repo info` path keys and the reference-transaction hook fix. Justin Tobler delivered a detailed review of the path-keys patches, and Kristofer Karlsson's fix making the rev-list connectivity check ignore replace refs was endorsed by Junio C Hamano as absolutely the right thing to do.

## Notable threads

### Ejecting unreviewed topics from `seen`
Junio C Hamano proposed a rule that topics unreviewed for N weeks are ejected from `seen`, announcing notices for every topic lingering unreviewed for more than a month. Patrick Steinhardt called the rule reasonable, noting evicted topics can be re-added if interest returns, and flagged the confusing "Will merge to `next`?" state; Junio explained the trailing question mark means "I plan to do this unless someone stops me," and Phillip Wood suggested clarifying that ejected topics can return to lessen contributor worry.

### `git describe` distance with skewed commit dates
Linus Torvalds' patch fixes `git describe` miscounting distance when commit dates are unreliable by using commit generation numbers. Today Patrick Steinhardt and Derrick Stolee agreed `ensure_generations_valid()` should be called in `describe_commit()`, accepting the cost without a commit graph is better than a wrong answer; Junio C Hamano asked whether the cost could be amortized by writing a commit-graph, and Linus Torvalds argued consistently wrong numbers are worse and suggested a flag to force exact computation.

### Connectivity check ignoring replace refs
Kristofer Karlsson's patch makes rev-list's `--connectivity-check` call `disable_replace_refs()` and drop `OBJECT_INFO_LOOKUP_REPLACE` from `tree-verify.c`, so it verifies the same original objects that fsck, prune, pack-objects and upload-pack read. Junio C Hamano endorsed it as absolutely the right thing to do, noting replacement objects being backed by reachable objects could be addressed separately.

## In brief
- Kristofer Karlsson posted v2 of a fix so `--alternate-refs` no longer lists the repository's own refs (including hidden ones) during quarantine connectivity checks, with a perf test showing a push into a repo with ~200,000 hidden refs improving from 0.31s to 0.09s.
- Phillip Wood reviewed Farid Zakaria's v5 patch making `git rebase --autosquash` honor `--empty` when a fixup empties its target, calling it pretty good but requesting comment restructuring and a rebase onto master.
- Jasper Mayone posted v4 treating a branch being rebased or bisected as checked out: `worktree remove` refuses such worktrees without `--force`, `%(worktreepath)` names the rebasing worktree, and `git branch` drops the `+` marker there. *Long thread: 22 replies so far from 7 participants over 13 days; 5 messages that day.*
- Christian Couder posted v4 standardizing early option scanning with a new `early_scan_options()` sub-API in parse-options, converting `git fast-import`'s `--allow-unsafe-features` handling.
- Mark C. Chu-Carroll posted v5 adding degree-of-concern reporting to `git repo structure`, with thresholds configurable via `repo.concern.*` keys.
- Patrick Steinhardt posted v2 of the CI test-variable consolidation series; Phillip Wood reviewed it as ready for `next`.
- Junio C Hamano proposed freezing the `repo_config_values` structure against new members in `environment.h`.
- Antonin Delpeuch submitted a patch adding `Documentation/Governance.adoc` describing current project governance; Junio C Hamano asked for line-wrapping and clarifications on several points.
