# MergerFS for Mere Mortals

Most home servers grow the same way: one
drive at a time, bought when the last one
filled, until the media library is a
guessing game across four mount points.
**MergerFS** is the open-source answer
built for exactly that: a union
filesystem that presents several ordinary
folders — each on its own drive, in its
own size and filesystem — as one pooled
folder, with plain rules deciding which
drive each new file lands on. The drives
are never reformatted and no file is
ever split. This guide explains how it
works, how to install it, how to choose
its policies with your eyes open, and
how to live with it afterwards — without
fooling yourself about the part that
matters most: a pool is a front door,
not a safety net, and it protects
nothing by itself.

The lab behind this guide is honest
about an unusual outcome. It installed
the real thing — **mergerfs 2.33.5**,
the package Ubuntu 24.04 ships — and
verified the program, its manual, and
its option defaults directly. It then
tried to mount a pool of two branch
folders, and could not: the build
sandbox has no FUSE device, cannot
create one, and no mergerfs setting
fixes a missing `/dev/fuse`. The mount
attempt and the diagnosis are on the
record in the lab's transcripts, the
guide says plainly in chapter 3 which
chapters are therefore taught from the
project's own manual rather than from
a live bench, and `criteria.md` keeps
the ledger of what was run and what
was only read. A guide that pretends
otherwise would teach you less than
this one does.

## What this guide covers

- [**What MergerFS is**](what-mergerfs-is.md) —
  the mental model: branches behind one
  front door, files whole on single
  drives; what it is not (RAID, a
  format, striping, a backup)
- [**The moving parts**](the-moving-parts.md) —
  the branches, the FUSE daemon, the
  policies, and the control interface:
  what each holds, and what breaks when
  each stops
- [**Installing MergerFS**](installing-mergerfs.md) —
  the `/dev/fuse` prerequisite, distro
  packages and their old versions,
  upstream builds, and the lab's
  declared result told in full
- [**Your First Pool**](your-first-pool.md) —
  two folders, one mount, and the habit
  of looking in the branches to see
  what the pool really did
- [**Policies in Practice**](policies-in-practice.md) —
  create, search and action categories;
  `ff`, `mfs`, and the `ep` family; path
  preservation and what `ep` really buys
- [**When a Drive Fills Up**](when-a-drive-fills-up.md) —
  minfreespace, the ENOSPC moment,
  moveonenospc and its honest limits,
  and the day every drive is full
- [**Making It Permanent**](making-it-permanent.md) —
  fstab without the boot-time traps,
  the config file, and the short list
  of options worth setting deliberately
- [**Runtime Tuning**](runtime-tuning.md) —
  reading and changing a live pool
  through its xattr control interface,
  and the cache options that actually
  matter
- [**The Pool and Other Software**](the-pool-and-other-software.md) —
  allow_other, sharing onwards over NFS
  and Samba, and the programs (databases
  above all) that push on the floorboards
- [**Backup and Drive Failure**](backup-and-drive-failure.md) —
  what a dead drive actually costs, the
  SnapRAID pairing, and backup rules the
  pool does not change
- [**Recipes**](recipes.md) — a media
  library, a download landing pad, a
  growing photo archive, and an
  everything drawer, each with its
  settings and its reasons
- [**The Neighbours**](the-neighbours.md) —
  RAID, LVM, ZFS, SnapRAID, OverlayFS
  and friends: what each one trades
- [**Limits**](limits.md) — no redundancy,
  no split files, FUSE overhead, no
  databases, stated plainly
- [**Troubleshooting**](troubleshooting.md) —
  the empty pool that isn't, the space
  error that is, the missing FUSE
  device, and the other traps, symptom
  first
- [**Glossary**](glossary.md) — the terms,
  collected in one place

## What it doesn't cover

Building mergerfs from source beyond a
pointer, tuning for workloads this
guide never measured (it invents no
benchmarks), mergerfs on non-Linux
systems, and SnapRAID itself — the
parity partner gets its own guide in
this series, and chapter 10 explains
the pairing without teaching the tool.
Exposing network shares to the internet
safely is likewise pointed at, not
taught.

## How to use this guide

Read chapter 1 for the model and
chapter 2 before installing anything —
knowing which part does what turns
later confusion into a named problem.
Chapter 3 includes the one check that
decides whether a pool can run on your
machine at all; do it first. Chapter 4
is a rehearsal with two plain folders:
run it before offering mergerfs real
drives, and keep its habit — look in
the branches — for as long as you run
pools. Chapter 10 is not the optional
ending: decide what protects the data
before the pool holds anything you
would miss. The lab's transcripts live
in `mergerfs-lab/transcripts/`; its
notes are `examples/lab-notes.txt`.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [SnapRAID for Mere Mortals](https://paulsullivan33-dev.github.io/snapraid-for-mere-mortals/)
- [ZFS for Mere Mortals](https://paulsullivan33-dev.github.io/zfs-for-mere-mortals/)
- [Samba and NFS for Mere Mortals](https://paulsullivan33-dev.github.io/samba-and-nfs-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/mergerfs-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/mergerfs-for-mere-mortals/blob/main/LICENSE-CODE).
