# Git mailing list daily digest for 2026/11/03

## The day in brief

The Git mailing list saw significant activity around **`git repo info` path keys**, with design decisions finalized on `/dev/null` handling and the `path.grafts` inclusion debate. **`git describe`** gained a critical regression fix for commit date skew, now queued with an opt-out `--exact` flag. **`git grep --recurse-submodules`** received a bugfix for nested submodule trees, now cooking in `next`. Documentation updates dominated discussions, with the `git diff` EXAMPLES section rewrite sparking debate over presentation format and terminology consistency.

## Notable threads

### `git repo info` path keys series reaches critical design decisions
The v7 series adding path-related keys to `git repo info` saw key design resolutions today. K Jayatheerth and Justin Tobler agreed to special-case `/dev/null` handling for both `path.hooks.absolute` and `path.hooks.relative` keys, returning `/dev/null` verbatim when `core.hooksPath` is set to that value. This improves usability for scripts and eliminates Windows path resolution inconsistencies. The `path.grafts` inclusion debate remains unresolved, with Justin questioning whether the deprecated graft files should be exposed at all, while K Jayatheerth notes the keys were included based on earlier discussions and the GSoC idea page. The series is at critical risk of abandonment due to persistent lack of reviewer engagement.

### `git describe` regression fix converges on opt-out `--exact` flag
Linus Torvalds' patch fixing `git describe`'s commit distance calculation when commit timestamps are non-monotonic gained consensus on its user-facing design. The discussion settled on an opt-out `--exact` flag (defaulting to on) with a warning mechanism when the commit graph is unavailable. Linus endorsed Patrick Steinhardt's counterproposal, removing the last substantive objection. The patch is now queued for integration, addressing a real-world regression in the Linux kernel's version reporting during the v7.4 merge window.

### `git grep --recurse-submodules` fix for nested trees cooking in `next`
Patrick Steinhardt's three-patch series fixing `git grep --recurse-submodules` for nested submodule trees was marked for `next` after addressing all review feedback. The series ensures submodule trees, sparse directory trees, and `.gitmodules` files are read via their correct repository instance rather than `the_repository`. The fix targets a regression introduced in Git v2.56.0 and is explicitly intended for backporting to stable branches. Junio's merge signal confirms the series' readiness after Toon Claes's tested review.

### Documentation updates spark debate over presentation and consistency
Julia Evans' rewrite of the `git diff` EXAMPLES section generated substantial discussion about documentation presentation and terminology consistency. Junio objected to the table format (citing line-wrapping and cut-and-paste issues), the promotion of `--staged` over `--cached`, and the omission of `git diff AUTO_MERGE`. While Julia conceded the latter two points, the presentation format dispute remains unresolved. The thread highlights tensions between modernization and preserving historical context in documentation.

## In brief

- **`parse-options` early scanning API** marked for `next` after Kaartic Sivaraam's substantive review confirmed all feedback was addressed.
- **`git bundle-uri`** gained a v2 patch fixing a crash when processing relative URIs with excessive `../` components, addressing Patrick Steinhardt's review feedback.
- **`git log --graph` orphaned branch rendering** received a bugfix patch from Caspar Duregger, with Pablo Sabater's approval.
- **`git reflog` `%gb` format placeholder** identified a regression in branch-name detection when `GIT_REFLOG_ACTION` is set.
- **`git maintenance` lock-handling** series agreed to split into two patches (behavioral change and Trace2 event) after Patrick Steinhardt's review.
- **`git credential-cache` daemon** race condition fix saw architectural feedback from Junio about mixing `unix_ss_*` and `tempfile` APIs.
- **`git push --force-with-lease`** reported as silently ignored when pushing to remote groups unless an explicit `<refname>` is provided.
- **`git diff --word-diff`** bug reported for incorrect output when comparing files with both textual changes and whitespace additions.
- **`git branch --switch`** RFC proposed adding a `-s`/`--switch` option, but faced skepticism about redundancy with `git switch -c`.
- **Outreachy proposals** saw discussion about expanding scope to include `builtin/` refactoring in the `the_repository` removal effort.
- **CI fixes** for Ubuntu 26.04 compatibility queued after Junio's review.
- **Rust `hash` module** gained a bugfix for panics when formatting `ObjectID` with invalid algorithms.

The day's discussions reflect Git's ongoing evolution, with particular focus on usability improvements, regression fixes, and the gradual removal of global state to enable libification. Documentation modernization continues to generate debate about balancing clarity with historical context.