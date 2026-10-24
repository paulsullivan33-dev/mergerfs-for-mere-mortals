# Glossary

**allow_other** — the FUSE mount option
that lets users other than the one who
mounted the pool see it at all. Needs
`user_allow_other` in `/etc/fuse.conf`.
Chapters 7 and 9.

**Branch** — one real directory, usually
a whole drive's root, given to mergerfs
as a source. The pool is the branches,
viewed together. Chapter 2.

**Category** — a family of operations
sharing one policy: **create** (new
files), **search** (lookups), **action**
(changes to existing files). Chapter 5.

**Control file / control interface** —
the live configuration of a running
pool, read and written through
`user.mergerfs.*` xattrs on the mount
point, and readable as text in the
hidden `.mergerfs` pseudo-file at the
pool's root. Chapter 8.

**ENOSPC** — "No space left on device":
the error a write gets when its branch
is physically full. Chapter 6.

**ep (existing path)** — the prefix on
policies (`epmfs`, `epff`, `epall`) that
first restricts candidates to branches
where the file's parent directory
already exists. Chapter 5.

**EXDEV** — "Invalid cross-device
link": a rename or link tried to cross
branches where the policy doesn't allow
it. Chapters 5 and 9.

**ff (first fit / first found)** — the
policy that takes the first branch
which qualifies: for creation, the
first with enough free space; for
search, the first holding the path.

**FUSE** — "Filesystem in Userspace":
the kernel mechanism that lets a
program like mergerfs provide a
filesystem. Needs `/dev/fuse`. Chapters
2 and 3.

**fusermount** — the helper that mounts
and unmounts FUSE filesystems for
unprivileged users: `fusermount -u
/pool`.

**minfreespace** — the free-space
threshold below which create policies
stop choosing a branch. Default 4G.
A creation threshold, not a reserve.
Chapter 6.

**moveonenospc** — when a write to an
existing file fails with ENOSPC,
mergerfs moves the file to another
branch and retries the write. Chapter
6.

**mfs** — "most free space": the create
policy that picks the branch with the
most free space; `epmfs` is its
existing-path form and the default.

**Path preservation** — the behaviour
of `ep*` policies in renames and links:
keep the file's branch consistent with
where its path already lives, even if
that means moving it between branches.
Chapter 5.

**Policy** — the rule that picks a
branch for an operation. Set per
category (`category.create=mfs`) or per
function (`func.create=ff`). Chapter 5.

**Pool** — the merged mount point: the
single directory where all branches
appear as one tree. Also, this guide's
word for the whole arrangement.

**SnapRAID** — the scheduled-parity
tool most often paired with mergerfs:
mergerfs presents; SnapRAID protects.
Chapter 10.

**Union filesystem** — any filesystem
that presents several directories as
one tree. MergerFS is one; OverlayFS is
the kernel's. Chapter 12.

**xattr** — extended attributes:
name/value metadata attached to files.
MergerFS uses them both on your files
(passed through to the branches) and as
its own control interface on the mount
point. Chapter 8.
