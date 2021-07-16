# polybar-dwm-module
![DWM
Module](https://github.com/mihirlad55/polybar-dwm-module/blob/master/dwm-module.png)

**polybar-dwm-module** is a fork of
[polybar](https://github.com/polybar/polybar) which implements a dwm module.


## Requirements
* dwm with the [IPC patch](https://github.com/mihirlad55/dwm-ipc) applied
* [dwmipcpp](https://github.com/mihirlad55/dwmipcpp) C++ client library for
  communicating with dwm (included as a submodule).
* [jsoncpp](https://github.com/open-source-parsers/jsoncpp) for polybar and
  dwmipcpp (required by module).

The [dwm-anybar patch](https://github.com/mihirlad55/dwm-anybar) is optionally
recommended for a better experience. This patch allows dwm to manage polybar and
fixes some weird quirks that you may experience without it.


## The DWM Module
The dwm module currently supports the following:
- Labels:
    * Display dwm tags
        - Separator label between tags
    * Display the current layout
    * Display the currently focused window title (per monitor)
    * Display label when focused window is floating
- Click Handlers:
    * Tags:
        - Left-click tag to view tag
        - Right-click tag to toggle view on tag
        - Scroll tags to view different tags (with wrapping, reverse scroll, and
          empty tag scroll)
    * Layout:
        - Left-click to set `secondary-layout` (specified in config)
        - Right-click to set previous layout
        - Scroll to cycle through layouts (with wrapping and reverse scroll)
- Different formatting for different tag states:
    * Focused: selected tag on selected monitor
    * Unfocused: selected tag on unselected monitor
    * Visible: unselected, but occupied tag on any monitor
    * Urgent: Tag with window that has the urgent hint set
    * Empty: Unselected and unoccupied tags
- The combined power of polybar


## How to Install
First, apply all the patches you want on dwm, saving the IPC patch for last.

Optionally, apply the [dwm-anybar
patch](https://github.com/mihirlad55/dwm-anybar) and make sure your `config.h`
contains the following
```
static const int showbar            = 1;        /* 0 means no bar */
static const int topbar             = 1;        /* 0 means bottom bar */
static const int usealtbar          = 1;        /* 1 means use non-dwm status bar */
static const char *altbarclass = "Polybar";     /* Alternate bar class name */
static const char *altbarcmd  = "$HOME/bar.sh"; /* Alternate bar launch command */
```
If your polybar is to be displayed on the bottom of the monitor, set `topbar`
to `0`.

Next, apply the [IPC patch](https://github.com/mihirlad55/dwm-ipc). There will
likely be merge conflicts. The IPC patch is mostly additive, so in most conflict
cases, you will be keeping both changes.

After applying all your patches, make sure you compile and install dwm
```
$ sudo make install
```

Make sure you have `jsoncpp` installed, and any other requirements from
[polybar](https://github.com/polybar/polybar).

Arch linux users can install the
[`polybar-dwm-module`](https://aur.archlinux.org/packages/polybar-dwm-module/)
package from the AUR.

Otherwise you can clone, make, and install polybar yourself. Follow the on
screen prompts in the `build.sh` script and enable any additional features you
want.
```
$ git clone https://github.com/mihirlad55/polybar-dwm-module
$ cd polybar-dwm-module
$ ./build.sh -d
```

Configure the bar!  You can view `/usr/share/doc/polybar/config` for a sample config that
includes the supported settings for the dwm module.

**IF YOU APPLIED THE ANYBAR PATCH**, make sure you have
`override-redirect = false` in your polybar config.

**IF YOU DID NOT APPLY THE ANYBAR PATCH**, make sure you have
`override-redirect = true` in your polybar config. You will have to set the bar
height in your dwm's `config.h` to match polybar's height and make sure
`showbar` is set to `1` in your `config.h`. Also make sure `topbar` is set to
the correct value based on if your polybar is a bottom/top bar.

## Launching Polybar
There are multiple ways to launch polybar. Whatever way you decide to use, make
sure polybar launches a few seconds after dwm, else dwm's socket will not be
initialized and the module will fail:

### Anybar Method
The `altbarcmd` variable can be set to the bar launch script. When you apply the
anybar patch, make sure `spawnbar()` occurs in the `setup()` function after
`setupepoll()` and this will ensure that the bar is launched after dwm is ready.
Also make sure that the script checks for running bars before launching more,
else if dwm is restarted, duplicate bars will be launched.

### Hacky Delay Method
In your `.xsession` script or whatever script launches at startup, add something
like the following:
```
$(sleep 4s && "$HOME/launch-polybar.sh") &
```
This will start your polybar script after a 4 second delay which should give dwm
enough time to initialize before polybar tries to connect to dwm. If this
doesn't work, try a longer delay.

### Autostart Method
Apply the [autostart patch](https://dwm.suckless.org/patches/autostart/). In
your `autostart_blocking.sh`, add your command to launch polybar.


## Sample Module Configuration
```ini
...

modules-left = ... dwm ...

...

[module/dwm]
type = internal/dwm
format = <label-tags> <label-layout> <label-floating> <label-title>
; Path to dwm socket (default: /tmp/dwm.sock)
; socket-path = /tmp/dwm.sock

; Left-click to view tag, right-click to toggle tag view
enable-tags-click = false
; Scroll to cycle between available tags
enable-tags-scroll = false
; If true and enable-tags-scroll = true, scrolling will view all tags regardless if occupied
tags-scroll-empty = false
; If true and enable-tags-scroll = true, scrolling will cycle through tags backwards
tags-scroll-reverse = false
; If true and enable-tags-scroll = true, wrap active tag when scrolling
tags-scroll-wrap = false
; Left-click to set secondary layout, right-click to switch to previous layout
enable-layout-click = false
; Scroll to cycle between available layouts
enable-layout-scroll = false
; Wrap when scrolling and reaching beginning/end of layouts
layout-scroll-wrap = false
; Reverse scroll direction
layout-scroll-reverse = false

; If enable-layout-click = true, clicking the layout symbol will switch to this layout
secondary-layout-symbol = [M]

; Separator in between shown tags
; label-separator = |

; Title of currently focused window
; Available tokens:
;   %title%
label-title = %title%
label-title-padding = 2
label-title-foreground = ${colors.primary}
label-title-maxlen = 30

; Defaults to 'Desktop'
label-title-default = "Desktop"

; Symbol of current layout
; Available tokens:
;   %symbol%
label-layout = %symbol%
label-layout-padding = 2
label-layout-foreground = #000
label-layout-background = ${colors.primary}

; Text to show when currently focused window is floating
label-floating = F

; States: focused, unfocused, visible, urgent, empty
; Available tokens:
;   %name%

; focused = Selected tag on focused monitor
label-focused = %name%
label-focused-background = ${colors.background-alt}
label-focused-underline= ${colors.primary}
label-focused-padding = 2

; unfocused = Unselected tag on unselected monitor
label-unfocused = %name%
label-unfocused-padding = 2

; visible = Unselected tag, but occupied tag on any monitor
label-visible = %name%
label-visible-background = ${self.label-focused-background}
label-visible-underline = ${self.label-focused-underline}
label-visible-padding = ${self.label-focused-padding}

; urgent = Unselected tag with window that has urgency hint set
label-urgent = %name%
label-urgent-background = ${colors.alert}
label-urgent-padding = 2

; empty = Unselected and unoccupied tag
; This can be set to an empty string to hide empty tags
label-empty = %name%
label-empty-background = ${colors.primary}
label-empty-padding = 2
```

You can also take a look at my
[personal](https://github.com/mihirlad55/dotfiles/blob/arch-dwm/.config/polybar/config#L99)
bar configuration which is more representative of the screenshot above.


## License
Polybar is licensed under the MIT license. [See LICENSE for more
information](https://github.com/polybar/polybar/blob/master/LICENSE).
