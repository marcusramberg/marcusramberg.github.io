---
date: "2015-01-07T00:00:00Z"
link: http://www.thinkwiki.org/wiki/Buttonless_Touchpad
published: true
tags:
  - from_ios
  - Linux
  - Thinkpad
title: Fixing the Thinkpad Buttonless Touchpad
---

> The default configuration of the xf86-input-synaptics driver makes the clickpad almost unusable (on the T440s). Clicks/Taps do not register properly, misclicking, no palm detection, no soft buttons where they should be, and so on...
> The following configuration can be put in /etc/X11/xorg.conf.d/99-synaptics.conf As a result the clickpad will react a lot more 'calm' than before and the middle & right click button areas match with the printed ones on top of the clickpad.

New laptop, new woes
