# cursedfs

Make a disk image formatted with both ZFS and FAT, at once.

```bash
~/cursedfs% wget 'https://github.com/pcd1193182/cursedfs/releases/download/v1.1/cursed.img'
~/cursedfs% sudo umount mountpoint/
~/cursedfs% sudo mount -o loop -t msdos cursed.img mountpoint/
~/cursedfs% ls mountpoint/
duckroll.jpg
~/cursedfs% sudo zpool import cursed .
~/cursedfs% ls /cursed/
rickroll.jpg
```

# Why?

[I got nerd-sniped](https://twitter.com/NieDzejkob/status/1217221249409191936)

# How?

Bulding on NieDzejkob's great work, since zfs leaves the first 8k of
the disk empty, you can easily have the two coexist. The FAT
filesystem lives in zfs's 3.5M of reserved boot space. This can be
extended with a read-only ext2 filesystem as well; I'll try that
eventually

# Can I write to the filesystems?

Yes! Because FAT reserves the sectors used by ZFS's labels and ZFS
doesn't write to the boot space, both exist in harmony.
