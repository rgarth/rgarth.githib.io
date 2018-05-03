---
layout: post
title: Corrupted drive, missing superblock
---
I was passed a currupt LVM volume today and asked to recover it. It was completely screwed. “dumpe2fs” could not give me a list of superblocks for me to pass to fsck.

I could probably calculate where the superblocks should be on the filesystem by hand if I needed to, but it was much simpler than that:

```# mkfs.ext3 -n```

The -n is a dry run. It will show me what would happen but not actually touch the drive. This will also show you where the superblocks should be. Pass one of these to you fsck command. If the previous partition had been created with a set of unique options this may not work. But your filesystem’s hosed anyway so it may be worth the punt.

If your filesystem is completely screwed though you may recover little. And what you do recover will probably be sitting in “lost+found”, fsck will recover your filesystem but it makes no guarantee about recovering files.
