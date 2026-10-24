# Backup and Drive Failure

This is the chapter mergerfs guides are
tempted to skip, and the one that
matters most. A pool is a convenience
layer. It provides **no redundancy at
all**: no parity, no mirror, no spare.
Reading this chapter takes five
minutes; skipping it can cost a library.

## When a drive dies

Say the pool has three branches and
disk2 fails outright. What happens is
unspectacular, which is the good news:

- The files that lived on disk2 are
  gone from the pool — and gone,
  period, unless a backup or parity
  scheme (below) says otherwise.
- Everything on disk1 and disk3 is
  exactly where it always was, whole
  and readable, through the pool or
  directly.
- Compare like with like: in a striped
  RAID 0 the same failure takes
  *everything*; in mergerfs it takes
  one drive's worth. The pool degrades
  by amputation, not by collapse.

Your recovery is then a restore
question, not a mergerfs question:
replace the drive, mount it, add it as
a branch (live, chapter 8), and restore
that drive's contents from backup into
the same folder layout. The pool
reassembles itself around the restored
branch because the pool was never more
than the branches.

**Check what you actually stand to
lose:** keep a per-branch file listing
somewhere (`find /mnt/disk2 -type f >
disk2-files.txt`, refreshed weekly by
cron). After a failure it is the
difference between "restore these
4,000 files" and "guess what used to
be on that drive".

## The usual partner: parity beside the pool

The combination most home pools grow
into pairs mergerfs with **SnapRAID**:
SnapRAID computes parity for the
*contents* of the data drives on a
schedule, so a dead drive can be
rebuilt from the survivors plus parity
— as long as you run its sync before
more drives fail than the parity
covers, and accept that files changed
since the last sync are only partly
protected. MergerFS presents; SnapRAID
protects; neither does the other's job.
SnapRAID is a guide of its own in this
series; here it is enough to know the
pairing exists and that running a pool
of irreplaceable files *without* it —
or without the next section — is a
decision, not a default.

## Backup, unchanged by the pool

MergerFS changes nothing about backup
fundamentals, and quietly tempts people
to think it does ("it's all one big
folder now, it must be safer"). It is
not safer. The working rules:

- **Back up through the pool, restore
  to the branches.** Backup tools read
  the pool as one tree — good. Restores
  should target a branch layout you
  choose, remembering a restored file
  can be larger than any single branch
  has room for if you pool small drives.
- **The 3-2-1 shape still applies:**
  three copies, two kinds of media, one
  elsewhere. A second pool in the same
  box is not "elsewhere".
- **A pool is a fine backup *target***
  for other machines — cheap, expandable
  storage is exactly its talent — as
  long as the backups on it are not the
  only copies of anything.
- **Test a restore.** Restore one
  folder from your backup to a scratch
  branch, compare it, delete it. The
  first restore you ever attempt should
  not be the one after a drive dies.

## Retiring a drive on purpose

The graceful version of drive failure:
to remove a branch forever, copy its
contents elsewhere in the pool first
(working in the branch folders directly
is fastest — the pool is not involved),
verify counts and sizes, then remove
the branch from the configuration
(chapter 8 for the live version, and
chapter 7 so the change survives a
reboot). Do not expect a policy to
drain a drive for you: policies place
*new* files, and no policy moves
existing ones. Copying is the drain;
everything else is tidying.
