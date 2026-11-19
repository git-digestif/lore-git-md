# Git mailing list daily digest for 2026/10/07

## Long threads

### New gitmergeconflicts(7) manual page

*Long thread: 39 replies so far from 6 participants over 14 days; 1 message that day.*

Julia Evans's documentation series adds a `gitmergeconflicts` manual page explaining merge conflicts with examples, linked from `git-merge`, `git-rebase`, `git-cherry-pick`, `git-revert` and `git-pull` instead of re-explaining the process in each command. Earlier revisions refined the "WHAT IS A MERGE CONFLICT?" and "OURS"/"THEIRS" sections under review by Junio C Hamano and D. Ben Knoble. Today Junio C Hamano corrected that a custom `conflict-marker-size` ensures payload lines that happen to look like markers are not mistaken for them, said the patch will be squashed into the step introducing the new file, and suggested the technique might be worth teaching in this series.

### http.sslVerifyStatus for stapled OCSP responses

*Long thread: 32 replies so far from 5 participants over 58 days; 1 message that day.*

Grayson Gordon's patch adds an `http.sslVerifyStatus` boolean mapping to `CURLOPT_SSL_VERIFYSTATUS` so libcurl requests stapled OCSP responses, defaulting to off because enabling it would block servers that do not staple. After v7 combined the tests and the topic was reverted out of `next` over a t5585 failure traced to a GnuTLS curl backend, today Junio C Hamano reported that bisecting between jch..seen blamed this topic for linux-* CI jobs spinning until their 6-hour timeout, and he is tentatively ejecting it from `seen`.

### Fetching fewer branches from new remotes in shallow repos

*Long thread: 25 replies so far from 4 participants over 19 days; 1 message that day.*

Harald Nordgren's four-patch series avoids fetching every branch of a new remote in a shallow repository: it adds `remote.<name>.refmap`, infers branches from local upstreams, adds `git remote add --limited-fetch`, and defaults to `--limited-fetch` in shallow repositories. Today he posted v7, fixing the fatal "--refmap option is only meaningful with command-line refspec(s)" error that fired when only `remote.<name>.refmap` was configured, with tests showing that when both `.fetch` and `.refmap` are set, a refspec-less fetch follows `.fetch` while `.refmap` only remaps refs already fetched by name.

### Teaching documentation navigation in `man git`

*Long thread: 15 replies so far from 5 participants over 10 days; 8 messages that day.*

Julia Evans's patch rewrites `man git` to explain that `git push -h` gives short help and `git push --help` or `git help push` the full documentation, drops the gittutorial and giteveryday references, and points to `git help`, `git help --guides`, `git help --user-interfaces` and `git help --web`. Today the thread covered why git.github.io was replaced by git-scm.com (same content plus a table of contents, overview and search), and Junio C Hamano asked the next iteration to justify dropping the user-manual, datamodel and cli references; Evans gave her reasoning, and Hamano suggested updating cli instead.

## The day in brief
Junio C Hamano tentatively ejected the `http.sslVerifyStatus` topic from `seen` after bisecting linux-* CI timeouts to it, while Harald Nordgren posted v7 of the shallow-repo fetch series fixing a `--refmap` configuration error. Scott Chacon's RFC to allow responsible AI-assisted contributions under the DCO drew objections from brian m. carlsson and critique from Junio C Hamano.

## Notable threads

### RFC: allow responsible AI assistance in SubmittingPatches
Scott Chacon's RFC patch to `Documentation/SubmittingPatches` would allow AI-assisted contributions under the existing DCO with human review and an `Assisted-by:` trailer, aligning with SFC, Linux, Xen and Debian guidance. brian m. carlsson objected that the DCO cannot honestly be asserted for LLM output without knowing all training sources, citing legal liability as a distributor. Junio C Hamano critiqued the text as resting on an OpenAI model's output, largely unenforceable, and giving maintainers less latitude than the kernel guidelines.

### Faster SHA-1 collision detection via sha1dc-accel
Scott Chacon's series ports Sam Reis's Rust sha1dc crate approach to C in `sha1dc-accel/`, reporting roughly 2.7x faster hashing. Johannes Schindelin suggested reusing existing compression functions, offered a Perl script to regenerate the UBC tables and a SHA-NI patch closing the gap to Rust sha1dc to about 1%, and favors accepting the series. Junio C Hamano asked how the AI-generated code can be confirmed free of incompatible sources; Chacon said he discussed the port with Reis and is confident signing off under the DCO.

### Packfile corruption from stale delta base cache entries
Patrick Steinhardt's series fixes packfile corruption caused by stale delta base cache entries keyed by `packed_git` address and offset, reproducible via recursive submodule merges. Today he confirmed the reproducer also fails on Alpine with musl, Jeff King called the fix obviously correct after rejecting a pack-name comparison alternative, and Junio C Hamano marked the topic for `next`.

## In brief
- Maciej Ciemborowicz posted v3 of "refs: run copy and rename through transactions", routing copy/rename through ordinary transactions so the reference-transaction hook sees the whole operation; Junio C Hamano reported two test failures when applied to master.
- Patrick Steinhardt confirmed the push-resends-common-history bug mechanism (`odb_has_object()` false negatives after a pre-push repack) and provided an untested `send-pack.c` patch batching the negative-object checks.
- On Ravi Mistry's patch defaulting `blame` to `.git-blame-ignore-revs`, Junio C Hamano clarified the parser audit was delegated and Mistry will post a v2 for security reviewers.
- Jeff King's CI patch bumps the debian-11 job to debian-12; Junio C Hamano said he plans to propose dropping the linux32 job and reworking linux-TEST-vars on ubuntu:rolling.
- Daniel Gullberg reported that an unreachable UNC `safe.directory` entry on Windows stalls commands about 25 s in repos failing the ownership check.
- Kristofer Karlsson posted v3 of "fetch: write commit-graph using updated refs only" with a trimmed commit message and simplified tests.
- Karthik Nayak posted v4 of the packed-refs `fwrite()` optimization (~20% faster deletions); Patrick Steinhardt is happy with it.
- René Scharfe analyzed the ZIP timestamp bugs, recommending rejecting non-representable timestamps in `git archive --format=zip` and arguing against rejecting negative timestamps in fast-import.
