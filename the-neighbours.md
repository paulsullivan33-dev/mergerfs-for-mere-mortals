# The Neighbours

MergerFS is one answer among many to
"my files are spread across drives."
These are the neighbours, and what each
trades. Nothing here is a ranking; the
right tool is the one whose failure
mode you can live with.

| Tool | Shape | Choose it when |
|---|---|---|
| **MergerFS** | Userspace (FUSE) union of plain folders; files whole on one drive | Your drives differ, arrive one at a time, and must stay readable on their own |
| **RAID 0** | Kernel striping across drives | You want one fast volume and accept that one dead drive takes everything |
| **RAID 1 / 5 / 6** (mdadm) | Mirroring or parity in the kernel | You want redundancy *and* one volume, with matched drives and rebuilds you understand |
| **LVM** | Volume manager: pools physical drives into logical volumes, striping or linear | You want resizing and snapshots more than per-file placement, on drives you'll manage as a set |
| **ZFS** | Filesystem and volume manager in one; pools (vdevs) with checksums and parity | You are building the storage properly from the start and can give it the drives (and RAM) it asks for |
| **SnapRAID** | Scheduled parity over standalone drives | Your data changes slowly and you want drive-level protection; the classic partner *under* mergerfs, not a rival |
| **UnionFS / OverlayFS** | Kernel union mounts, built for container image layers | You are stacking a writable layer over read-only ones — images, live systems — not pooling data drives |
| **Windows: Storage Spaces / DrivePool** | The platform equivalents | Your drives live on a Windows box; DrivePool in particular is mergerfs's closest cousin |
| **Just mount the drives and keep a list** | No software at all | Honestly, for two drives and a tidy mind — until the third drive arrives |

MergerFS's own trade is visible in the
table: it is the only entry that leaves
every drive independently readable and
demands nothing of the drives — no
matching sizes, no formatting, no
commitment — and the only cost it
refuses to hide is that it protects
nothing. RAID and ZFS make the opposite
trade: real redundancy, real lock-in,
drives that mean nothing alone.

Two honest notes. Moving *between*
these is mercifully easy in one
direction: leaving mergerfs means
unmounting it and owning some folders —
an afternoon, not a migration. Moving
*to* it from a striped array is the
usual backup-restore project. And the
question every row of the table must
answer is chapter 10's: when a drive
dies, what exactly do you lose, and
what exactly brings it back?
