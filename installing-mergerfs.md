# Installing MergerFS

MergerFS is one program with no
database, no runtime, and no
dependencies worth the name beyond
FUSE itself. Installing it is the
smallest chapter in this guide. The
interesting part — the part this
chapter spends its length on — is the
one prerequisite that decides whether
any of the rest of the guide can run
at all: a working FUSE device.

## The prerequisite: /dev/fuse

MergerFS is a FUSE filesystem
(chapter 2), so before installing
anything:

```
$ ls -l /dev/fuse
crw-rw-rw- 1 root root 10, 229 … /dev/fuse

$ grep -w fuse /proc/filesystems
nodev   fuse
```

On a normal desktop or server install
of any mainstream distribution, both
are simply true and always have been.
In containers, CI sandboxes, and other
restricted environments, the device is
commonly absent or unusable — passed
through only if whoever runs the
environment chooses (`--device
/dev/fuse` and friends). No mergerfs
option works around a missing device;
if the first command fails, fix the
environment first or run mergerfs
somewhere else. This guide's lab hit
exactly that wall, in full, below.

## Route 1: your distribution's package

```
sudo apt install mergerfs        # Debian / Ubuntu
sudo dnf install mergerfs        # Fedora family
```

One honest warning about this route:
**distributions ship old mergerfs.**
This lab installed Ubuntu 24.04's
package and got **version 2.33.5** —
several years behind upstream, whose
development has continued through the
2.40 series (the version Debian 12
ships). The core of the guide —
branches, policies, fstab, the control
interface — is the same in both, and
where the versions differ in a way a
reader would trip over, the chapters
say which is which (the control
interface moved; chapter 8). But if a
feature from a recent manual is
missing on your system, an old distro
package is the first suspect.

## Route 2: upstream builds

The project publishes release packages
(.deb and .rpm for common
distributions and architectures) and a
static tarball on its GitHub releases
page, and building from source is a
plain `make` away for those who need
it. This is the route to a current
version, at the usual price: you, not
your distribution, now own the updates.

## What this guide's lab actually did

The lab installed the distribution
package on Ubuntu 24.04 (transcripts
in `mergerfs-lab/transcripts/`):

- **t02–t03** — the package from the
  Ubuntu archive for 24.04: mergerfs
  **2.33.5-1ubuntu2**. It installed and
  runs: `mergerfs --version` answers
  `2.33.5`. One packaging wrinkle,
  recorded because a reader on the same
  release may meet it: the old `fuse`
  (v2) package was left unconfigured
  because the sandbox already carried
  `fuse3`, which declares a conflict
  with it. MergerFS itself was
  unaffected — the binary bundles its
  FUSE library (an `ldd` shows no
  libfuse at all) — and `fusermount`
  resolves to fuse3's. On a stock
  Ubuntu desktop this conflict does
  not arise; in this sandbox it is a
  footnote, not a blocker.
- **t01** — the FUSE check, and the
  lab's real result: `/proc/filesystems`
  lists `fuse` (the kernel supports
  it), but **`/dev/fuse` does not
  exist, the module cannot be loaded,
  and creating the node is refused**
  (`mknod: Operation not permitted`).
  The sandbox simply does not provide
  the device.
- **t04** — the mount attempt, made
  anyway so the failure is on the
  record: two branch folders with a
  planted file each, and mergerfs
  answers with the canonical error:

```
$ mergerfs …/diskA:…/diskB …/pool -o allow_other,use_ino,category.create=epmfs
fuse: device not found, try 'modprobe fuse' first
```

Everything in this guide after this
chapter that involves a *live* pool is
therefore **documented-only**: written
from the project's manual — the same
manual, shipped with the package, that
the lab verified its claims against
(**t05**: the category defaults
`epall` / `epmfs` / `ff`, minfreespace's
4G default, the full policy table, the
control-interface keys) — and labelled
as documented-only in `criteria.md`,
which keeps the ledger. The chapters
do not pretend otherwise anywhere: if
a "Check:" block describes a mounted
pool, it describes what a reader on a
normal machine will see, not what this
bench saw.

## Post-install checks that do work anywhere

```
$ mergerfs --version
mergerfs version: 2.33.5

$ man mergerfs        # the manual is half the product; keep it close
```

**Check:** if `--version` answers and
`/dev/fuse` exists on your machine,
you have everything chapter 4 needs.
If the second half fails, chapter 14's
"FUSE device not found" entry is your
next stop — and it is an environment
problem, never a mergerfs setting.
