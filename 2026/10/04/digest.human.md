# Git mailing list daily digest for 2026/10/04

## Long threads

### format-patch range-diff notes options

*Long thread: 25 replies so far from 4 participants over 42 days; 4 messages that day.*

Kristoffer Haugsbakk's series teaches `git format-patch` the `--[no-]range-diff-notes` options so the range diff can show a different notes ref, or none, than the patches themselves. Earlier revisions settled on `--no-range-diff-notes` meaning no range-diff notes at all, implemented via a parse-options callback. Today he posted v4, trimming expository text, refactoring `rdiff_notes_cb` to call `parse_opt_string_list`, and shortening the documentation. Junio C Hamano reviewed v4 2/2, proposed a shorter replacement for the "why not error out like `--creation-factor`" paragraph, and said the patch otherwise looks good. Kristoffer agreed and posted v5, whose patch 2/2 adopts the shorter wording; patch 1/2 is unchanged.

### shallow-repo fetch of new remotes via refmap

*Long thread: 23 replies so far from 4 participants over 16 days; 3 messages that day.*

Harald Nordgren's four-patch series avoids fetching every branch of a new remote in a shallow repository: it adds `remote.<name>.refmap` as a per-remote default for `--refmap`, infers branches to fetch from local upstreams, adds `git remote add --limited-fetch`, and defaults to that in shallow repositories. Earlier revisions removed the automatic default-branch fetch. Today he posted v6, whose only change removes a leftover reference to the deleted default-branch logic in patch 3's commit message. Junio C Hamano reviewed v6, calling it much better, but flagged that with only patch 1/4 applied, a remote with `remote.origin.refmap` and no tracked branches makes `git fetch origin` die with "fatal: --refmap option is only meaningful with command-line refspec", and raised the unresolved question of what should happen when both `remote.origin.fetch` and `remote.origin.refmap` are configured. Harald answered by proposing that `remote.<name>.fetch` should win.

### CI annotations linked to test scripts

*Long thread: 20 replies so far from 4 participants over 10 days; 2 messages that day.*

Harald Nordgren's two-patch series links GitHub Actions failure and leak annotations to the test script's file and line, and stops leak-sanitizer jobs at the first failure; earlier revisions removed percent-escaping after CI verification. Today Harald asked whether the inability to link annotations to test files unchanged in a PR is enough to call the change a regression, suggesting that part might not be worth doing. He objected to the tone of Phillip Wood's earlier message, acknowledged he should clarify that the change affects the summary page, and proposed deciding on the regression question before progressing, noting the topic's point was to make leak reporting less bad with everything else a bonus.

## The day in brief

Kristoffer Haugsbakk posted v5 of the `format-patch --[no-]range-diff-notes` series with Junio C Hamano's condensed wording, while Harald Nordgren posted v6 of the shallow-repo refmap fetch series and answered an open design question about `fetch` versus `refmap`. Coy Geek reported that `repack --drop-filtered --dry-run` on Git 2.56.0 writes new pack files, and with explicit `-d` deletes existing ones, despite the documented no-write promise.

## Notable threads

### repack --dry-run writes packs bug report
Coy Geek reported that on Git 2.56.0, `git repack -a --filter=blob:limit=1m --drop-filtered --dry-run` writes four new files (`.pack`, `.idx`, `.promisor`, `.rev`) to the pack directory and, with explicit `-d`, deletes 12 existing pack and sidecar files, contradicting the manual's promise that `--dry-run` lists candidates without rebuilding or deleting; both variants exit 0 and print the candidate blob ID.

### fsmonitor untracked cache after a trivial response
Ilia K posted a patch adding `untracked_cache_invalidate_stale_dirs()` so that, after the fsmonitor daemon sends a trivial response, untracked-cache directories whose stat data changed or disappeared are invalidated before the index is saved next to the new token, preventing later `git status` runs from trusting stale valid bits.

### post-worktree hook with prune notification
Domen Kožar posted v3 of the series adding a single `post-worktree` lifecycle hook for add, move, and remove, invoked with the event, worktree identifier, and old and new absolute paths. Patch 2/2 extends it to pruning: each pruned entry emits a remove event (skipped during dry runs), `should_prune_worktree()` now returns the recorded `.git` path even when it points to a missing location, and hook failures are reflected in the exit status while remaining entries are still processed.

## In brief
- Phillip Wood reviewed Harald Nordgren's `--delete-merged` squash-merge detection, endorsed the `git cherry` patch-id algorithm as the model, and cautioned that rebased branches may evade detection; Harald agreed the per-branch upstream walk can be improved and said he is iterating on shared logic.
- Colin Hinton posted v5 deferring `fetch.followRemoteHEAD` validation, storing the raw string with a `follow_remote_head_seen` flag and validating lazily in `do_fetch()` only when consulted, after Junio C Hamano endorsed dying rather than warning on bad values.
- Junio C Hamano marked Kristoffer Haugsbakk's `doc: interpret-trailers: fix cmd examples` topic for `next`.
