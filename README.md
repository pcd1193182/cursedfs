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