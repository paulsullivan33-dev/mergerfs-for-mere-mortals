# Troubleshooting

Symptom first, cause second, fix third.
Every entry here is a pool behaving
exactly as configured by someone who
expected something else.

## The pool is empty — but the drives are full

**Symptom:** `/pool` shows nothing; the
branch folders are full of files.

**Causes, in order of likelihood:** the
pool was mounted over branch mount
points *before* the drives themselves
were mounted, so mergerfs pooled the
empty directories on the root
filesystem, and the real drives later
mounted over the tops of their own
folders. Or: you are looking at the
mount as a different user without
`allow_other` (see below) — the pool
isn't empty, it's just not yours to
see. Or the branch list has a typo and
you have pooled two empty folders with
total sincerity.

**Fix:** unmount the pool, confirm the
branches are mounted (`findmnt` each
one, `ls` the files), then mount the
pool again. For the boot-time version,
chapter 7's `x-systemd.requires-mounts-for`
is the prevention.

## Other users see an empty directory or "permission denied"

**Cause:** FUSE's default is a private
mount. The pool needs `allow_other`,
and `allow_other` needs `user_allow_other`
enabled in `/etc/fuse.conf`.

**Fix:** uncomment/set that line,
remount with `allow_other`, and confirm
with `findmnt /pool` that the option is
present. This is the single most common
mergerfs support question; it is never
a mergerfs bug.

## "No space left on device" — but df shows space

**Cause:** `df` on the pool sums all
branches. A file needs *one* branch with
room for all of it (chapter 6), and
creation also respects `minfreespace`.

**Fix:** `df -h` the branches
individually. Free space on the full
one, or lower a minfreespace that was
set larger than your files need, or add
a branch. If the failing write is to an
*existing* file and you expected
`moveonenospc` to save it, check the
option is actually set — `getfattr -d
/pool` shows the live truth (chapter 8).

## A program reports EXDEV / "Invalid cross-device link"

**Cause:** a rename or hard link tried
to cross branches. Under a
non-path-preserving policy renames
never cross branches and fail this way;
hard links can only ever join names on
the same branch (chapter 9).

**Fix:** decide which behaviour you
want *on purpose*: an `ep*` create
policy makes same-branch placement
likely; `ignorepponrename=true` makes
renames stay put and tells you so
honestly; or restructure so the two
folders involved resolve to one branch.

## Files "vanish" after a reboot

**Cause:** the fstab race of chapter 7,
or the pool never being in fstab at
all — or a drive failing to mount and
`nofail` doing its silent job. The
files have almost certainly not gone
anywhere: they are on their branches,
unpooled.

**Fix:** check the branches directly,
check `findmnt /pool`, check the boot
log for the drive. Files in mergerfs
land are nearly always *unviewed*, not
*deleted* — work the layers from the
bottom (drive mounted? branch listed?
pool mounted?) before concluding
anything worse.

## The mount itself fails: "fuse: device not found"

**Cause:** there is no working
`/dev/fuse` — a missing device node, a
container without the device passed
through, a sandbox that forbids
creating it. This guide's lab met
exactly this: kernel FUSE present in
`/proc/filesystems`, no `/dev/fuse`,
and `mknod` refused (see
`mergerfs-lab/transcripts/`).

**Fix:** on a normal machine, load the
module (`modprobe fuse`) and check the
node exists with sane permissions. In
a container, the device must be given
to the container by whoever runs it
(`--device /dev/fuse`, plus the
privileges FUSE needs). Inside a
restricted environment you do not
control, there is no fix — and no
mergerfs setting substitutes for the
device.

## One drive's files appear twice / a stale copy answers

**Cause:** the same relative path
exists on two branches (someone copied
files *between branches* directly, or a
restore landed beside the original),
and the `search` policy picks one to
show — `ff` picks the first branch's
copy, always. The other copy is real,
invisible, and waiting to confuse a
future `newest` policy or a branch
removal.

**Fix:** search the branches, not the
pool: `find /mnt/disk* -name 'the-file'`.
Decide which copy is true, delete the
other *from its branch*, and let this
be the standing lesson: maintain data
through the pool, or through the
branches, and know which you are doing.

## Performance feels wrong

**Causes worth checking before
tuning:** a branch on a failing or
sleeping drive makes the whole pool
feel haunted; `cache.files` left at a
default your workload hates; `df`-heavy
tools polling a pool whose statfs
caching is off; or the honest FUSE
overhead of chapter 13 meeting a
many-small-files workload.

**Fix:** time the same operation on a
branch directly. If the branch is slow,
it was never mergerfs. If only the pool
is slow, change one cache option
(chapter 8), re-time, and write down
what you found — folklore is not
tuning.
