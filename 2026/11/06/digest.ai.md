# Git mailing list daily digest for 2026/11/06

## The day in brief
The Git mailing list saw active discussion on several fronts today. Key developments include maintainer feedback on the global config file handling series (now technically complete), a policy update for the `seen` integration branch to frame ejections as reversible, and substantive reviews identifying correctness issues in the `%gb` reflog format placeholder and `git log --follow` performance series. Additionally, a bug in `git init`'s default object format under `WITH_BREAKING_CHANGES=1` was confirmed and a fix is in progress.

## Notable threads

### Global config file handling inconsistency fix
**What changed?**
Junio C Hamano provided final maintainer feedback on Delilah Ashley Wu's v3 series fixing Git's global config file handling inconsistency. The series is now technically complete with all feedback addressed.

**Problem/goal**
The series fixes an inconsistency where `git config list --global` only showed `$HOME/.gitconfig` while Git actually reads both that and `$XDG_CONFIG_HOME/git/config` for unscoped operations.

**Subsystem**
Configuration handling (`config.c`, `builtin/config.c`)

**Kind of change**
Bugfix

**Impact**
The fix ensures consistent behavior between config listing and actual config reading operations, improving user experience and script reliability.

**Today's developments**
- [2026/11/06/13-08-09] Junio accepted the first patch ("t1300: test list with missing global config") with a minor question about test portability regarding XDG paths
- [2026/11/06/13-28-09] Junio accepted the second patch ("config: read global scope via config_sequence") with a request for a defensive `BUG()` check in `git_config_system()` to ensure no caller passes a NULL `opts` pointer

The series is now ready for integration pending this minor defensive programming tweak.

### Integration pipeline policy update
**What changed?**
Junio C Hamano adopted Phillip Wood's suggestion to explicitly frame ejection from the `seen` branch as reversible.

**Problem/goal**
The policy change aims to reduce contributor anxiety by emphasizing that ejection from `seen` is temporary and contingent on renewed engagement, not a judgment on the patch's merit.

**Subsystem**
Integration pipeline/workflow

**Kind of change**
Policy update

**Impact**
This social change makes the `seen` branch more welcoming to contributors by signaling that ejection is a low-stakes, reversible action rather than a final rejection.

**Today's development**
- [2026/11/06/20-38-57] The maintainer updated the "Discarded" section wording in "What's cooking" reports to: *"Many of these topics were picked up prematurely but did not prove ready for inclusion, and will be removed. They can be resubmit if they gained enough interest and showed good fit to the system in the future, unless noted otherwise."*

### Reflog format placeholder correctness issues
**What changed?**
Phillip Wood identified two remaining correctness issues in the `%gb` reflog format placeholder implementation.

**Problem/goal**
The `%gb` placeholder is meant to show the branch name for each reflog entry, but currently has issues with detached HEAD entries and rebase checkout parsing.

**Subsystem**
Reflog walking (`reflog-walk.c`)

**Kind of change**
Bugfix

**Impact**
These issues could undermine the feature's reliability for users tracking branch history through complex operations like rebases.

**Today's development**
- [2026/11/06/11-08-05] Phillip identified:
  1. Detached HEAD entries are mislabeled with the commit *before* switching away rather than the commit where HEAD was originally detached
  2. The rebase checkout parsing logic is fragile, incorrectly treating remote branch names as local and being confused by commit subjects starting with "checkout "
- Phillip provided a failing test case that exercises both issues

The series will need to address these issues before merging.

### `git log --follow` performance and correctness
**What changed?**
Patrick Steinhardt identified correctness regressions in Miklos Vajna's `git log --follow` performance series.

**Problem/goal**
The series aims to improve performance of `git log --follow` when tracking file history across complex merge topologies, but has introduced correctness issues.

**Subsystem**
Revision walking (`revision.c`, `log-tree.c`)

**Kind of change**
Performance optimization with correctness fixes

**Impact**
The identified issues could cause `git log --follow` to miss important history in certain merge scenarios.

