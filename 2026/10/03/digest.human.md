# Git mailing list daily digest for 2026/10/03

## The day in brief
The Git mailing list saw active discussion on several fronts today. The most consequential developments include a critical usability regression identified in GitHub Actions CI failure reporting, a Rust-based SHA-1 backend series facing policy-level objections over Rust version requirements, and ongoing design debates about incremental rebase workflows. Documentation improvements and build system optimizations also advanced, with some threads nearing completion.

## Notable threads

### GitHub Actions CI failure reporting regression
Harald Nordgren’s two-patch series improving GitHub Actions failure reporting is now blocked by a critical usability regression. Phillip Wood identified that the new annotation links on the GitHub Actions summary page **only work if the test file was modified in the same pull request**; otherwise, they point to a broken or irrelevant location. This directly contradicts the patch’s core goal of providing actionable failure links.

The series had previously addressed all substantive review feedback, including removing percent-sign escaping and improving commit messages. The regression surfaces when a test fails in a file that wasn’t changed by the patch under test—clicking the annotation now takes users to the file diff view rather than the test output. This is a functional regression that must be resolved before the series can proceed.

**What’s at stake**: The patches aim to make test failures easier to debug by linking directly to the exact line in the test script where the failure occurred. The regression undermines this usability improvement and risks making the feature worse than the current behavior.

### Rust-based SHA-1 backend and Rust version policy
Johannes Schindelin’s RFC series introducing a Rust-based SHA-1 backend received a policy-level objection from brian m. carlson. The series requires Rust 1.87, but Git’s `Cargo.toml` currently targets Rust 1.49.0, and the project’s informal policy aligns with Debian stable (Rust 1.85.1). Carlson argues this version bump should be handled in a separate policy discussion, not as part of this feature series.

Schindelin clarified that the Rust and C backends differ in their collision-detection implementation (SIMD-optimized UBC filter in Rust vs. original DV checks in C) but cover the same affine solution space over GF(2). The series is positioned as a performance optimization with an escape hatch (`core.sha1dcBackend=c`), but the Rust version requirement remains a point of friction.

**What’s at stake**: The series promises a 3× performance improvement for `git index-pack` on Windows and large repositories, but the version requirement could block integration unless the project revisits its Rust version policy.

### Incremental rebase workflows: `git brebase` vs. `git rebase` integration
The RFC thread proposing `git brebase` as a simpler alternative to `git imerge` for complex rebases saw active design discussion today. Alejandro Colomar refined the 164-line shell script to address feedback, including adding `trap` for temporary file cleanup, filtering the callback script filename from `git bisect run` output, and fixing a detached HEAD support bug.

Nico Williams countered with a proposal to integrate bisect-driven conflict resolution into `git rebase` as the default behavior when conflicts arise. The debate centers on whether the functionality should be a new command (`git brebase`), a flag for `git rebase` (e.g., `--first-conflict`), or the default behavior. Colomar argues for a standalone command to preserve `git rebase`’s plumbing role, while Williams frames the incremental workflow as the *intended* design of `git rebase`.

**What’s at stake**: The discussion touches on Git’s design philosophy—whether `git rebase` should remain a simple plumbing command or evolve to handle more complex workflows natively. The outcome could shape how users interact with Git’s rebase functionality for years to come.

### MIDX reachability closure corner cases
Taylor Blau’s eight-patch v2 series fixing MIDX reachability closure corner cases saw substantive follow-ups today. The series addresses scenarios where MIDXs can end up containing objects not closed under reachability, violating the MIDX’s invariant and causing bitmap generation failures.

Key developments:
- Jeff King endorsed patch 4/8 (a mechanical refactoring replacing a linear search with a binary search) as correct and sensible.
- Patch 6/8, which introduces a `preferred_pack` field to explicitly track the preferred pack, was confirmed as necessary after Blau clarified that the existing code marks *multiple* preferred packs and selects the *last* marked entry, making the choice order-dependent.
- The data structure choice for `extra_roots` in patch 2/8 remains contested. Elijah Newren’s test cases demonstrated that an `oid_array` preserves critical path and namehash ordering required for delta compression quality, while an `oidset` would introduce functional regressions.

**What’s at stake**: The series fixes subtle but important bugs in Git’s multi-pack-index machinery, which is critical for performance in large repositories. The unresolved `oidset` vs. `oid_array` debate could impact delta compression quality and memory usage.

### Documentation improvements
Several documentation threads advanced today:
- Julia Evans’ `gitmergeconflicts(7)` series saw progress on the "WHAT IS A MERGE CONFLICT?" section, with D. Ben Knoble endorsing the clarity and Junio C Hamano resolving the last open stylistic question about the term "unstaged."
- Kristoffer Haugsbakk’s `gitbreaking-changes(7)` RFC series saw pushback on Junio’s proposed hybrid URL+message-ID format for message-ID references, with Haugsbakk citing usability concerns in HTML output.
- Haugsbakk also updated the `git-interpret-trailers` man page to improve the phrasing of `trailer.<key-alias>.cmd` examples.

**What’s at stake**: These threads aim to improve Git’s documentation by centralizing merge conflict guidance, making breaking changes more discoverable, and clarifying trailer configuration examples. The outcomes will affect how users learn and interact with Git’s features.

## In brief
- **`git history` signing series**: Souma’s v5 follow-up patches address the last technical hurdle (completion test regression) by adding `--gpg-sign`/`--no-gpg-sign` to `t9902-completion.sh`. The series is now unblocked and ready to graduate to `next`.
- **`git filter-branch` bugfix**: Grant Moyer’s v3 patch fixes the `--state-branch` commit mapping inversion regression introduced in Git 2.50.0. The patch is self-contained, well-motivated, and has no remaining loose ends.
- **Precompiled headers**: SZEDER Gábor reported a side effect of precompiled headers: the `__FILE__` macro in `git-compat-util.h` expands to `"./git-compat-util.h"` instead of `"git-compat-util.h"`, affecting error messages in assertions and compiler diagnostics.
- **`fetch.followRemoteHEAD` refactoring**: Colin Hinton’s v4 patch defers validation of the config setting until it is actually used in `do_fetch()`, eliminating spurious warnings for fetches that never consult the setting.
- **`post-worktree` hooks**: Maciej Ciemborowicz provided a real-world use case for Domen Kožar’s proposed `post-worktree` hooks, demonstrating that IDEs (e.g., VS Code with Codex) create worktrees directly, bypassing wrappers and making hooks necessary for observability.
- **Bisect plumbing**: Alejandro Colomar and D. Ben Knoble explored plumbing commands to robustly check whether `git bisect` has isolated a single commit. The discussion highlighted the limitations of existing approaches and suggested `git rev-list --bisect` as a potential solution.
- **Zsh completion**: Fionn contributed a patch to `git-completion.zsh` to exclude already-specified file arguments from further completion candidates, aligning Git’s bundled Zsh completion with Zsh’s native `_git` completion.
- **`BISECT_HEAD` feature request**: Alejandro Colomar requested that `BISECT_HEAD` be set unconditionally, even when `--no-checkout` is not used, to allow referencing the bisect commit after switching branches.