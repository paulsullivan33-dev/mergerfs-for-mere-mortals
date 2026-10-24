# Policies in Practice

A policy is mergerfs's answer to a
question it must answer constantly:
*several drives could take this — which
one?* You do not set one policy; you set
one per **category** of operation, and
you may override individual functions if
you ever need to.

| Category | Question it answers | Default |
|---|---|---|
| **create** | A new file or directory is being made — on which branch? | `epmfs` |
| **search** | Several branches hold this path — which copy answers the lookup? | `ff` |
| **action** | A change is being made (chmod, truncate, delete…) — to which copies? | `epall` |

Set them at mount time:

```
-o category.create=mfs,category.search=ff,category.action=epall
```

or change them later on the running
pool (chapter 8). Individual functions
can be overridden with `func.<name>=`
— `func.create=ff`, say — but most
pools never need finer grain than the
three categories.

## The create policies you will actually meet

| Policy | Rule | Character |
|---|---|---|
| **ff** | First branch with enough free space (first fit) | Fills drives in order; predictable; the later drives stay empty until needed |
| **mfs** | Branch with the **m**ost **f**ree **s**pace | Spreads new files across drives by current emptiness |
| **epmfs** | `mfs`, but **e**xisting-**p**ath first: prefer branches where the parent folder already exists, and only among those pick most-free-space | Keeps a folder's files together on a drive where the folder already lives |
| **epff** | Existing-path, then first fit | Same family instinct as `epmfs`, fills in order within the folder's home |
| **lfs / lus** | Least free space / least used space | Deliberately fills the fullest drive first — for herding data onto drives you plan to retire |
| **rand / eprand** | Random branch | Simple spreading when you trust nothing else |
| **erofs** | Not a placement rule at all: answers every such operation with EROFS ("read-only filesystem") | A way to forbid a whole function category outright |

And the search/action regulars:

| Policy | Rule |
|---|---|
| **ff** (search) | The first branch holding the path answers |
| **newest** (search) | The copy with the newest modification time answers |
| **all / epall** (action) | Apply the change to every copy — `epall` prefers branches where the path exists (in practice, the same set) |

## Existing path: the two letters that matter most

Watch what `ep` does with a new file,
`/pool/music/new-album/track1.flac`,
when `music/new-album` already exists
on disk2 only:

- **`epmfs`** — candidates are branches
  where `music/new-album` exists: just
  disk2. The track lands beside its
  album. Whole albums live on one drive,
  which is what you want when a drive is
  unmounted, browsed alone, or dies.
- **`mfs`** (no `ep`) — candidates are
  all branches; the emptiest wins, album
  be scattered. The pool works fine, but
  no single drive holds the album.

**Check your instinct:** if a branch
were removed tomorrow, would you rather
find complete folders on the survivors
or a tenth of every folder on each?
That answer chooses between `ep*` and
bare policies for most home pools, and
it is why `epmfs` is the create default.

## Path preservation and renames

Create policies marked path-preserving
(the `ep` family) also shape **renames
and hard links**: a path-preserving
rename keeps the file on its branch if
the destination directory exists there,
and *moves it between branches* if
that is what keeping the path rule
requires; a non-path-preserving rename
never crosses branches and fails with
`EXDEV` ("Invalid cross-device link")
when a program insists. Programs that
rename across folders (download tools
moving finished files into the library
are the classic) behave differently
under each — chapter 14 has the
symptoms. The mount option
`ignorepponrename=true` turns renames
and links non-path-preserving without
changing the create policy.

## What the defaults assume

The shipped defaults — create `epmfs`,
search `ff`, action `epall`,
minfreespace 4G — describe the pool the
author expects: media-ish files, folders
that belong together, drives of uneven
size where the emptiest should absorb
growth. Change one at a time, watch
where files land with the branches
themselves (`ls` the real folders, not
the pool), and only then change another.
