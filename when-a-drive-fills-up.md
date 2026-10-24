# When a Drive Fills Up

A pool does not make space; it only
spends the space it has more politely.
This chapter is about the three moments
that surprise people: a drive reaching
the threshold, a write hitting a truly
full drive, and the slow realisation
that every drive is full.

## minfreespace: the polite threshold

Create policies stop *choosing* a
branch once its free space drops below
**minfreespace** (default 4G, set with
`-o minfreespace=...`, per-branch
overrides exist). Understand what that
means precisely:

- It affects **new creations** only.
  Existing files on the branch can
  still grow — an appending log or a
  downloading file can take its drive
  from "below threshold" to "physically
  full" without asking any policy.
- It is evaluated per branch, in bytes
  (suffixes `K`, `M`, `G` understood).
- On small drives, 4G may be most of
  the drive; on a 20 TB drive it is a
  rounding error. Set it to the size of
  "the largest file I might create,
  plus a margin" — for media pools,
  10–20G is a common sanity figure.
  This is guidance, not a measurement;
  the right number depends on your
  setup.

## ENOSPC: the write that hits the wall

When a program writes to an existing
file and the branch underneath reaches
zero free bytes, the write fails with
**ENOSPC** — "No space left on device".
The application's error message will
usually blame the pool, not the drive.
Two mergerfs features are built for
exactly this moment:

**moveonenospc** (`-o moveonenospc=true`)
— when a write fails with ENOSPC (or a
quota error), mergerfs picks a new
branch using the `mfs` rule (or a policy
you name: `moveonenospc=mfs`), moves
the whole file there — trying to keep
its metadata — unlinks the original,
and retries the write, all inside the
failing operation. The application
notices nothing but a pause.

Honest limits, from the manual:

- It rescues **writes that fail**, not
  creations: a brand-new file is placed
  by the create policy in the first
  place.
- The file being moved needs somewhere
  to go: another branch with room for
  the whole file.
- A file already open and memory-mapped
  is not a candidate for a mid-write
  relocation party trick.

**Check:** with moveonenospc on, fill
one branch nearly full, then append to
a file living on it until you cross the
line. The append succeeds, and looking
in the branches afterwards shows the
file has changed drives. (This guide's
lab describes its attempt at this in
`mergerfs-lab/`; where the bench could
not run a mount, the chapter you are
reading follows the project's manual
and says so — see `criteria.md`.)

## All drives full

When every branch is below
minfreespace, creations fail even
though `df` on the pool may still show
free bytes summed across drives —
because the pool's total is not a
place a file can live. A file needs
*one* branch with room for *all* of it.
Remedies, in the order people actually
apply them:

1. Delete or move something off the
   fullest branches (the branches are
   plain folders; do it directly).
2. Add a drive: mount it, add its
   folder as a branch — live, via the
   control file (chapter 8), or by
   remounting with the longer list.
3. Lower minfreespace if it was set
   larger than your largest files need.
4. Rebalance by hand if `ff`-style
   filling left the space unusable —
   there is no built-in balancer in
   mergerfs itself; the companion
   *mergerfs-tools* project ships
   utilities, and plain `rsync` between
   branch folders achieves the same for
   a folder at a time.

## Watching it happen

`df -h /pool` shows the pool's summed
capacity — useful for "how much room
does the library have", useless for
"will this file fit". For the second
question, look at the branches:

```
df -h /mnt/disk1 /mnt/disk2 /mnt/disk3
```

Run that habitually, or let any disk
monitor you already trust watch the
branches. The pool hides imbalance by
design; your monitoring should not.
