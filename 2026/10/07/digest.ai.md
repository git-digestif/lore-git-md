# Git mailing list daily digest for 2026/10/07

## The day in brief
The Git mailing list saw significant activity around security features, performance optimizations, and policy discussions. Key developments include a CI timeout issue with the `http.sslVerifyStatus` patch, test breakages in the ref-transaction hook series, progress on SHA-256 interoperability work, and a contentious debate about allowing AI-assisted contributions. Performance optimizations for packed-refs and fetch commit-graph writing also advanced, while documentation improvements for merge conflicts and detached HEAD state were proposed.

## Notable threads

### http.sslVerifyStatus CI timeout investigation
**What changed?**
Junio C Hamano reported that the `http.sslVerifyStatus` patch, though reverted from `next` due to test failures, may be interacting badly with other topics in the `seen` integration branch. CI jobs on older Ubuntu runners are spinning indefinitely and timing out after six hours when the patch is present.

**Why it matters**
The patch adds OCSP staple validation to Git's HTTPS certificate verification, a security feature motivated by government customers who mandate OCSP stapling. The CI timeout issue threatens to delay this security improvement, which is already technically complete and addresses all prior review feedback.

**Technical details**
- The patch adds a boolean `http.sslVerifyStatus` option (default `false`) that enables staple validation via libcurl's `CURLOPT_SSL_VERIFYSTATUS`
- The timeout occurs only in `seen`, not when the patch is tested in isolation
- Root cause is unclear—whether it's a direct interaction with the OCSP test harness, a latent bug in the patch, or a side effect of combining it with other in-flight topics

**Next steps**
A targeted CI experiment with the patch alone is needed to determine whether the timeout is reproducible in isolation.

---

### Ref-transaction hook test breakages
**What changed?**
Junio C Hamano identified test breakages in Maciej Ciemborowicz's v3 patch series that fixes the `reference-transaction` hook omission during branch rename/copy operations. The series now fails `t0600.16` and `t5510.35`, indicating regressions in packed-refs locking behavior or transaction rollback logic.

**Why it matters**
The series addresses a long-standing bug where the `reference-transaction` hook misses the creation of new refs during branch rename (`git branch -m`) and copy (`git branch -c`) operations. The test breakages suggest the refactoring in patches 3/4 or 4/4 may have unintended side effects on core ref-deletion safety mechanisms.

**Technical details**
- The series consists of four focused patches: internal hook suppression, reflog replacement support, copy/rename integration, and removal of backend-specific callbacks
- The breakages occur in unrelated commands (`update-ref -d`, `fetch --prune`), suggesting broader unintended side effects
- Performance regressions are acknowledged (~137% slowdown for files-based rename with large reflogs) but deemed acceptable

**Next steps**
The series requires a v4 to address the test breakages without reintroducing the original hook omission bug.

---

### SHA-256 interoperability work progress
**What changed?**
Christian Couder volunteered to help upstream early patches from the `sha256-interop-part-2` branch within the next one to two weeks. Brian m. carlson confirmed this would unblock critical progress by enabling pack index v3 and object map support.

**Why it matters**
The interoperability work is essential for Git's planned SHA-256 transition in Git 3.0 (March 2027). The current lack of upstreamed interoperability code is a major blocker for users testing the SHA-256 transition.

**Technical details**
- The interoperability work enables rewriting repositories between hash algorithms during clone/fetch
- Key gaps include missing in-place migration tooling (critical for submodules) and unoptimized performance
- The work is publicly available in carlson's GitHub fork but remains experimental and incomplete

**Next steps**
Christian's offer to upstream early patches is a concrete step toward addressing the upstreaming bottleneck, though the full series remains constrained by contributor availability.

---

### AI-assisted contributions policy debate
**What changed?**
Scott Chacon proposed amending `Documentation/SubmittingPatches` to explicitly permit AI-assisted contributions under clear conditions. The proposal sparked a contentious debate about legal risks, enforceability, and the practicality of certifying AI-generated output under the Developer Certificate of Origin (DCO).

