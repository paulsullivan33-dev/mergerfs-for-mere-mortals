# The Pool and Other Software

Most programs treat a pool exactly like
any other directory, because it is one.
This chapter collects the exceptions
and the interfaces — the places where
software notices the floor is FUSE.

## Other users: allow_other

By default a FUSE mount is visible only
to the user who mounted it. A media
server running as its own user, a
backup job running as root, a second
login — all see an empty directory or a
permission error where the pool should
be. The fix is the mount option
**`allow_other`**, plus one system
setting it depends on: `user_allow_other`
must be enabled in `/etc/fuse.conf`.
Distributions differ in whether that
line starts commented out; if
`allow_other` seems to do nothing,
that file is the first place to look
(chapter 14).

## Sharing the pool onwards: NFS and Samba

You can export or share a pool like any
directory, with two honest warnings:

- **Serve the pool, or serve the
  branches — choose one.** Clients that
  sometimes see the pool and sometimes
  a branch will meet inconsistent file
  handles, inode numbers, and locks.
  The usual answer is: applications and
  network shares use the pool; only
  you, doing maintenance, touch the
  branches directly.
- **NFS is the fussy one.** Exporting
  FUSE filesystems over NFS needs care
  (stable file handles across restarts
  is the recurring theme); the mergerfs
  manual carries a section on it,
  including the `fsid` and inode
  settings it recommends. Samba shares
  of pools are common and comparatively
  uneventful. Both are documented-only
  in this guide — the lab shared its
  pool with no second machine (see
  `criteria.md`) — so follow the
  project's notes, and test with the
  client that matters before trusting
  it.

## Programs that push on the floorboards

- **Databases** (SQLite included): the
  `mmap` and locking patterns databases
  rely on meet FUSE's limits. The
  mergerfs manual is explicit that
  database-heavy applications are not
  what it is for. Keep the database on
  a real filesystem; let it *reference*
  files in the pool if it must.
- **Hard links across the pool**: a
  link can only join names on the same
  branch (that is where the file
  physically is). Download tools that
  seed by linking from a download
  folder to a library folder need both
  folders' paths to resolve onto one
  branch — `ep*` policies (chapter 5)
  exist substantially to make that
  likely. `EXDEV` in a log is this
  chapter knocking.
- **Torrents and preallocation**: tools
  that preallocate and write sparsely
  interact with `moveonenospc` and
  free-space accounting in ways worth
  testing on your own pool before a
  large download depends on them.
- **Everything else** — media servers,
  photo managers, backup *sources*,
  download targets, document trees —
  is the pool's home turf and the
  reason it exists.

## Watching the pool from outside

From another shell, the pool's health
is three questions: is the mergerfs
process alive, do the branch mounts
exist, and how full is each branch
(chapter 6). Tools like
`mergerfs.collect-info` (from the
companion mergerfs-tools) bundle the
configuration and branch state for a
bug report; for everyday purposes,
`findmnt /pool`, `df` on the branches,
and the `.mergerfs` control file
(chapter 8) answer everything.
