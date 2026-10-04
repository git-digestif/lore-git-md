# Git mailing list daily digest for 2026/10/03

## The day in brief
The Git mailing list saw active discussion on several fronts today. The `git history` signing series resolved its last blocker with a follow-up patch adding completion test coverage. A real-world use case for worktree hooks emerged, strengthening the observability argument. The MIDX reachability series advanced with clarifications on preferred-pack tracking, while the Rust SHA-1 backend faced policy pushback on its Rust version requirement. Documentation efforts continued, with the `gitmergeconflicts(7)` guide gaining a foundational "WHAT IS A MERGE CONFLICT?" section and the `gitbreaking-changes(7)` RFC hitting a formatting snag. CI improvements for GitHub Actions were blocked by a critical usability regression in annotation links.

## Notable threads

### Teach `git history` to sign rewritten commits
**What changed?**
Souma’s v5 series for signing rewritten commits (`drop`, `fixup`, `reword`, `split`, `squash`) is now unblocked after addressing the last technical hurdle: a mechanical regression in `t9902-completion.sh`. The follow-up patch adds `--gpg-sign` and `--no-gpg-sign` to the expected completion output for `git history split`, matching the behavior already established for `git rebase` and `git commit`.

**Why it matters**
The series teaches `git history` to respect the same configuration (`commit.gpgsign`) and command-line options (`-S/--gpg-sign`, `--no-gpg-sign`) as other Git commands, ensuring consistent signing behavior across the toolchain. The completion fix removes the last blocker for graduation to `next`, where the series has been cooking since v4.

**Technical details**
- Files touched: `t/t9902-completion.sh` (two lines added to expected output for `git history split`).
- The fix aligns the completion script with the internal `--git-completion-helper` output, which lists all options before `--` and generated negated options after it.
- No changes to the core signing logic or replay API plumbing; the series remains focused on user-facing consistency.

---
### Worktree hooks for observability
**What changed?**
Maciej Ciemborowicz provided a real-world use case for Domen Kožar’s proposed `post-worktree` hook: IDEs (specifically VS Code with Codex) create worktrees directly, bypassing any wrapper script. His tool provisions isolated container environments for each worktree, but without hooks, there is no reliable way to detect creation or removal events when the IDE—not the user or agent—initiates them. This results in races between worktree creation and agent startup, and between worktree removal and environment cleanup.

**Why it matters**
The report strengthens Domen’s observability argument by demonstrating that wrappers are insufficient when the caller is uncontrolled. It provides concrete, high-stakes evidence (parallel AI agents in IDEs) that the problem is not hypothetical, directly addressing Junio C Hamano’s earlier skepticism about the necessity of native hooks.

**Technical details**
- Use case: VS Code with Codex creates worktrees directly, making wrappers ineffective for observability.
- Impact: races between worktree lifecycle events and agent provisioning/cleanup.
- No code changes proposed; the report is purely evidence-based and does not engage with Kristoffer Haugsbakk’s `--id=<string>` counterproposal or the potential complementarity of hooks and IDs.

---
### MIDX reachability closure corner cases
**What changed?**
Taylor Blau and Jeff King (Peff) resolved two substantive review points in the MIDX reachability series. Patch 6/8, a refactoring in `repack-midx.c` to explicitly track the preferred pack, is now uncontroversial after Blau clarified that the existing code marks *multiple* preferred packs (one per MIDX layer during compaction) and selects the *last* marked entry, making the choice order-dependent. Sorting the string list would preserve the marks but could change which pack is last, breaking the intended behavior. Peff accepted the design as correct and endorsed the patch as a cleaner alternative to the current pointer-cast-heavy implementation.

**Why it matters**
The series fixes corner cases where MIDXs can end up containing objects not closed under reachability, violating the MIDX’s invariant and causing failures in bitmap generation. The preferred-pack refactoring removes dependency on string-list order and enables future optimizations like sorting for faster membership checks.

**Technical details**
- Patch 6/8 introduces a `preferred_pack` field in the `midx_compaction_step` struct.
- The existing code’s order-dependency is now documented and preserved.
- Peff’s endorsement confirms the refactoring’s correctness and readiness for integration.

---
### Rust-based SHA-1 backend
**What changed?**
Brian M. Carlson raised a policy-level concern about the Rust version requirement in Johannes Schindelin’s Rust-based SHA-1 backend series. The series requires Rust 1.87, but Git’s `Cargo.toml` currently targets Rust 1.49.0, and the project’s informal policy aligns with Debian stable (Rust 1.85.1). Carlson argues this version bump should be handled in a separate policy discussion, not as part of this feature series.

**Why it matters**
The objection is procedural, not technical, but it is framed as a blocker unless addressed through broader consensus. The performance gains (3× speedup in `git index-pack` on Windows) may not justify a unilateral version bump without project-wide agreement on Rust toolchain support.

**Technical details**
- Rust version requirement: 1.87 (vs. Git’s current 1.49.0 and informal target of 1.85.1).
- Carlson notes that support for gccrs (a GCC-based Rust compiler) is now unrealistic given its slow progress and Git 3.0’s impending release.
- The series remains under review, with the version policy question now the primary open issue.

---
### `gitmergeconflicts(7)` documentation
**What changed?**
D. Ben Knoble endorsed Julia Evans’ proposed "WHAT IS A MERGE CONFLICT?" section for the new `gitmergeconflicts(7)` guide, resolving the last open stylistic question. Knoble praised the section’s clarity and noted that its oversimplification of 3-way merges (e.g., omitting the common ancestor) does not harm user understanding. Junio C Hamano conceded that the term "unstaged" is reasonable in the guide’s example-driven phrasing, despite its potential ambiguity in other contexts.