**Why it matters**
The current policy creates uncertainty for contributors who produce valuable, reviewed patches using AI assistance. The debate touches on broader questions about open-source sustainability, legal risks, and the role of AI in software development.

**Technical details**
- Proposed requirements: human review, DCO compliance, `Assisted-by:` trailer for substantial AI help
- Legal concerns: LLMs may reproduce training-set material from sources that do not permit copying, modification, or distribution
- Enforceability concerns: reviewers may be overwhelmed with plausible-sounding but low-quality submissions

**Positions**
- **Scott Chacon**: Proposes the change, arguing that responsible AI use should be allowed under the same review standards as human-written patches
- **brian m. carlson**: Opposes the change on legal and ethical grounds, citing risks of copyright infringement and license violations
- **Junio C Hamano**: Questions enforceability and reviewer bandwidth, noting the current policy is more direct and easier to apply

**Next steps**
The thread remains unresolved, with no clear consensus on whether to proceed with the policy change.

---

### Packed-refs performance optimization
**What changed?**
Karthik Nayak posted v4 of a performance optimization patch for the packed-refs backend, which speeds up operations that rewrite the packed-refs file by avoiding redundant formatting of unchanged refs. Patrick Steinhardt explicitly approved the patch, calling the ~20% speedup significant.

**Why it matters**
Operations like deleting references can be significantly faster in repositories with many refs, improving Git's performance for large-scale deployments.

**Technical details**
- The patch tracks raw byte positions of unchanged refs and writes them directly with `fwrite()`
- Performance improvement: deleting 100,000 packed refs now takes ~23.8 ms, down from ~28.7 ms (~20% faster)
- Side effects: preserves original formatting (case, whitespace, malformed refnames), which may affect tools parsing packed-refs files

**Next steps**
The patch is ready for maintainer pickup and appears uncontroversial.

---

## In brief
- **CI/build system**: Junio C Hamano proposed removing two CI jobs (`linux32` and `linux-TEST-vars`) and updating the remaining `linux-TEST-vars` job to use `ubuntu:rolling`
- **git blame**: Ravi Mistry confirmed the path forward for the v2 patch that makes `git blame` automatically look for `.git-blame-ignore-revs` if no ignore file is configured
- **Matching-inspired fetch mode**: Harald Nordgren posted v7 of a series introducing a "matching-inspired" fetch mode for shallow repositories, resolving the fatal error and design ambiguity from v6
- **Merge conflict documentation**: Julia Evans continued refining the `gitmergeconflicts(7)` guide, incorporating feedback on terminology, tooling advice, and foundational explanations
- **Stash sharing RFC**: Johannes Sixt demonstrated how the proposed stash-sharing use case can be achieved using existing Git commands, effectively neutralizing the core motivation for the feature
- **Packfile corruption fix**: Junio C Hamano marked the series fixing packfile corruption from stale delta base cache entries for inclusion in `next`
- **Push negotiation bug**: Patrick Steinhardt proposed a fix for a push negotiation bug that causes redundant transfer of common history when a pre-push hook repacks the local object database
- **Test infrastructure cleanup**: Muhammed Dilshad A posted a three-patch series migrating mergesort tests to Clar and retiring unused subcommands
- **Combined diff --relative fix**: Muhammed Dilshad A posted v2 of a series fixing `--relative` handling in combined diffs, addressing all concerns from Junio C Hamano's initial review
- **Non-intrusive clone RFC**: The thread pivoted to addressing archive-based threats after D. Ben Knoble corrected the threat model, but no alternative solution has been proposed
- **Windows performance regression**: Daniel Gullberg reported a performance regression in the `safe.directory` ownership check with unreachable UNC paths
- **Detached HEAD documentation**: Julia Evans posted a patch rewriting the explanation of detached HEAD state, with feedback from Junio C Hamano and Kristoffer Haugsbakk
- **What's cooking**: Junio C Hamano's report highlighted 25+ topics in `next` or `seen`, with active work in refs, ODB, tests, and CI subsystems