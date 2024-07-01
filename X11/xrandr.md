# X11

### Fix missing display resolutions

If you don't know what these commands do it's recommmended you read the instructions below.

#### Commands
```
# My Framework 13 laptop is 2256x1504 @ 60Hz.  Make sure you have the correct values for your display.

# Use GTF to get VESA settings
gtf 2256 1504 60  # Using 2256x1504 resolution at 60Hz

# Create new mode via xrandr
xrandr --newmode "2256x1504_60.00"  288.30  2256 2424 2672 3088  1504 1505 1508 1556  -HSync +Vsync

# Set display to use new mode
xrandr --addmode Virtual-1 "2256x1504_60.00"

```

### Introduction

Sometimes when I set up VMs on a new machine I find the resolution I need isn't listed.  I ran into this with my Framework 13 which has a 2256x1504 resolution with an Ubuntu Guest machine.   I was able to get the supported resolution added manually via xrandr.  Here is how you do this but take care you are using supported resolutions and refresh rates.

### 1 - Calculate VESA with `gtf` 

So you know you the resolution and refresh rate you want but xrandr is going to need information like the pixel refrequency, total pixels, total vertical lines, etc.  The easiest way to go this information is to use `gtf` which given vertical, horizontal and refresh rate, output in what xorg needs (gtf outputs in xorg mode by default).

```
gtf 2256 1504 60  # Using 2256x1504 resolution at 60Hz

# 2256x1504 @ 60.00 Hz (GTF) hsync: 93.36 kHz; pclk: 288.30 MHz
  Modeline "2256x1504_60.00"  288.30  2256 2424 2672 3088  1504 1505 1508 1556  -HSync +Vsync
```

If you want to know a little more about somem of these values you can pass the verbose flag.  We can the 288.30 comes from the PIXEL FREQNECY 288.295685 for example.
```
gtf 2256 1504 60 -v
 1: [H PIXELS RND]             :     2256.000000
 2: [V LINES RND]              :     1504.000000
 3: [V FIELD RATE RQD]         :       60.000000
 4: [TOP MARGIN (LINES)]       :        0.000000
 5: [BOT MARGIN (LINES)]       :        0.000000
 6: [INTERLACE]                :        0.000000
 7: [H PERIOD EST]             :       10.708749
 8: [V SYNC+BP]                :       51.000000
 9: [V BACK PORCH]             :       48.000000
10: [TOTAL V LINES]            :     1556.000000
11: [V FIELD RATE EST]         :       60.013874
12: [H PERIOD]                 :       10.711226
13: [V FIELD RATE]             :       60.000000
14: [V FRAME RATE]             :       60.000000
15: [LEFT MARGIN (PIXELS)]     :        0.000000
16: [RIGHT MARGIN (PIXELS)]    :        0.000000
17: [TOTAL ACTIVE PIXELS]      :     2256.000000
18: [IDEAL DUTY CYCLE]         :       26.786633
19: [H BLANK (PIXELS)]         :      832.000000
20: [TOTAL PIXELS]             :     3088.000000
21: [PIXEL FREQ]               :      288.295685
22: [H FREQ]                   :       93.360001
17: [H SYNC (PIXELS)]          :      248.000000
18: [H FRONT PORCH (PIXELS)]   :      168.000000
36: [V ODD FRONT PORCH(LINES)] :        1.000000
```


### 2 - Create the new mode (resolution) 

So gtf did a lot of work for us and gave us the values we need for `xrandr`.

```
xrandr --newmode "2256x1504_60.00"  288.30  2256 2424 2672 3088  1504 1505 1508 1556  -HSync +Vsync
```

This will create a new mode called "2256x1504_60.00".  You can see all your current options via `xrandr` by using `-q`

```
xrandr -q
Screen 0: minimum 320 x 200, current 2256 x 1504, maximum 8192 x 8192
Virtual-1 connected primary 2256x1504+0+0 (normal left inverted right x axis y axis) 325mm x 203mm
   1280x800      74.99 +
   5120x2160     50.00  
   4096x2160     50.00  
   3840x2160     60.00    50.00    59.94  
   ...
   1856x1392     60.00  
   1792x1344     60.00  
   1024x768      60.00*  
   800x600       60.32  
   640x480       60.00    59.94  
   2256x1504_60.00  60.00      # <---- This is the new mode
```


### 3 - Add the mmode to your display
So once the new mode has been created you need to add it to your display.  The below commmand adds the "2256x1504_60.00" mode we created in step 2, to my display named `Virtual-1`.  Note that your display name may be different.  You can use `xrand -q` which should output all your connected displays.

```
xrandr --addmode Virtual-1 "2256x1504_60.00"
```

Once you have added the mode you should be able to select it via display settings.  You can also set it via xrand if you want.

I should mentipon I still had issues with this in my Ubuntu guest.  I had to disable Wayland by editing
`/etc/gdm3/custom.conf` and uncommmenting the line
```
#WaylandEnable=false
```



### Persisting these changes
These changes may not persit after a reboot.  The easiest way to make this persist is via `.xprofile` 

Create a file called `.xprofile` in your home directory  and add in your xrand commands.  
```
cat ~/.xprofile 
xrandr --newmode "2256x1504_60.00"  288.30  2256 2424 2672 3088  1504 1505 1508 1556  -HSync +Vsync
xrandr --addmode Virtual-1 "2256x1504_60.00"
```

There are other ways to do this but this is the simplest.