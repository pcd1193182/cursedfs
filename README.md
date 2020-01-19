# cursedfs

Make a disk image formatted with ZFS, ext2, and FAT all at once.

```bash
~/cursedfs% wget 'https://github.com/pcd1193182/cursedfs/releases/download/v2.0/cursed.img'
~/cursedfs% sudo mount -o loop -t msdos cursed.img mountpoint/
~/cursedfs% ls mountpoint/
duckroll.jpg
~/cursedfs% sudo umount mountpoint
~/cursedfs% sudo mount -o loop -t ext2 cursed.img mountpoint/
~/cursedfs% ls mountpoint/
rickroll.jpg
~/cursedfs% sudo zpool import cursed -d .
~/cursedfs% ls /cursed/
rickroll.mp4
```

# Why?

[I got nerd-sniped](https://twitter.com/NieDzejkob/status/1217221249409191936)

# How?

FAT uses the first 512 bytes to store its superblock. ext2 stores its
metadata starting at the third 512 bytes. ZFS stores its metadata
starting at the 8k marker. By using a program that creates an ext2
filesystem that marks many of its blocks unusable, all three of these
can be made to not overlap their metadata, allowing them to live in
harmony. This builds on NieDzejkob's great work. The ext2 and FAT
filesystems live inside ZFS's 3.5MiB boot block space; the ext2
filesystem lives in the first 512KiB of it, and the FAT filesystem
lives in the next 3 MiB.

# Can I write to the filesystems?

Yes! Because FAT and ext2 reserve the sectors used by ZFS's labels,
and FAT reserves all space used by ext2, and ZFS doesn't write to the
boot space, all three exist in harmony.

# Notes

You will need to use the [mkext2
utility](https://github.com/pcd1193182/mkext2) to build your cursed
filesystem, since it knows how to reserve blocks when creating an ext2
filesystem.

# FAQs

 * What about Btrfs or NTFS or <insert filesystem here>?
  
There are a number of restrictions that we have to consider when trying to add a
new filesystem. First, the critical metadata needs to not overlap; if the superblock
lives in one of the sectors we're already using or can't avoid using, we're sunk.
Second, the filesystem needs some way to reserve some space that it won't write to
where our other filesystems can live. Basically all filesystems can reserve space
beyond a certain point; if you create the disk image at a small size and then expand
it, the filesystem won't write beyond where it thinks the end of the image is. But
you can't overlay more than two filesystems without a space reservation capability.

Btrfs's first metadata is at 64 KiB, which leaves plenty of space at the start for
other FS's metadata (though not ZFS, the two couldn't coexist). But I don't know of
a mechanism by which it can reserve space later in the disk where the other
filesystems could store their data, so unless we wanted to share 64KiB among all the
other mounts, we can't use Btrfs.

NTFS uses the first sector for its superblock, so it's not compatible with FAT.

APFS uses the first sector for its superblock, so it's not compatible with FAT.
In addition, I believe the superblock is larger than a KiB, so it also impinges
on ext2's superblock.

 * I'm confused about how this works! How do the filesystems know what space they can
and can't use?

First, each filesystem is created on its own disk image of a specific size.
In the given example, the FAT filesystem is created on a 4M filesystem, and
the ext2 filesystem is 2M. As a result, neither will try to write beyond that
point, so the main body of the ZFS storage space is safe from them.

Second, each filesystem has been created with some space at the start (after their
superblocks and other critical metadata) marked as reserved, so they won't
write to it. For ext2, that's the next 512KiB. For FAT, it's the next MiB.
This protects the ZFS labels from both the ext2 and FAT filesystem. It also
protects the ext2 filesystem from the FAT filesystem, since it lives entirely
in that reserved MiB where FAT won't try to allocate any data. Finally, the FAT
and ext2 filesystem are protected from ZFS writes because they live entirely in
its reserved boot block and before the first label, where ZFS won't try to issue
any writes.