**Today's developments**
- [2026/11/06/14-14-03] Patrick identified a potential regression where `follow_needs_renames()` disables rename detection for commits with unrelated histories
- [2026/11/06/14-14-17] Patrick identified a correctness regression in file copy handling: the new logic fails to track edits to the *source* of a copy when the merge keeps the copied file but drops the original

The series will need significant revision to address these issues.

### Git v3 compatibility bug
**What changed?**
Brian M. Carlson confirmed a bug in `git init`'s default object format when Git is built with `WITH_BREAKING_CHANGES=1`.

**Problem/goal**
`git init` incorrectly defaults to SHA-1 even when Git is built with `WITH_BREAKING_CHANGES=1`, which should enable Git v3 behavior where SHA-256 is the expected default.

**Subsystem**
Repository initialization (`builtin/init-db.c`)

**Kind of change**
Bugfix

**Impact**
This bug affects downstream consumers like Nixpkgs that rely on the breaking-changes flag to test Git v3 compatibility.

**Today's developments**
- [2026/11/06/16-50-03] Adam Dinwoodie reported the issue
- [2026/11/06/20-46-24] Brian M. Carlson confirmed the bug and reported having a fix already prepared and undergoing CI testing

## In brief

- **[2026/11/06/14-05-09]** D. Ben Knoble suggested simplifying a test case in the submodule subsystem by directly creating a branch at a non-ref start point
- **[2026/11/06/14-10-10]** D. Ben Knoble confirmed the technical soundness of a submodule bugfix but recommended improvements to commit message and in-code comments
- **[2026/11/06/04-03-40]** Junio asked Matt Hunter to test his alternative implementation for the `remote.<name>.followRemoteHEAD` series in the `seen` branch
- **[2026/11/06/09-34-46]** Matt Hunter reported that his alternative implementation produces the same mechanical conflict with `hn/fetch-refmap` as the original
- **[2026/11/06/16-15-10]** Junio endorsed Matt Hunter's folded-in `get_follow_remote_head()` rewrite as "much easier for callers" and "very cleanly done"
- **[2026/11/06/00-36-16]** Junio raised a design concern about hardcoding option strings in the `git rev-list` option-parsing fix
- **[2026/11/06/13-25-56]** The author proposed refactoring the early option scan to reuse the main parser's option-recognition logic
- **[2026/11/06/18-18-59]** Junio challenged the architectural premise of the proposed refactor, suggesting to eliminate the early scan entirely
- **[2026/11/06/14-14-03]** Patrick Steinhardt reiterated mailing-list norms about inline replies and requested a `range-diff` for future iterations
- **[2026/11/06/18-11-45]** Aniket Gupta identified a security edge case in the `git apply` regression fix
- **[2026/11/06/20-36-07]** Junio questioned the fundamental premise of the `git apply` fix, asking whether the scenario represents valid input or corrupt data
- **[2026/11/06/12-10-24]** Antonin Delpeuch proposed adding a dedicated section for communication channels in the governance documentation
- **[2026/11/06/18-31-14]** Junio responded to governance documentation questions but didn't know who moderates IRC/Discord
- **[2026/11/06/00-03-07]** Brian M. Carlson resolved the open question about algorithm ID 0 in the Rust hash module
- **[2026/11/06/11-16-46]** Patrick Steinhardt acknowledged concerns about the `QUICK` flag in the ODB cache invalidation fix
- **[2026/11/06/16-02-13]** Junio endorsed landing the ODB cache invalidation fix while documenting the broader `QUICK` flag limitations
- **[2026/11/06/08-50-01]** Karthik Nayak added `OPT_ALIAS` to the `parse-options` API documentation
- **[2026/11/06/10-37-23]** Karthik Nayak suggested replacing a `for` loop with dynamic backend detection in the ODB test suite
- **[2026/11/06/11-01-58]** Patrick Steinhardt declined the suggestion, arguing the test should explicitly verify ref storage format behavior
- **[2026/11/06/16-27-44]** Ravi Mistry accepted a 10-second default for `http.retryAfter` with exponential backoff
- **[2026/11/06/20-24-06]** Christian Couder addressed three of four commands in an Outreachy proposal about `early_scan_options()`
- **[2026/11/06/17-20-57]** Junio questioned the performance difference between `Expand-Archive` and `tar(1)` in a CI speedup patch
- **[2026/11/06/17-33-01]** Junio suggested a stylistic refinement to a test script for better readability
- **[2026/11/06/17-39-47]** Junio acknowledged the CI speedup series as "interesting" and "fully made sense"
- **[2026/11/06/19-36-02]** Junio objected to a `memset()`-based refactor on stylistic grounds, advocating for compound literals
- **[2026/11/06/14-54-32]** Patrick Steinhardt confirmed the core problem in a test fix for `t5608-clone-2gb.sh`
- **[2026/11/06/15-12-38]** The author confirmed that `fetch.unpackLimit` has never applied to `git clone`
- **[2026/11/06/12-51-15]** The author reported a bug in `unpack-objects` where blob sizes >4GB are truncated on 32-bit platforms
- **[2026/11/06/12-51-16]** The author added assertions to `t5608-clone-2gb.sh` to ensure packs are truly the sizes they claim
- **[2026/11/06/13-18-36]** Shashank S posted a revised Outreachy proposal for implementing promisor remote fetch ordering
- **[2026/11/06/21-06-38]** Alejandro Colomar conceded the maintainer's argument against adding a `--type` filter to `git ls-files`
- **[2026/11/06/16-44-06]** Srujith Poojary posted an Outreachy proposal to eliminate a hidden global dependency in pathspec skip-worktree matching
- **[2026/11/06/17-26-48]** Fien Dora proposed refactoring `git column` argument parsing to use `early_scan_options()`
- **[2026/11/06/17-44-49]** Joie Joie proposed replacing manual pack iteration with `repo_for_each_pack()` in `fast-import.c`
- **[2026/11/06/21-02-10]** Junio posted the "What's cooking in git.git" report for November 2026
- **[2026/11/06/01-59-11]** Junio reviewed a stash optimization patch as "makes sense"
- **[2026/11/06/01-59-12]** Junio reviewed a stash optimization patch as "an obvious simplification. Very nice."
- **[2026/11/06/07-36-46]** Kaartic Sivaraam reported that updated microproject guidelines are now live
- **[2026/11/06/10-51-13]** Karthik Nayak suggested documenting that `RERERE_NOAUTOUPDATE` outputs conflict resolution information to stderr
- **[2026/11/06/10-56-58]** Karthik Nayak confirmed that preparatory commits for the `git rerere` subcommand modernization are straightforward
- **[2026/11/06/19-53-13]** Cynthia Uche suggested minor copy-edits to the `parse-options` API documentation
- **[2026/11/06/10-40-23]** Karthik Nayak confirmed a change to `git rev-parse` as "makes sense"
- **[2026/11/06/23-16-31]** Junio suggested alternatives for an error message in the glossary documentation
- **[2026/11/06/21-31-03]** Andy Koppe posted v3 of the glossary and documentation alignment series
- **[2026/11/06/21-31-04]** Andy Koppe posted the first patch of the glossary documentation series
- **[2026/11/06/21-31-05]** Andy Koppe posted the second patch of the glossary documentation series
- **[2026/11/06/21-31-06]** Andy Koppe posted the third patch of the glossary documentation series
- **[2026/11/06/21-31-07]** Andy Koppe posted the fourth patch of the glossary documentation series
- **[2026/11/06/21-31-08]** Andy Koppe posted the fifth patch of the glossary documentation series
- **[2026/11/06/22-02-57]** Andy Koppe replied to Julia Evans's feedback on the glossary's "pseudoref" entry
- **[2026/11/06/22-13-34]** Junio reviewed the first patch of the glossary documentation series
- **[2026/11/06/22-16-56]** Junio reviewed the second patch of the glossary documentation series
- **[2026/11/06/23-08-15]** Junio reviewed the third patch of the glossary documentation series