# Git mailing list daily digest for 2026/09/14

## The day in brief
The Git mailing list saw significant activity around several key areas: a new incremental connectivity check for fetch/push operations, a critical bugfix for concurrent repack operations, and ongoing refinements to the `git var` extension and advice system. Outreachy mentor assignments were finalized, and several documentation and test modernization efforts progressed. The maintainer's "What's cooking" report provided a comprehensive overview of integration status for Git 2.56-rc0.

## Notable threads

### Incremental connectivity check for fetch/push
**[PATCH v1 0/2] Add incremental connectivity check for fetch/push** by Kristofer Karlsson

Kristofer Karlsson introduced an opt-in incremental connectivity check (`transfer.connectivityCheck=incremental`) to reduce the cost of verifying object connectivity during fetches and pushes. The new mode processes incoming commits in topological order, tracking trusted objects from parent commits to skip unchanged subtrees. This shifts the cost from being proportional to repository size to being proportional to incoming change size.

The implementation adds a new `tree-verify.c` file and updates `builtin/rev-list.c` and `connected.c`. Benchmark results show dramatic speedups for small pushes to large repositories (up to 190x faster) with significant memory savings. Junio C Hamano raised concerns about promisor object handling and NULL-dereference risks, which the author plans to address in a follow-up version.

**Why it matters**: This feature addresses a significant performance bottleneck in large repositories, particularly for CI/CD pipelines and development workflows that involve frequent small pushes to large repositories.

### Fix concurrent-push race leading to data loss
**[PATCH 0/6] repack: fix concurrent-push race leading to data loss** by qeesung

Qin ShiCheng (qeesung) posted a critical bugfix series addressing a race condition in `git repack -d` where concurrent pushes can cause the repack machinery to delete packs whose objects were never copied elsewhere, resulting in data loss. The series ensures `.keep` files are only removed by the process that created them and replaces the `--honor-pack-keep` mechanism with a consistent snapshot of kept packs observed at repack startup.

The series includes six patches that:
1. Prevent `receive-pack` from removing `.keep` files it didn't create
2. Fix traversal logic in `pack-objects`
3. Fix a latent bug in the cruft-pack walk
4. Optimize `--keep-pack` name lookups
5. Add `--keep-pack-from-file` to handle repositories with many kept packs
6. Pass `repack`'s snapshot of `.keep` packs to `pack-objects`

**Why it matters**: This is a production-tested fix for a subtle but serious data-loss bug that affects repositories with concurrent push operations.

### Git var extension series reaches v8
**[PATCH v8 0/4] Extend git var to provide unified identity and signing configuration** by Andrew Pleeter

Andrew Pleeter posted v8 of the `git var` extension series, which provides a unified, scriptable interface to Git's identity and signing configuration. The v8 patch addresses Junio's v7 feedback by adopting `VARIABLE=value` output format and aligning exit code behavior with the implementation.

The series extends `git var` to support:
- Identity components (name/email/date for author and committer)
- Signing keys (`GIT_SIGNING_KEY`)
- Multiple variable arguments
- NUL-terminated output with `-z`

**Why it matters**: This extension provides a consistent interface for scripts and tools to access Git's configuration, particularly for signing operations, which is increasingly important for secure development workflows.

### Outreachy December 2026 cohort mentor assignments complete
**Git participation in Outreachy December 2026 cohort** by Kaartic Sivaraam

Kaartic Sivaraam reported that mentor assignments for both Outreachy projects are now complete:
- "Improve how command arguments and options are scanned and parsed" (co-mentors: Christian Couder, Siddharth Asthana)
- "Reduce Git's global state to enable Git's libification" (co-mentors: Christian Couder, Pablo Sabater)

Additionally, Kaartic proposed expanding the application to include two promisor-remote projects originally prepared for GSoC 2026.

**Why it matters**: Outreachy provides valuable internship opportunities for underrepresented groups in open source, and Git's participation helps grow the contributor base and advance key technical goals.

### What's cooking in git.git (Sep 2026)
**What's cooking in git.git (Sep 2026)** by Junio C Hamano

The maintainer's "What's cooking" report provided a comprehensive overview of integration status for Git 2.56-rc0. Key highlights include:
- Rust integration topics in `next` (js/rust-in-windows-ci, jc/rust-cargo-build-target)
- ODB abstraction work (ps/odb-stop-registering-in-memory-sources, ps/odb-alternates-at-creation, ps/odb-pluggable-fsck)
- Ref storage format standardization (ps/ref-storage-format)
- New experimental commands like `git history squash` (hn/history-squash)
- Performance optimizations like Bloom filter usage in `git last-modified` (tc/last-modified-bloom)

**Why it matters**: This report provides visibility into what features and improvements are likely to land in the next release, helping contributors and users plan accordingly.

## In brief

- **[PATCH v4 0/2] rerere: fix race condition between rebase and background maintenance** by Thomas Bachem: Posted v4 of the rerere lock race series, splitting into two patches as requested by reviewers.
- **[PATCH v4 0/2] dir: fix common prefix calculation with leading exclude pathspec** by Yannik Tausch: Posted v4 of the pathspec exclusion series, reverting to a two-patch structure and adding deterministic test cases.
- **[PATCH v4 0/2] push: --force-if-includes fixes** by Tyler Cipriani: Posted v4 addressing backwards-compatibility concerns about rejecting non-branch pushes when `--force-if-includes` is enabled.
- **[PATCH v5] advice: add scope hint for disabling advice messages** by Junio C Hamano: Posted v5 implementing the uniform `--global` hint for all `advice.*` settings.
- **[PATCH 0/3] imap-send: OpenSSL compatibility and correctness fixes** by Beat Bolli: Clarified the OpenSSL 4.1 compatibility macro as a forward-compatibility fix to avoid build failures under `DEVELOPER=1`.
- **[PATCH v2] completion: add support for `git worktree repair`** by Yoichi NAKAYAMA: Posted v2 with adjusted commit message for the worktree repair completion patch.
- **[PATCH v2 0/2] doc: lint and fix backtick quoting in git-refs and git-pack-refs** by Todd Zullinger: Posted v2 addressing Junio's correction to restrict backtick-quoting in `pack-refs-options.adoc` to configuration keys only.
- **[PATCH] t7610-mergetool.sh: modernize test helpers** by Tanishq Singh: Junio identified a logical error in the test modernization patch, requiring a v2.
- **Bug report: `git status --ignored` with pathspec substring matching** by Sean Whitton: Reported a bug where `git status --ignored` with a pathspec incorrectly matches ignored directories whose names contain the pathspec as a substring.