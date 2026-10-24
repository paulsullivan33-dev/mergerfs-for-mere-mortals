# Your First Pool

> **Lab status: documented-only.** This
> chapter follows the project's manual
> step by step, but this guide's bench
> could not run the mount — the sandbox
> has no FUSE device (chapter 3 tells
> the whole story, transcripts
> included). On any ordinary machine
> the commands below are the real
> first-run experience; nothing in the
> chapter depends on exotic hardware.

Two folders, one pool, and the habit
that makes mergerfs make sense: after
every action in the pool, look in the
branches to see what really happened.

## Set up the branches

Real pools use whole drives; the first
one can use two plain folders, which
is also how you rehearse before
committing real disks:

```
mkdir -p /mnt/disk1/media /mnt/disk2/media /pool
echo "from disk1" > /mnt/disk1/media/on-disk1.txt
echo "from disk2" > /mnt/disk2/media/on-disk2.txt
```

Two folders on (for now) the same
drive, each with a `media` folder and
one planted file. The planted files
matter: a pool over empty branches
proves nothing.

## Mount it

```
sudo mergerfs /mnt/disk1:/mnt/disk2 /pool \
  -o allow_other,use_ino,category.create=epmfs
```

(The colon-separated list is the
branches, in order. `allow_other` so
other users can see the pool —
chapter 9; `use_ino` as the manual
recommends; `epmfs` is the default
create policy, named here so the
command says what it does.)

**Check:**

```
$ ls /pool/media
on-disk1.txt  on-disk2.txt

$ findmnt /pool
/pool  /mnt/disk1:/mnt/disk2  fuse.mergerfs  rw,…
```

One folder, both files. Neither file
moved: each is still sitting in its
own branch, and the pool is showing
you the union. This is the moment the
model of chapter 2 becomes physical —
there is no pool *thing* apart from
the branches and the program
presenting them.

## Create something, then look behind the door

```
$ echo "made through the pool" > /pool/media/new-file.txt
$ ls /mnt/disk1/media /mnt/disk2/media
```

One of those listings now includes
`new-file.txt` — exactly one. The
create policy chose its branch (with
`epmfs`: among branches where `media`
exists — both — the one with most free
space). Repeat with a few more files
and a new subfolder, and predict the
branch *before* you check; being
wrong, once, teaches the `ep` rule
(chapter 5) faster than any table.

Reading behaves symmetrically: open
`on-disk1.txt` through the pool and
you are reading disk1's file. Delete
it through the pool and it is deleted
from disk1. The pool is not a copy, a
cache, or an index — it is a way of
addressing the branches.

## Unmount it

```
sudo fusermount -u /pool       # or: sudo umount /pool
```

**Check:** `/pool` is an empty folder
again, and both planted files are
still on their disks. That round trip —
mount, use, unmount, nothing changed
underneath — is the whole safety story
of experimenting with mergerfs: the
branches were never in danger, because
the branches were never modified by
being pooled.

## Where to go next

- Make it survive a reboot:
  chapter 7.
- Choose policies like you mean it:
  chapter 5.
- Meet the day a drive fills:
  chapter 6.
- Change a running pool's mind:
  chapter 8.
