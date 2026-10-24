# Writing Criteria

Every guide in this repo is written to the same
standard. If you're adding or editing a guide —
human or AI — hold it to these.

1. **Plain words, real ideas.** Simple language,
   never dumbed-down concepts. Explain jargon on
   first use; collect terms in the glossary.
2. **No hype.** No marketing adjectives, no
   "revolutionary," no promises about what
   something will do for the reader.
3. **Honest about limits.** Say what the thing
   *can't* do. State uncertainty instead of
   smoothing it over. Validation notes include
   what remains unverified.
4. **No invented numbers.** Benchmarks, defaults,
   and specs are either measured or qualified with
   "depends on your setup." If it wasn't tested,
   say so.
5. **Checkable.** Commands are meant to be run,
   with "Check:" blocks showing what success looks
   like. Sources are linked, not asserted.
6. **Respect the reader's setup.** Call out
   Windows/Mac/Linux differences. Don't assume a
   GPU, a budget, or prior experience.
7. **Safety isn't skipped.** Real risks get
   stated plainly, not buried.
8. **Scoped.** Each guide says what it covers and
   points elsewhere for the rest. No guide
   pretends to be the whole subject.
9. **Structure over prose.** Tables, before/after
   examples, symptom → fix. Scannable beats essay.

## Verification notes for this guide

Built and verified 2026-10-23/24 against
**mergerfs 2.33.5** — the package Ubuntu 24.04
(noble) ships, `mergerfs 2.33.5-1ubuntu2` — on
Ubuntu 24.04.5 LTS, kernel 7.0.0-39-generic, in
this series' build sandbox. Ubuntu's package is
several years behind upstream (whose development
has continued through the 2.40 series, the
version Debian 12 ships); where the versions
differ in ways a reader would trip over, the
chapters say which is which.

**Primary sources read, not remembered:** the
mergerfs manual page shipped with the installed
package (`mergerfs(1)`, rendered locally and
quoted from), the installed program's own
`--help` output, and the project's online
documentation for the policy set, the runtime
control interface and the recommended option
combinations the chapters cite.

**Ran for real** (transcripts in
`mergerfs-lab/transcripts/`):

- t00 — the first `apt-get update` failed on a
  lists lock held by another process sharing the
  sandbox. The lab used an isolated apt state
  directory with a minimal noble (main + universe)
  source list and fetched the .deb packages
  directly; no other process's state was touched.
- t01 — the FUSE facts, checked four ways:
  `/proc/filesystems` lists `fuse` (kernel support
  present), but `/dev/fuse` does not exist,
  `modprobe fuse` fails (no module for this
  kernel), creating the node is refused
  (`mknod: Operation not permitted`), and opening
  the path fails. The sandbox provides no FUSE
  device and no way to make one.
- t02–t03 — the install, for real: mergerfs
  2.33.5-1ubuntu2 installed with dpkg alongside
  libfuse2t64 2.9.9; `mergerfs --version` answers
  `2.33.5`; two branch folders created, each with
  a `media/` folder and one planted file.
  Recorded wrinkle: the v2 `fuse` package stayed
  unconfigured because fuse3 3.14 was already
  present and declares a conflict; mergerfs was
  unaffected — its binary bundles its FUSE
  library (an `ldd` shows no libfuse) — and
  `fusermount` resolves to fusermount3.
- t04 — the mount attempt, made anyway so the
  failure is evidence: `mergerfs diskA:diskB pool
  -o allow_other,use_ino,category.create=epmfs`
  answers `fuse: device not found, try 'modprobe
  fuse' first` (exit 1). The pool stayed empty;
  `fusermount -u` on the never-mounted folder
  fails as it should.
- t05 — every load-bearing claim about defaults
  and policies verified against the shipped
  manual: category defaults (action `epall`,
  create `epmfs`, search `ff`), minfreespace's
  4G default, the full policy table, the
  `user.mergerfs.*` control keys (including the
  `.mergerfs` pseudo-file form this version uses
  and branch modes `=RW`), and moveonenospc's
  default of `false`.

**Documented only, and labelled as such in the
chapters:** everything that requires a live
pool, which the missing FUSE device made
impossible on this bench — mounting and daily
pool behaviour (chapter 4), policies choosing
branches in action (chapter 5), minfreespace
refusals and moveonenospc rescues (chapter 6),
boot behaviour of the fstab forms (chapter 7),
runtime xattr changes on a running pool
(chapter 8), and sharing a pool onwards over
NFS or Samba (chapter 9). Those chapters follow
the project's manual — the same manual t05
verified the defaults against — and none of
them claims a result this lab measured. No
performance figure anywhere in the guide was
measured or invented: chapter 13 says what
FUSE overhead is, not how big it is.

**Fixtures:** the two branch folders (`diskA`,
`diskB`) with their `media/` folders and the
planted text files `on-a.txt` and `on-b.txt`
are invented for the lab, as are every drive
name, folder and library in the chapters and
recipes. All examples are for general
audiences. No secrets appear in any transcript.
