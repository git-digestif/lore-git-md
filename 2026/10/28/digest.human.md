# Git mailing list daily digest for 2026/10/28

## The day in brief

The Git mailing list saw **126 emails** today, with key developments in **submodule traversal**, **rebase autosquash**, **MIDX incremental layers**, and **fetch refspec matching**. Notable threads include **William Hatfield’s `git submodule foreach` enhancements (v2)**, **Farid Zakaria’s `--autosquash` fix (v5)**, **Taylor Blau’s MIDX incremental layer bugfix**, and **Harald Nordgren’s fetch refspec matching series (v12)**. Several **refactoring and cleanup patches** also advanced, including **Patrick Steinhardt’s object storage extension (v4)** and **Junio C Hamano’s dead-code removals**.

---

## Notable threads

### 1. `git submodule foreach` gains dependency-ordered traversal (v2)
**What changed?**
William Hatfield’s **v2 series** (3 patches) enhances `git submodule foreach` with new traversal options:
- `--reverse-traversal`: post-order processing (children before parents)
- `--include-superproject`: execute command in superproject after submodules
- `--reversive`: shorthand combining `--recursive`, `--reverse-traversal`, and `--include-superproject`

### Why it matters

This enables **dependency-sensitive workflows** (e.g., builds/deployments) in complex submodule hierarchies. The series addresses prior feedback on **naming semantics** (`--reverse-traversal` vs `--postorder-traversal`) and **environment handling** (preserves caller’s `GIT_DIR`/`GIT_INDEX_FILE`).

### Current status

- **Implementation finalized** (v2 addresses all technical feedback).
- **Naming discussion ongoing**: Junio C Hamano prefers algorithmic/directional terms.
- **No merge status reported**.

**Files touched**: `builtin/submodule--helper.c`, `git-submodule.sh`, `t/t7427-submodule-foreach-reversive.sh`, `Documentation/git-submodule.txt`.

---

### 2. `--autosquash` fix for empty commits (v5)
**What changed?**
Farid Zakaria’s **v5 patch** fixes a long-standing edge case in `git rebase --autosquash` where `fixup!`/`squash!` commits that cancel out their target (resulting in empty commits) **failed to honor `--empty` options** (`drop`, `keep`, `stop`). The fix introduces:
- `is_amended_head_empty()` to detect when applying a `fixup!` empties the target commit.
- Persistent state (`$GIT_DIR/rebase-merge/fixup-target`) to track whether the target was originally picked with changes.

### Why it matters

This resolves a **real-world workflow issue** at Meta, where internal LLVM patches are managed as single commits using `fixup!` autosquash. The fix enables **automatic dropping of both a commit and its revert** when `--empty=drop` is used.

### Current status

- **Technically approved** (Phillip Wood’s review confirms readiness).
- **Follow-up discussion**: Competing designs for a `revert!` autosquash directive remain unresolved.
- **No merge status reported**.

**Files touched**: `sequencer.c`, `Documentation/git-rebase.adoc`, `t/t3415-rebase-autosquash.sh`, `t/t5407-post-rewrite-hook.sh`.

---

### 3. MIDX incremental layer bugfix (v2)
**What changed?**
Taylor Blau’s **v2 series** (8 patches) fixes a regression where **MIDX writes published empty layers** (zero objects) without a reverse index, causing subsequent `--bitmap` writes to fail. The fix:
- Exits early in `write_midx_internal()` when no objects are found.
- Addresses all MIDX write modes (incremental, non-incremental, compaction).

### Why it matters

This prevents **corruption in large repositories** where MIDX is critical for performance. The series also resolves **exit-status consistency** (silent success vs. error) and **test suite refinements**.

### Current status

- **Ready for integration** (v2 addresses all feedback).
- **Open question**: Should empty MIDX writes return silent success (0) or error?
- **Maintainer query**: Junio asks if the series is still improving or has stalled.

**Files touched**: `midx-write.c`, `t/t5334-incremental-multi-pack-index.sh`, `t5326-multi-pack-bitmaps.sh`.

---

### 4. Fetch refspec matching speedup (v12)
**What changed?**
Harald Nordgren’s **v12 series** (5 patches) introduces `remote.<name>.fetchRefmap` to **decouple "what to fetch" from "where to store"**, resolving scalability issues in shallow repositories. Key features:
- **Dual storage**: Colon-separated `remote.<name>.fetch` entries with `fetchRefmap` store refs in both destinations.
- **Precedence**: `remote.<name>.fetch` > `fetchRefmap` > dynamic branch enumeration.
- **`--limited-fetch` flag**: Configures `fetchRefmap` for shallow repositories.

### Why it matters

This **10–100x speedup** for `git fetch` in shallow repositories (e.g., 2.5s → 0.15s for 100 delete refspecs) by avoiding negotiation for all remote branches.

### Current status

- **Design finalized** (Junio and Phillip Wood endorse v12).
- **Edge case resolved**: Dual storage behavior confirmed for colon-separated `fetch` entries.
- **Documentation clarity**: Remaining work to clarify dual storage and precedence rules.

**Files touched**: `remote.c`, `builtin/fetch.c`, `builtin/remote.c`, `Documentation/`.

---

## In brief

- **`git worktree list --branch`**: Eric Sunshine proposes `--filter=branch:<name>` for extensibility; Junio sketches `--filter-command` for arbitrary logic.
- **`git cat-file --remote-object-info`**: Pablo Sabater’s RFC lifts the 10k OID limit via internal batching (50k-OID chunks).
- **`git rerere` subcommand modernization**: Junio rejects `rerere_perform()` in favor of reverting to `rerere()`.
- **`git init` absolute path regression**: Patch merged to `next` (restores pre-20e38cba3926 behavior).
- **Alternates reloading fix**: Patrick Steinhardt’s series (6 patches) restores dynamic reload of alternates during object lookups.
- **`git bundle-uri` crash fix**: Aniket Gupta’s patch handles relative URIs with excessive `../` components.
- **`git diff-files` refactoring**: Edson’s patch converts manual option parsing to `parse-options` API.
- **Dead-code removals**: Junio removes unused `author_ident_sufficiently_given()`, `verify_uptodate()` (post-recursive-merge cleanup).

---

## Key decisions pending

1. **`git submodule foreach` naming**: Junio to decide between `--reverse-traversal` (current) and `--postorder-traversal`/`--bottom-up-traversal`.
2. **`revert!` autosquash directive**: Consensus needed between empty marker commits (Phillip Wood) and inverse-patch approaches (Yuxuan Chen).
3. **MIDX empty-layer exit status**: Derrick Stolee’s input sought on silent success (0) vs. error.
4. **Fetch refspec documentation**: Harald to clarify dual storage behavior and precedence rules.