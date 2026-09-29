# Git mailing list daily digest for 2026/09/28

## The day in brief
The Git project saw **135 emails** today, with key developments including: a **Rust-based SHA-1 backend** proposed for performance gains; **reftable timezone encoding** fixed to match specification; **stash autostash bugfix** series facing integration challenges; **CI improvements** for leak sanitizer reporting; and **documentation updates** including a new `gitbreaking-changes(7)` manpage. The **Git 2.56.0 release** was announced, and **Git for Windows 2.56.0** followed shortly after.

## Notable threads

### Rust-based SHA-1 backend (3× performance gain)
**What changed?**
Johannes Schindelin proposed a four-patch series introducing an optional Rust-based SHA-1 backend (`sha1dc` crate) that claims **3× performance improvement** for `git index-pack` on Windows and large repositories. The series includes:
- Patch 1: Adds the Rust crate as a compile-time option
- Patch 2: Implements a runtime escape hatch (`core.sha1dcBackend=c`)
- Patch 3: Provides `pthread_once()` shims for Windows/NO_PTHREADS
- Patch 4: Makes `sha1dc_init()` thread-safe

**Why it matters**
This represents a significant step in Git's **Rustification effort**, offering concrete performance benefits while maintaining backward compatibility. The optional nature and runtime escape hatch address concerns about mandatory Rust dependencies.

**Current status**
Under review; no objections yet but the Rust version requirement (1.87 vs Git's 1.63 minimum) may need justification.

---

### Reftable timezone encoding fix
**What changed?**
Josh McKinney reported a discrepancy where Git's reftable backend writes timezone offsets as signed HHMM (e.g., +05:30 → 0530) but the specification requires signed minutes (e.g., +05:30 → 330). This breaks interoperability with JGit.

**Why it matters**
The reftable backend is critical for Git's future scalability, and this fix ensures compatibility with other implementations.

**Current status**
Consensus reached to align Git with the specification. No patch yet, but the fix is straightforward and expected soon.

---

### Stash autostash bugfix series integration challenges
**What changed?**
D. Ben Knoble's five-patch series fixing `git stash` autostash with staged index entries faced two integration blockers:
1. A **NULL-dereference risk** in the in-core merge logic (identified by Junio)
2. A **bisectability regression** in the test suite (identified by Phillip Wood)

**Why it matters**
The series fixes a real bug where autostashing fails to correctly handle staged index entries, but the integration issues highlight the complexity of modifying core Git functionality.

**Current status**
Author confirmed both issues and will address them. The series was ejected from `seen` pending fixes.

---

### CI improvements for leak sanitizer reporting
**What changed?**
Harald Nordgren posted a v2 series improving GitHub Actions CI failure reporting:
- Patch 1: Stops leak-sanitizer scripts at first failure and annotates leaks
- Patch 2: Points regular test failures to their exact file/line

**Why it matters**
These changes make CI failures more actionable by providing better context and reducing noise.

**Current status**
Under review; Junio identified a correctness issue in the annotation logic that needs addressing.

---

### Documentation updates
**New `gitbreaking-changes(7)` manpage**
Julia Evans proposed a four-patch RFC converting the `BreakingChanges` document into a proper manpage for better discoverability.

**Git tutorial rewrite**
Julia Evans proposed a two-phase approach to rewrite Git's tutorial: first remove the outdated `gittutorial-2`, then add a new simplified version.

**Why it matters**
These changes improve Git's documentation ecosystem, making it more accessible to users.

**Current status**
Under discussion; no objections to the approaches.

## In brief
- **Git 2.56.0 released**: Junio announced the release with 748 commits from 104 contributors
- **Git for Windows 2.56.0 released**: Johannes Schindelin announced the Windows release
- **`git reflog expire` regression fix merged**: The fix for swapped default expiry times was merged to master
- **`git replay` signing series blocked**: Patrick Monette's series adding GPG signing to `git replay` is blocked on Tian Yuchen's `git history` signing work
- **CI resource exhaustion fixes**: Tamir Duberstein's series reducing CI resource exhaustion was marked "Will merge to next?"
- **`git remote` documentation clarification**: Junio suggested a broader approach to clarify that all `git remote` commands operate locally
- **`includeIf "hostname:..."` proposed**: Isabella Caselli proposed adding hostname-based configuration inclusion for multi-machine setups
- **`git(1)` manpage rewrite**: Julia Evans proposed rewriting the intro to better orient users toward the help system
- **`gittutorial-2` removal**: Julia Evans proposed removing the outdated tutorial in a three-patch series