**Why it matters**
The new section addresses a critical gap in the guide: users unfamiliar with merge conflicts now have a foundational explanation before diving into resolution techniques. The endorsement and Junio’s concession remove the last obstacles to including the section in the next revision (v2).

**Technical details**
- Section title: "WHAT IS A MERGE CONFLICT?"
- Key phrasing retained: "Git will not try to guess how to combine the changes, because it can’t tell which one is correct. Instead, it marks the file as conflicted and leaves it up to you to decide."
- Junio’s suggested refinements (e.g., "as there is no overlap" instead of "it can easily combine them") were incorporated, but Julia retained "will not try to guess" to emphasize Git’s intentional conservatism.

---
### CI improvements for GitHub Actions
**What changed?**
Phillip Wood identified a critical usability regression in Harald Nordgren’s CI improvement series: the new annotation links on the GitHub Actions summary page only work if the test file was modified in the same pull request. Otherwise, they point to a broken or irrelevant location, directly contradicting the patch’s core goal of providing actionable failure links.

**Why it matters**
The regression undermines the series’ primary value proposition. The links are useless or broken unless the test file was changed in the PR, making the feature counterproductive for most users. The series remains in `next` but cannot graduate until this issue is resolved.

**Technical details**
- Regression: annotation links point to the file diff view, which is useless if the test file was not modified.
- Phillip provided live GitHub URLs demonstrating the broken behavior in the v3 CI run.
- The issue is a functional regression that must be addressed before the series can proceed.

---
### `gitbreaking-changes(7)` RFC
**What changed?**
Kristoffer Haugsbakk pushed back on Junio C Hamano’s proposed hybrid URL+message-ID format for message-ID references in the new `gitbreaking-changes(7)` manpage. Haugsbakk cited usability concerns in HTML output (message-IDs rendered as `mailto:` links) and the visual similarity of short message-IDs to email addresses, preferring plain URLs. He suggested dropping the patch if consensus cannot be reached, as the current URL-only approach is already an improvement over raw message-IDs.

**Why it matters**
The disagreement is narrow but technical, centering on documentation formatting rather than the series’ broader goal. The hybrid format is a refinement, not a blocker, but the pushback highlights the tension between preserving raw message-IDs for future reference and prioritizing user experience.

**Technical details**
- Proposed hybrid format: `cf. https://lore.kernel.org/git/xmqqa59i45wc.fsf@gitster.g/[<xmqqa59i45wc.fsf@gitster.g>^]`.
- Haugsbakk’s concerns: message-IDs rendered as `mailto:` links in HTML, visual similarity to email addresses.
- Two message-IDs required URL encoding to avoid breaking AsciiDoc parsing.

---
### `git brebase` RFC
**What changed?**
The `git brebase` RFC saw active discussion today, with Nico Williams and Alejandro Colomar exchanging implementation refinements and design perspectives. Colomar added detached HEAD support to the shell script, addressing a key limitation identified by Williams. Williams proposed embedding the callback logic directly into the main script via a `--bisect-run-callback` flag and positional arguments, eliminating the need for a temporary file. The broader design debate—whether the functionality should be a new command, a flag for `git rebase`, or the default behavior—remains unresolved, but the script’s robustness improved.

**Why it matters**
The RFC proposes a tool for incrementally rebasing complex histories (branches with merge commits, arbitrary commit skipping) by automating conflict detection and resolution up to the first semantic conflict. The discussion explores whether this workflow should live in a new command (`git brebase`), be added as a flag to `git rebase`, or become the default behavior of `git rebase` itself.

**Technical details**
- Detached HEAD support: writes the current commit hash to a temporary file when no branch name is available.
- Callback embedding: uses `--bisect-run-callback` flag and positional arguments to pass values like `$branch`, `$gropts`, `$pre`, and `$post`.
- No changes to Git itself proposed yet; the discussion remains conceptual.

## In brief
- **[filter-branch]**: Grant Moyer’s v3 patch for the `--state-branch` commit mapping inversion regression is ready for merging after addressing Junio’s final review requests.
- **[precompile git-compat-util.h]**: SZEDER Gábor reported a side effect of precompiled headers: the `__FILE__` macro in `git-compat-util.h` expands to `"./git-compat-util.h"` instead of `"git-compat-util.h"`, affecting error messages in assertions and compiler diagnostics.
- **[fetch.followRemoteHEAD]**: Colin Hinton’s v4 defers validation of `fetch.followRemoteHEAD` until `do_fetch()` consults it, introducing a `follow_remote_head_raw` field and `get_follow_remote_head()` helper.
- **[git-interpret-trailers]**: Kristoffer Haugsbakk’s v2 updates `trailer.<key-alias>.cmd` examples to match the style used in the `see` trailer example.
- **[limbo object-format]**: Brian M. Carlson proposed advertising both SHA-1 and SHA-256 (e.g., `object-format=sha256` and `alt-object-format=sha1`) to explicitly signal support for SHA-256, citing a 2023 fix for multi-valued capability parsing.
- **[bisect plumbing]**: Alejandro Colomar and D. Ben Knoble explored plumbing alternatives to `git bisect visualize`, with Knoble suggesting `git rev-list --bisect` and its variants as a potential solution.
- **[BISECT_HEAD]**: Alejandro Colomar requested `BISECT_HEAD` be set unconditionally, even when `--no-checkout` is not used.
- **[Zsh completion]**: Fionn introduced a `__git_file_exclude` array and `compadd -F` to exclude already-specified file arguments from Zsh completion candidates.