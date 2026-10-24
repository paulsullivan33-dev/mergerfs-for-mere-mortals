# Recipes

Four pools people actually build, with
the settings that make each one behave.
All assume the install of chapter 3 and
the fstab shape of chapter 7.

## 1. The media library

*The job:* several drives of films,
music and photos behind one folder the
media server reads.

```
branches=/mnt/disk1:/mnt/disk2:/mnt/disk3
category.create=epmfs
minfreespace=20G
moveonenospc=true
allow_other,use_ino
cache.files=partial,dropcacheonclose=true
```

Why: `epmfs` keeps a film's folder —
and a show's whole run — on one drive
(chapter 5); `moveonenospc` rescues the
occasional file still downloading when
its drive tops out; read-mostly
workloads are where `cache.files=partial`
earns its place. Pair with parity
(chapter 10) before the library is
bigger than your willingness to
re-rip it.

## 2. The download landing pad

*The job:* one incoming folder fed by
download tools, emptied by whatever
sorts the results into libraries.

```
branches=/mnt/fast:/mnt/spare
category.create=ff
minfreespace=10G
moveonenospc=true
```

Why: `ff` fills the fast drive first
and only spills to the spare when it
must — order matters more than balance
here. Keep the *sorted* libraries out
of this pool; the pad is scratch space
with good manners, and chapter 9's
hard-link notes apply if the sorter
links finished files into a library on
the same pool.

## 3. The growing photo archive

*The job:* a family archive that gains
a drive every few years and must never
scatter one year's photos across all
of them.

```
branches=/mnt/photos1:/mnt/photos2
category.create=epff
minfreespace=50G
```

Why: `epff` — existing-path, then
first fit — fills the older drive
completely before the new one takes a
single new folder, and every folder
still lives whole on one drive. When
photos1 fills, photos2 inherits the
*future*, not fragments of the past.
Back this one up off-box (chapter 10);
archives are the files you cannot
re-create.

## 4. The everything drawer

*The job:* one big unglamorous folder
for a household's miscellany — ISOs,
installers, exports, "put it in the
drawer" — where convenience is the
entire point.

```
branches=/mnt/disk1:/mnt/disk2
category.create=mfs
minfreespace=5G
allow_other
```

Why: bare `mfs` spreads by emptiness;
nobody cares which drive holds an ISO,
and even wear is a small kindness to
old drives. The drawer's risk is not
placement but amnesia: label it scratch
in everyone's mind, and keep anything
irreplaceable out of it or backed up
from it.

## A recipe deliberately missing

There is no "the safe archive" recipe
built on mergerfs alone, on purpose.
Pools present files; they do not
protect them. If a recipe above tempts
you to skip chapter 10, reread
chapter 10.
