# Limits

What mergerfs cannot do, collected in
one place and stated plainly. Most of
these are in earlier chapters in
context; this is the unvarnished list.

- **No redundancy.** No parity, no
  mirroring, no error correction. A
  dead drive's files are gone unless a
  backup or a parity partner (chapter
  11) covers them. This is the limit
  people most often talk themselves
  out of believing.
- **Files are never split.** No
  striping means no file reads faster
  than its own drive, and no file can
  be larger than the free space of a
  single branch. A pool of five 1 TB
  drives cannot hold a 1.2 TB file.
- **FUSE overhead is real.** Every
  operation passes through a userspace
  program. For media serving and
  general storage this is usually
  invisible; for workloads of many
  small files, heavy metadata traffic,
  or the last percent of disk
  throughput, measure on your own
  hardware before committing. (This
  guide invents no numbers: it did not
  benchmark, and neither should its
  claims.)
- **Not for databases.** `mmap`-heavy
  and locking-sensitive applications —
  databases above all — meet real FUSE
  limits. The project's manual says so
  itself. Keep databases on plain
  filesystems.
- **A dead mergerfs process is a dead
  mount.** The branches are unharmed,
  but everything talking to the pool
  sees errors until it is restarted.
  It is one daemon; run it like one
  (chapter 7, and monitor it).
- **Permissions are the branches'
  permissions.** MergerFS does not add
  an access-control layer of its own;
  it acts with the credentials of the
  operations it serves. If the branch
  folders are a mess, the pool is a
  neat window onto a mess.
- **Free space is a sum, not a place.**
  The pool can report gigabytes free
  while being unable to accept a file,
  when no single branch has room for
  it whole (chapter 6).
- **Case, charset, and clock quirks
  are inherited.** Branches on
  filesystems with different filename
  rules or skewed clocks produce the
  behaviours you'd predict once you
  remember the pool is only a view:
  `newest`-style policies trust those
  clocks.
- **It needs FUSE.** No `/dev/fuse`, no
  pool — in locked-down containers and
  some virtualised environments that
  is the end of the conversation, as
  this guide's own lab found
  (chapter 3).
