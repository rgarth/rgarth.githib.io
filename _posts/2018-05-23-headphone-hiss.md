---
layout: post
title: White noise in headphones running linux on a Thinkpad T470s
---
This has been bugging me for a while, finally took a look today. Under Linux on
a Thinkpad T470s I was experiencing a whitenoise/hissing sound whenever I had
headphones plugged in, and no app was actively playing sound. It happened under various kernels and distros, and it looks like it happens on other thinkpad
models as well.

The culprit seems to be the snd_hda_codec_realtek kernel mod.

To blacklist it:
```bash
echo blacklist snd_hda_codec_realtek | sudo tee -a /etc/modprobe.d/sound.conf
```

After a reboot the crackle and hiss in your headphones should be gone
