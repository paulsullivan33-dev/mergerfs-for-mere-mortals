# Making It Permanent

A pool you mount by hand vanishes at
reboot. Making it permanent means
telling the system, in a file it reads
at boot, the same things you typed at
the command line — and being honest
about ordering: the branch drives must
be mounted *before* the pool that sits
on them.

## fstab: the standard way

Mount the branch drives by UUID in
`/etc/fstab` as usual, then one line
for the pool itself:

```
/dev/disk/by-uuid/…  /mnt/disk1  ext4  defaults  0 2
/dev/disk/by-uuid/…  /mnt/disk2  ext4  defaults  0 2

/mnt/disk1:/mnt/disk2  /pool  fuse.mergerfs  defaults,allow_other,use_ino,cache.files=partial,dropcacheonclose=true,category.create=mfs,minfreespace=20G,moveonenospc=true  0 0
```

Points that bite people:

- The filesystem type is
  **`fuse.mergerfs`**. The branch list
  goes where the device would go,
  colons and all.
- Add **`x-systemd.requires-mounts-for=/mnt/disk1`**
  (and disk2, and…) on systemd systems
  if there is any chance the pool line
  is processed before the drives are
  mounted. A pool mounted over
  not-yet-mounted branch mount points
  will cheerfully pool the *empty
  directories* on the root filesystem,
  and your files will seem to vanish
  when the real drives mount over them
  later. Chapter 14 has this as a
  symptom; the option is the vaccine.
- **`nofail`** on the pool line is
  worth considering: a missing drive
  then means a missing pool at boot,
  not a boot stopped at an emergency
  prompt. The trade is silence versus
  a loud failure — pick consciously.
- Test before rebooting, always:
  unmount the pool, then `mount /pool`
  and check the branches show through.
  fstab errors found by `mount -a` are
  free; the same errors at 2 a.m. after
  a power cut are not.

**Check:**

```
$ findmnt /pool
TARGET  SOURCE                        FSTYPE  OPTIONS
/pool   /mnt/disk1:/mnt/disk2         fuse.mergerfs  rw,…
```

## The config file

MergerFS also reads an ini-style config
file, named with the `config=` option
or placed where your invocation looks
for one:

```
branches=/mnt/disk1:/mnt/disk2
minfreespace=20G
category.create=mfs
moveonenospc=true
cache.files=partial
```

The same keys work in three places —
the `mergerfs` command line, fstab, and
the config file — because they are one
option language. A tidy pattern is to
keep the fstab line short
(`config=/etc/mergerfs/pool.conf`) and
the real settings in the file, where
they are easier to read, diff, and back
up.

## The options worth setting deliberately

The project's own manual suggests
starting sets rather than a blank page.
For most home pools, the reading list
is short:

| Option | Why |
|---|---|
| `allow_other` | Other users (and the services you run as them) can see the pool at all |
| `use_ino` | mergerfs supplies inode numbers, so hard-linked files report the same inode |
| `cache.files=partial` (and `dropcacheonclose=true`) | Sane page-cache behaviour for large files; `off` if nothing you run needs mmap |
| `category.create=mfs` or `epmfs` | The placement instinct of chapter 5 |
| `minfreespace=…` | Chapter 6's threshold, sized to your largest files |
| `moveonenospc=true` | The mid-write rescue of chapter 6 |

## What "permanent" does not cover

fstab brings the pool back; it does not
check the pool's health. A drive that
fails to mount leaves a pool quietly
missing a branch (see `nofail` above).
If the pool matters, monitor the
branches: their mount points present,
their free space sane (chapter 6), and
the pool answering. That is a cron job
and three lines of shell, and it is the
difference between learning about a
dead drive from your monitoring and
learning about it from a missing film.
