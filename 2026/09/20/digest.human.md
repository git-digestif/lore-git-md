# Git mailing list daily digest for 2026/09/20

## The day in brief
Maciej Ciemborowicz advanced two critical ref backend fixes, submitting a v2 series for the `reference-transaction` hook regression and a new patch for branch rename visibility. Harald Nordgren proposed a usability improvement for shallow clones, while a new RFC for worktree lifecycle hooks sparked discussion about prior art.

## Notable threads

### GitLab CI Windows breakage fix validated
Johannes Schindelin reported that the v1 patch series fixing GitLab CI breakage after Rust was enabled for Windows passed validation. The `build:mingw64` job succeeded as intended, though Schindelin noted the job name may be outdated (`ucrt64` would be more accurate). An unrelated timeout in the `build:msvc-meson` job—due to credential handling—was observed but does not affect the core fix. Karthik Nayak’s internal GitLab MR (#671) and pipeline (#2863888081) remain the key validation artifacts.

The series restores CI pipeline functionality for Windows builds by ensuring the correct Rust toolchain is provisioned, preserving `.git/info/exclude` contents, and making `cargo` accessible after the minimal SDK’s login profile resets `PATH`. No new technical issues were raised about the Rust toolchain, GNU linker availability, or `.git/info/exclude` preservation.

### `reference-transaction` hook now sees branch renames
Maciej Ciemborowicz submitted a patch to fix the long-standing issue where the `reference-transaction` hook omitted the new ref when a branch was renamed (`git branch -m`). The patch introduces a new transaction type (`REF_TRANSACTION_TYPE_RENAME`/`COPY`) and a `REF_TRANSACTION_FLAG_SKIP_HOOK` flag to suppress hooks for nested operations. It ensures both the deletion of the old ref and the creation of the new ref are reported as a single transaction, addressing the inconsistency between `git branch -m` and `git update-ref --stdin`.

Test coverage is thorough, including rename, copy, forced updates, D/F conflicts, concurrent updates, and hook-triggered rollback. The fix touches both the files and reftable backends, preserving backend-specific reflog handling while ensuring hooks see a unified logical transaction. This addresses a real pain point for tools relying on the `reference-transaction` hook, such as Gerrit and GitLab.

### `reference-transaction` hook regression fix advances to v2
Maciej Ciemborowicz posted a v2 series restoring the correct behavior of the `reference-transaction` hook when branches, tags, or remote refs are deleted. The series addresses a regression introduced in Git 2.31, where the hook received all-zero OIDs instead of the resolved old OID. The v2 patches explicitly document the intentional trade-off: conditional deletion (compare-and-delete) restores the pre-regression safety check, preventing concurrent updates from being silently overwritten.

The first patch modifies `refs_delete_refs()` to accept an optional `oid_array` of old OIDs, making deletions conditional if an OID is provided. The second and third patches update `git branch`, `git tag`, `git fetch --prune`, and `git remote prune` to pass the resolved old OIDs, ensuring the hook receives meaningful values without additional ref reads. The series includes comprehensive test coverage for both files and reftable backends, SHA-1/SHA-256, broken/symbolic refs, and atomic fetch pruning, with new regression tests for concurrent updates and UI consistency.

Karthik Nayak’s earlier review raised a substantive concern about a race condition in the first patch, where the ref’s value could change between resolution and transaction execution. Maciej clarified that the failure mode is intentional—it prevents concurrent updates from being silently overwritten—and provided a reproduction case. The v2 series frames this as a deliberate choice to restore the pre-regression safety check, leaving the project’s acceptance of this trade-off as the remaining open question.

### Shallow clone advice improves usability
Harald Nordgren submitted a patch adding a new advice message (`advice.shallowHistory`) to explain why `<ref>~N` or `<ref>^N` fails in a shallow clone. The advice suggests the exact `git fetch --deepen=<n>` command needed to resolve the issue, firing only when the walk stops at a recorded shallow boundary. The implementation is context-specific: for `<ref>~N`, it suggests the exact `--deepen` value needed, accounting for any history already present; for `<ref>^N`, it suggests `--deepen=1`, as a shallow boundary commit has no parents recorded locally.

D. Ben Knoble reviewed the patch, noting an inconsistency in the messaging: the commit message and documentation use "ref" and "commit-ish" interchangeably, which could confuse users. The advice is documented as applying to `~N` and `^N` syntax, but the commit message implies it only triggers for refs. Knoble also suggested using `<rev>~<n>` and `<rev>^[<n>]` in documentation to match `gitrevisions(7)`. The core logic—detecting shallow boundaries and suggesting context-aware `git fetch --deepen` commands—remains uncontested.

## In brief
- **Worktree lifecycle hooks proposed**: Maciej Ciemborowicz suggested adding native Git hooks for worktree operations (`add`, `remove`, `move`, `lock`, `unlock`, `prune`). Two design options were presented: separate post-operation hooks or a unified `worktree-lifecycle` hook. Kristoffer Haugsbakk linked to a prior discussion from 2024, signaling that this is not a novel idea. The proposal avoids veto hooks and focuses on post-operation events, aligning with Git’s existing `reference-transaction` pattern.