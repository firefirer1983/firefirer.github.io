---
title: Ubuntu Settings
date: 2018-05-07 09:57:43
tags:
categories:
- ubuntu settings
---
# Ubuntu Settings
## customize resolution with xrand
    cvt 1848 998 60   // 查看 1848x998@60hz 的设置信息
    # 1848x998 59.94 Hz (CVT) hsync: 62.09 kHz; pclk: 152.50 MHz
    Modeline "1848x998_60.00"  152.50  1848 1960 2152 2456  998 1001 1011 1036 -hsync +vsync
    xrandr --newmode "1848x998_60.00"  152.50  1848 1960 2152 2456  998 1001 1011 1036 -hsync +vsync
    xrandr --addmode Virtual1 "1848x998_60.00"
    xrandr --output Virtual1 --mode "1848x998_60.00"
    最后将后面3条xrandr语句写入到 ~/.xprofile就可以
