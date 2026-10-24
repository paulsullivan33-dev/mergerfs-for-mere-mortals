# What MergerFS Is

MergerFS is a union filesystem: it takes
two or more ordinary directories — each
living on its own drive, its own
filesystem, its own size — and presents
them as one directory tree. The drives
stay exactly as they were. Nothing is
reformatted, nothing is striped, and no
file is ever split across drives. What
mergerfs adds is a single front door:
you read and write through the pooled
mount point, and mergerfs decides, file
by file, which drive each file really
lives on.

It runs in userspace, through FUSE (the
kernel mechanism that lets a normal
program provide a filesystem). That one
fact explains most of its character: it
is easy to install, safe to experiment
with — unmount it and your drives are
untouched — and a little slower than a
kernel filesystem, because every file
operation takes a trip through the
mergerfs program.

## The shape: branches behind one door

Everything in mergerfs is expressed in
three ideas:

| Idea | What it is |
|---|---|
| **Branch** | One real directory on one real drive, given to mergerfs as a source |
| **Pool** (the mount point) | The single directory where the branches appear merged |
| **Policy** | The rule mergerfs uses to choose a branch — for a new file, for a lookup, for a rename |

That is nearly the whole model. Files
keep their names and paths: a file at
`/pool/films/example.mkv` is really at
`/drive2/films/example.mkv` on exactly
one branch, whole and readable without
mergerfs. Chapter 2 works through the
parts; for now, hold the shape: many
real folders behind one apparent folder,
and a set of rules deciding which real
folder each operation lands on.

## Who it is for

The classic user has several drives of
mixed ages and sizes holding media —
films, music, photos, downloads — and is
tired of remembering which drive has
room and which one holds the film they
want. Applications that expect one big
library folder (a media server is the
usual case) point at the pool; the
drives behind it can be added, swapped,
and replaced one at a time, without
rebuilding anything. It also suits
download and scratch storage, where the
cost of the pool stopping is low and the
convenience of one big namespace is the
whole point.

## What it is not

- **Not RAID.** There is no parity and
  no mirroring. If a drive dies, the
  files on that drive are gone, exactly
  as if mergerfs had never existed.
  Chapter 10 is blunt about this and
  about the usual partner (SnapRAID)
  that supplies the missing redundancy.
- **Not a filesystem format.** Branches
  keep their own filesystems — ext4,
  XFS, NTFS, whatever they already are.
  A branch unplugged from the pool is
  just a drive with files on it.
- **Not striping.** A file is never
  spread across drives, so no single
  file reads faster than its one drive,
  and no file can outgrow the largest
  single branch. A 3 TB file needs a
  branch with 3 TB free.
- **Not a backup.** A pool makes
  accidental deletion *easier* to do at
  scale, not harder. The backup chapter
  of your life still needs writing.
- **Not invisible to software.** Most
  programs never notice the pool. A few
  do: anything using `mmap` heavily,
  databases, and hard-link gymnastics
  meet real limits, covered in chapters
  9 and 13.

## The one-sentence version

If your files are scattered across
drives because the drives arrived one at
a time, mergerfs gives them one front
door and a set of placement rules — at
the price of remembering that the door
is not a safety net.
