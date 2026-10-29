# Git mailing list daily digest for 2026/10/28

## The day in brief

The Git mailing list saw significant activity today across multiple fronts. Key developments included:

1. **Submodule enhancements**: William Hatfield posted v2 of the `git submodule foreach` series with improved traversal options
2. **Rebase improvements**: Farid Zakaria's v5 patch for `--autosquash` behavior with empty commits is ready for merging
3. **Fetch optimizations**: Karthik Nayak's v5 series addressing shallow fetch performance regressions gained momentum
4. **Architectural discussions**: Patrick Steinhardt's object storage extension series received final approval
5. **Bugfixes**: Multiple patches addressing signal handling, commit-graph refactoring, and test modernizations were posted

## Notable threads

### Feature series enhancing `git submodule foreach` (William Hatfield)
The v2 series introduces new traversal options for dependency-ordered operations in submodule hierarchies:
- `--reverse-traversal`: Enables post-order processing (children before parents)
- `--include-superproject`: Runs command in superproject after submodules
- `--reversive`: Shorthand combining `--recursive`, `--reverse-traversal`, and `--include-superproject`

The implementation now integrates tests with corresponding patches and addresses naming feedback. The series enables workflows like dependency-sensitive builds in complex submodule hierarchies while maintaining careful environment handling for superproject execution.

### Fix for `--autosquash` behavior with empty commits (Farid Zakaria)
The v5 patch addresses a long-standing edge case where `fixup!`/`squash!` commits that cancel out their target commit weren't handled according to the `--empty` option. The fix:
- Introduces `is_amended_head_empty()` to detect when applying a fixup empties the target
- Honors `--empty` for such commits while preserving existing behavior for commits picked empty
- Handles edge cases like mid-chain fixups and conflict resolution
- Adds comprehensive test coverage for all `--empty` modes

The patch is motivated by real-world workflows at Meta and enables automatic dropping of both a commit and its revert when `--empty=drop` is used.

### Shallow fetch performance regression fix (Karthik Nayak)
The v5 series fixes a performance regression where batched reference updates delayed committing references until after tag backfilling, causing the client to report no 'have' references during negotiation. Key improvements:
- Exposes pending transaction state via `negotiation_include_tips`
- Adds `transport_options_release()` to fix memory management
- Restores correct 'have' reporting while preserving atomicity
- Achieves 17.38× speedup over pre-batched-update behavior

The series is ready for maintainer review after addressing all technical feedback.

### Object storage extension series (Patrick Steinhardt)
The v4 series introducing `extensions.objectStorage` received final approval from Karthik Nayak. The series:
- Enables pluggable object database backends
- Implements baseline `files` backend with directory relocation support
- Adds environment variable and configuration support
- Includes inspection tools and submodule propagation

The architectural debate about exposing this as a public API remains unresolved, but the implementation is technically sound and ready for integration.

### Signal handler chain repair (Ian Hu)
The discussion advanced toward a thread-level fix for signal handler concurrency issues. Phillip Wood endorsed:
- Blocking signals during thread creation
- Using `sigaction()` to mask signals during handler execution
- Dropping the current "repair-after-corruption" patch

This approach would prevent both chain corruption and concurrent handler execution.

## In brief

- **MIDX empty layer fix**: Taylor Blau declared the v2 series ready for integration despite some remaining confusion
- **Fetch refspec matching**: Harald Nordgren's v12 series for matching-inspired fetch mode is nearing completion
- **Repository error reporting**: Kaartic Sivaraam's v3 series improves error messages for invalid repositories
- **Beginner tutorial**: Julia Evans's v4 tutorial replacement received final feedback
- **Refspec matching speedup**: Jon Simons's v2 series achieves 10-100× speedups for many deletions
- **Loose-ref iterator fix**: Venkat Ragavan's patch addressing shared-prefix mispositioning awaits maintainer input
- **Commit-graph refactoring**: Patrick Steinhardt's series to prepare for pluggable backends received mixed feedback
- **Bundle URI handling**: Aniket Gupta's patch prevents crashes with malformed relative URIs
- **Test modernizations**: Multiple patches converting legacy test idioms to modern helpers
- **Dead code removal**: Junio C Hamano posted several cleanup patches removing unused functions

The day's activity reflects ongoing efforts to improve Git's performance, reliability, and extensibility while maintaining backward compatibility. Several series are now ready for integration, pending maintainer review.