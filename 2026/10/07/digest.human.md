# Git mailing list daily digest for 2026/10/07

## The day in brief

The Git mailing list saw significant activity around security features, performance optimizations, and policy discussions. Key developments include a CI timeout issue with the `http.sslVerifyStatus` feature, test breakages in the ref transaction hook series, a policy debate on AI-assisted contributions, and performance improvements for packed-refs and commit-graph operations.

## Notable threads

### CI timeout with `http.sslVerifyStatus` feature
**What changed?**
Junio C Hamano reported that the `http.sslVerifyStatus` patch, though reverted from `next` due to test failures, may be interacting badly with other topics in the `seen` integration branch. CI jobs on older Ubuntu runners are spinning indefinitely and timing out after six hours.

### Why it matters

The `http.sslVerifyStatus` feature adds OCSP staple validation to Git's HTTPS certificate verification, motivated by government customers who mandate OCSP stapling. The timeout issue suggests a deeper interaction problem that could affect integration stability.

### Technical details

- The patch adds a boolean `http.sslVerifyStatus` option (default `false`) that enables staple validation via libcurl's `CURLOPT_SSL_VERIFYSTATUS`.
- The CI timeout occurs only when the patch is present, suggesting a side effect of combining it with other in-flight topics.
- No new technical objections to the feature itself have been raised.

### Next steps

A targeted CI experiment with the patch alone is needed to determine whether the timeout is reproducible in isolation.

---

### Test breakages in ref transaction hook series
**What changed?**
Junio C Hamano identified test breakages in Maciej Ciemborowicz's v3 patch series that fixes the `reference-transaction` hook omission during branch rename/copy operations. The series now fails `t0600.16` and `t5510.35`, indicating regressions in packed-refs locking behavior or transaction rollback logic.

### Why it matters

The series aims to ensure the `reference-transaction` hook sees both the deletion of the old ref and the creation of the new one in a single transaction. The breakages suggest unintended side effects that could compromise ref safety.

### Technical details

- The series consists of four patches: internal hook suppression, reflog replacement support, copy/rename integration, and removal of backend-specific callbacks.
- The failures occur in unrelated commands (`update-ref -d`, `fetch --prune`), suggesting broader unintended side effects.
- The breakages are likely due to changes in `ref_transaction_commit()` or the removal of backend-specific callbacks.

### Next steps

The series requires a v4 to address the test breakages without reintroducing the original hook omission bug.

---

### Policy debate on AI-assisted contributions
**What changed?**
Scott Chacon proposed amending `Documentation/SubmittingPatches` to explicitly permit AI-assisted contributions under clear conditions, sparking a debate about enforceability, legal risks, and reviewer bandwidth.

### Why it matters

The proposal aims to replace Git's current ambiguous stance ("we may reject anything that looks AI-generated") with a predictable framework. However, legal and ethical concerns about AI-generated code's provenance and license compliance have emerged as significant blockers.

### Technical details

- The patch introduces an `Assisted-by:` trailer for substantial AI assistance, requiring contributors to review, test, and sign off on submissions.
- No code or build system changes are involved; the change is purely documentation.
- The proposal cites Linux, Xen, and Debian as precedents.

### Key positions

- **Scott Chacon**: Proposes the change, arguing that responsible AI use should be allowed under the same review standards as human-written patches.
- **brian m. carlson**: Opposes the change on legal and ethical grounds, citing risks of license violations and copyright infringement from AI-generated code.
- **Junio C Hamano**: Questions enforceability and reviewer bandwidth, noting that the current policy is more direct and easier to apply.

### Next steps

The thread remains unresolved, with no clear consensus on how to balance the benefits of AI assistance against the legal and quality risks.

---

### Performance optimization for packed-refs backend
**What changed?**
Karthik Nayak posted v4 of a performance optimization patch that speeds up operations that rewrite the packed-refs file (e.g., deleting references) by avoiding redundant formatting of unchanged refs.

### Why it matters

The patch addresses a measurable performance regression in repositories with many refs, reducing `fetch/write-commit-graph` time by ~20% in synthetic benchmarks.

### Technical details

- The patch tracks raw byte positions of unchanged refs and writes them directly with `fwrite()`, bypassing `fprintf()`.
- It introduces a `record_start` field in `struct packed_ref_iterator` to mark the start of each ref record.
- The optimization is confined to `refs/packed-backend.c` and has no on-disk format changes.

### Status

The patch has received explicit approval from Patrick Steinhardt and is ready for maintainer pickup.

---

## In brief

- **`git blame` ignore-revs default file**: Junio C Hamano clarified that his feedback on Ravi Mistry's security refinements was intended to delegate the security audit to others, not to endorse a specific plan. The v2 patch is pending.
- **Matching-inspired fetch mode for shallow repositories**: Harald Nordgren posted v7 of a series introducing `remote.<name>.refmap` and dynamic branch selection logic, resolving the fatal error and design ambiguity from v6.
- **Packfile corruption fix**: Junio C Hamano marked Patrick Steinhardt's series for inclusion in `next`, confirming the fix for stale delta base cache entries.
- **Push negotiation bug**: Patrick Steinhardt proposed a fix for a bug where a pre-push hook repacking the local object database causes redundant transfer of common history.
- **Test infrastructure cleanup**: Muhammed Dilshad A posted a three-patch series migrating mergesort tests to Clar and retiring unused subcommands, endorsed by Junio C Hamano.
- **`--relative` with combined diffs**: Muhammed Dilshad A posted v2 of a series fixing `--relative` handling in combined diffs, addressing Junio C Hamano's feedback on path normalization and rename handling.
- **Windows performance regression**: Daniel Gullberg reported a performance regression in `safe.directory` ownership checks with unreachable UNC paths, causing 25-second stalls on Windows.
- **Detached HEAD documentation**: Julia Evans posted a patch rewriting the explanation of detached HEAD state, with feedback from Junio C Hamano and Kristoffer Haugsbakk focusing on clarity and build-system omissions.