# Runtime Tuning

Most FUSE filesystems treat their mount
options as carved in stone at mount
time. MergerFS treats them as the
*starting* state. A running pool can be
inspected and reconfigured through
extended attributes (xattrs) on the
mount point — no unmount, no
interruption, no asking running
programs to let go of their files.

## Reading the live configuration

The pool's control interface is a
family of `user.mergerfs.*` extended
attributes. On current releases you
address them on the mount point itself;
on the 2.33 series — Ubuntu 24.04's
package, and the version this guide's
lab installed — you address them on a
hidden pseudo-file, **`.mergerfs`**, at
the pool's root (it does not appear in
listings, but it can be `stat`ed and
given xattrs; upstream has marked the
pseudo-file deprecated in favour of the
mount point, with removal planned in
3.0). Either way the keys are the same:

```
$ getfattr -d /pool/.mergerfs        # 2.33 form; current releases: /pool
user.mergerfs.branches="/mnt/disk1=RW:/mnt/disk2=RW"
user.mergerfs.category.create="epmfs"
user.mergerfs.minfreespace="4294967296"
user.mergerfs.moveonenospc="false"
…
```

(The example follows the manual's own
listing format, including the `=RW`
branch mode suffixes — branches can be
read-write, read-only, or no-create.
Install the `attr` package for
`getfattr`/`setfattr` if your system
lacks them.) Both views tell the truth
the mount command may no longer
remember — the pool you have, not the
pool you intended.

## Changing things live

Setting an attribute changes the
running pool (examples in the 2.33
pseudo-file form; on current releases
the target is the mount point):

```
# switch the create policy, now
setfattr -n user.mergerfs.category.create -v mfs /pool/.mergerfs

# add a newly installed drive to the pool, now
setfattr -n user.mergerfs.branches -v "+/mnt/disk3" /pool/.mergerfs
```

Branch edits support a small algebra,
spelled out in the manual: a plain
value replaces the list, a leading `+`
adds a branch, a leading `-` removes
one, and `=` sets the list to exactly
what follows. Removing a branch does
not move its files — they simply leave
the pool's view until the branch
returns, still sitting on their drive.

Worth knowing precisely:

- **Changes last until unmount.** The
  xattrs are the daemon's live state,
  not a file. Anything you want kept
  must also be written into fstab or
  the config file (chapter 7), or the
  next reboot quietly reverts it.
- **Not everything is settable.** Some
  options are mount-time only; the
  manual marks which keys the control
  interface accepts. If a `setfattr`
  answers `EOPNOTSUPP` or the value
  silently does not change, that option
  is one of them — remount instead.
- **The pool validates as you go.** A
  mistyped policy name is refused at
  `setfattr` time rather than being
  discovered at the next file creation,
  which makes experimenting on the live
  pool safer than it sounds.

## Cache behaviour: the tuning that actually matters

The option family people reach for
after "it feels slow" is caching —
mergerfs sits between applications and
real filesystems, so two page caches
can stack up:

| Option | Effect in brief |
|---|---|
| `cache.files=off` | No extra caching of file data; best behaved for some workloads, required by nothing |
| `cache.files=partial` | Cache what was actually read/written — the recommended default for media pools |
| `cache.files=full` / `auto-full` | Cache whole files on access; can help repeated reads of small files |
| `dropcacheonclose=true` | Tell the kernel it may drop a file's cached pages when the file closes; pairs with `partial`/`full` to stop double-caching from evicting everything else |

Also in the same drawer: `cache.statfs`
(how long free-space answers are
cached — raise it if `df`-heavy tools
feel sluggish, knowing the numbers go
stale), and the attribute/entry cache
timeouts inherited from FUSE. Honest
guidance: change cache settings one at
a time, with a workload you can repeat,
and keep the numbers you observed. The
defaults plus `cache.files=partial`
are right for most pools; the rest of
this section exists for the day they
are not.

## A safe first experiment

On a pool you can afford to confuse for
a minute: read the full configuration
from `.mergerfs`, change the create
policy to `ff`, create three test files
in a fresh folder and look in the
branches to see them fill in order,
then set the policy back. Ten minutes
that teach more than this page can —
the pool's rules are only real once
you have watched files obey them. (On
the lab bench of chapter 3 this
experiment could not run — the
sandbox had no FUSE device — so treat
it as your first homework on a real
machine, not as a result this guide
measured.)
