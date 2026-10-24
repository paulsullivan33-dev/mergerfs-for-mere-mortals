# The Moving Parts

A mergerfs pool is three things: some
real directories, one program, and a
list of rules. This chapter names each
part and what breaks when it stops,
because most confusion with mergerfs is
really confusion about which layer is
doing what.

## The branches

A **branch** is an ordinary directory,
usually the root of a drive's
filesystem:

```
/mnt/disk1   (a 2 TB ext4 drive)
/mnt/disk2   (a 4 TB XFS drive)
/mnt/disk3   (an older 1 TB drive)
```

Branches do not have to be whole
drives — any directory will do — and
they do not have to match: sizes,
filesystems, and speeds can all differ.
MergerFS reads and writes the branches
directly, as the user it runs as. If it
cannot write a branch (permissions, a
read-only mount), the pool still serves
what it can read, and creation policies
skip branches they cannot use.

Two consequences follow from branches
being plain directories:

- **Your data outlives mergerfs.** Stop
  mergerfs forever and every file is
  still sitting on its drive, in the
  same folder layout, readable by
  anything. There is no pool format to
  undo.
- **You can look behind the door.** Any
  file in the pool can be found in
  exactly one branch under the same
  relative path. `find` on the branches
  tells you the truth about where
  things live; chapter 4 leans on this
  hard, because it is the fastest way
  to understand policies.

## The program

`mergerfs` itself is a single FUSE
daemon. You give it the branch list and
a mount point:

```
mergerfs /mnt/disk1:/mnt/disk2:/mnt/disk3 /pool
```

and it stays in the foreground of its
own little world, answering the
kernel's file requests by consulting
the branches. Unmount with
`fusermount -u /pool` (or `umount` as
root) and it exits; nothing is left
behind. If the mergerfs process dies —
crash, `kill`, machine reboot — the
mount point goes dead (programs see
transport errors) but the branches are
untouched. Restart mergerfs and the
pool is back, identical, because the
pool *is* the branches.

FUSE needs `/dev/fuse` and permission
to use it. On a normal Linux install
both are simply there. In containers
and restricted sandboxes they often are
not, and no amount of configuration
fixes a missing device — chapter 3
meets this head-on, because this
guide's lab did.

## The policies

Every operation mergerfs performs —
creating a file, looking one up,
renaming, deleting — needs a branch
chosen, and a **policy** is the rule
that chooses. Policies come in families:

| Category | Decides | Examples |
|---|---|---|
| **create** | Where a *new* file or directory goes | `mfs`, `epmfs`, `ff` |
| **search** | Which copy answers a lookup when several branches have the path | `ff`, `newest` |
| **action** | Which copies a change (chmod, delete, rename…) applies to | `epall`, `all` |

Policy names beginning with `ep`
("existing path") first look for
branches where the file's parent
directory already exists, and only fall
back to all branches when none do. That
one letter is the difference between a
pool that keeps related files together
on one drive and a pool that scatters a
folder across every drive it owns.
Chapter 5 is devoted to choosing them.

Two settings steer the create policies:

- **minfreespace** (default 4G): a
  branch counts as "full" for creation
  once its free space falls below this.
  It is a threshold for choosing, not
  a reserve that is enforced later.
- **moveonenospc** (default off): if a
  *write to an existing file* fails
  because the drive filled up, mergerfs
  can move the file to another branch
  mid-write and retry. Chapter 6 covers
  what it can and cannot rescue.

## The control interface

A running pool carries a control
interface: extended attributes
(**xattrs**) in a `user.mergerfs.*`
family. Reading them shows the live
configuration; setting them changes it
— branches, policies, thresholds — with
no remount. On current releases the
attributes are set on the mount point
itself. On the 2.33 series (which is
what Ubuntu 24.04 packages — chapter 3)
the handle is a hidden pseudo-file,
**`.mergerfs`**, at the pool's root; the
same keys work through it. Chapter 8
uses this properly. For now, remember
the shape: the configuration is not
only what you typed at mount time; it
is state you can inspect and adjust
while the pool runs.

## What breaks when each part stops

| Part | If it stops | Your files |
|---|---|---|
| A branch drive | The pool loses that branch's files from view; the rest serve on | Safe on the dead drive, if the drive is safe |
| The mergerfs process | The mount point errors until restarted | Untouched, on their branches |
| The machine | Everything stops | Untouched; remount after boot (chapter 7) |
| Your attention | The pool fills every drive to the brim, one file at a time | Present, but chapter 6 is about not living there |
