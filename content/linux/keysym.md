---
title: 从键盘到字符
date: 2024-12-17
summary: linux下从键盘收到信号后到屏幕上显示字符的过程
tags:
  - hardware
status: in-progress
---

# 从键盘到字符

## 前置知识

参考[archwiki](https://wiki.archlinux.org/title/Keyboard_input)

- 硬件，即键盘本身，发出扫描码（scancode），发送给内核；
- 内核并将其转换为键码（keycode）。键码是操作系统用来识别按键的数字表示。
- 受到键码后，X服务器或Wayland合成器将其转换为键符（keysym）。键符是一个更高级别的表示，表示按键的实际字符或功能。

在xorg下，有[x keyboard extension](https://wiki.archlinux.org/title/X_keyboard_extension)来使用keycode

## 识别

xorg下管理接入的输入设备，例如 find id of touchpad:

```
xinput list | grep -i "Touchpad" | awk '{print $6}' | sed 's/[^0-9]//g'
```

似乎需要添加文件 `/usr/share/X11/xorg.conf.d/30-touchpad.conf`

```conf
Section "InputClass"
        Identifier "MyTouchpad"
        MatchIsTouchpad "on"
        Driver "libinput"
        Option "Tapping" "on"
EndSection
```

之后才能正常识别`touchpad`：

```xinput list

⎡ Virtual core pointer                    	id=2	[master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer              	id=4	[slave  pointer  (2)]
⎜   ↳ GXTP7863:00 27C6:01E0 Mouse             	id=11	[slave  pointer  (2)]
⎜   ↳ GXTP7863:00 27C6:01E0 Touchpad          	id=12	[slave  pointer  (2)]
⎜   ↳ HS6209 2.4G Wireless Receiver Keyboard  	id=9	[slave  pointer  (2)]
⎜   ↳ HS6209 2.4G Wireless Receiver Mouse     	id=10	[slave  pointer  (2)]
⎣ Virtual core keyboard                   	id=3	[master keyboard (2)]
    ↳ Virtual core XTEST keyboard             	id=5	[slave  keyboard (3)]
    ↳ Video Bus                               	id=6	[slave  keyboard (3)]
    ↳ Power Button                            	id=7	[slave  keyboard (3)]
    ↳ Huawei WMI hotkeys                      	id=13	[slave  keyboard (3)]
    ↳ AT Translated Set 2 keyboard            	id=14	[slave  keyboard (3)]
```

否则只会识别两个设备，没有 `Mouse` `Touchpad`

```sh
$ xinput list

⎡ Virtual core pointer                    	id=2	[master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer              	id=4	[slave  pointer  (2)]
⎜   ↳ GXTP7863:00 27C6:01E0                   	id=10	[slave  pointer  (2)]
⎜   ↳ GXTP7863:00 27C6:01E0                   	id=11	[slave  pointer  (2)]
⎣ Virtual core keyboard                   	id=3	[master keyboard (2)]
    ↳ Virtual core XTEST keyboard             	id=5	[slave  keyboard (3)]
    ↳ Video Bus                               	id=6	[slave  keyboard (3)]
    ↳ Power Button                            	id=7	[slave  keyboard (3)]
    ↳ Huawei WMI hotkeys                      	id=12	[slave  keyboard (3)]
    ↳ AT Translated Set 2 keyboard            	id=13	[slave  keyboard (3)]
```

`xev`: find keycode;

## 修改

### xorg

支持的修改可以参考 `/usr/share/X11/xkb/rules/base.lst`

```
  caps                 Caps Lock behavior
  caps:internal        Caps Lock uses internal capitalization; Shift "pauses" Caps Lock
  caps:internal_nocancel Caps Lock uses internal capitalization; Shift does not affect Caps Lock
  caps:shift           Caps Lock acts as Shift with locking; Shift "pauses" Caps Lock
  caps:shift_nocancel  Caps Lock acts as Shift with locking; Shift does not affect Caps Lock
  caps:capslock        Caps Lock toggles normal capitalization of alphabetic characters
  caps:shiftlock       Caps Lock toggles Shift Lock (affects all keys)
  caps:swapescape      Swap Esc and Caps Lock
  caps:escape          Make Caps Lock an additional Esc
  caps:escape_shifted_capslock Make Caps Lock an additional Esc, but Shift + Caps Lock is the regular Caps Lock
  caps:backspace       Make Caps Lock an additional Backspace
  caps:return          Make Caps Lock an additional Return key
  caps:super           Make Caps Lock an additional Super
  caps:hyper           Make Caps Lock an additional Hyper
  caps:menu            Make Caps Lock an additional Menu key
  caps:numlock         Make Caps Lock an additional Num Lock
  caps:ctrl_shifted_capslock Make Caps Lock an additional Ctrl and Shift + Caps Lock the regular Caps Lock
  caps:ctrl_modifier   Make Caps Lock act as an additional Ctrl modifier, but keep identifying as Caps Lock
  caps:digits_row      Caps Lock gives digits on the digits row (Azerty layouts)
  caps:digits_row_independent_lock Shift + Caps locks the digits on the digits row, Caps Lock alone behaves as usual (Azerty layouts)
  caps:none            Caps Lock is disabled

  ctrl                 Ctrl position
  ctrl:nocaps          Caps Lock as Ctrl
  ctrl:lctrl_meta      Left Ctrl as Meta
  ctrl:swapcaps        Swap Ctrl and Caps Lock
  ctrl:grouptoggle_capscontrol Caps Lock as Ctrl,  Left Control switches to another layout
  ctrl:hyper_capscontrol Caps Lock as Ctrl, Ctrl as Hyper
  ctrl:ac_ctrl         To the left of "A"
  ctrl:aa_ctrl         At the bottom left
  ctrl:rctrl_ralt      Right Ctrl as Right Alt
  ctrl:ralt_rctrl      Right Alt as Right Control
  ctrl:menu_rctrl      Menu as Right Ctrl
  ctrl:swap_lalt_lctl  Swap Left Alt with Left Ctrl
  ctrl:swap_ralt_rctl  Swap Right Alt with Right Ctrl
  ctrl:swap_lwin_lctl  Swap Left Win with Left Ctrl
  ctrl:swap_rwin_rctl  Swap Right Win with Right Ctrl
  ctrl:swap_lalt_lctl_lwin Left Alt as Ctrl, Left Ctrl as Win, Left Win as Left Alt
```

#### xmodmap

```man

XMODMAP(1)                  General Commands Manual                  XMODMAP(1)

NAME
       xmodmap - utility for modifying keymaps and pointer button mappings in X

SYNOPSIS
       xmodmap [-options ...] [filename]

DESCRIPTION
       The  xmodmap  program  is used to edit and display the keyboard modifier
       map and keymap table that are used by  client  applications  to  convert
       event  keycodes into keysyms.  It is usually run from the user's session
       startup script to configure the keyboard according to personal tastes.

OPTIONS
       The following options may be used with xmodmap:

       -display display
               This option specifies the host and display to use.

       -help   This option indicates that a brief description  of  the  command
               line  arguments should be printed on the standard error channel.
               This will be done whenever an unhandled  argument  is  given  to
               xmodmap.

       -grammar
               This option indicates that a help message describing the expres‐
               sion  grammar  used  in  files and with -e expressions should be
               printed on the standard error.

       -version
               This option indicates that xmodmap should print its version  in‐
               formation and exit.

       -verbose
               This option indicates that xmodmap should print logging informa‐
               tion as it parses its input.

       -quiet  This option turns off the verbose logging.  This is the default.

       -n      This  option  indicates  that xmodmap should not change the map‐
               pings, but should display what it would do,  like  make(1)  does
               when given this option.

       -e expression
               This  option specifies an expression to be executed.  Any number
               of expressions may be specified from the command line.

       -pm     This option indicates that the current modifier  map  should  be
               printed  on  the  standard output.   This is the default mode of
               operation if no other mode options are specified.

       -pk     This option indicates that the current keymap  table  should  be
               printed on the standard output.

       -pke    This  option  indicates  that the current keymap table should be
               printed on the standard output in the form of  expressions  that
               can be fed back to xmodmap.

       -pp     This  option  indicates  that  the current pointer map should be
               printed on the standard output.

       -       A lone dash means that the standard input should be used as  the
               input file.

       The  filename specifies a file containing xmodmap expressions to be exe‐
       cuted.  This file is usually kept in the user's home  directory  with  a
       name like .xmodmaprc.

EXPRESSION GRAMMAR
       The  xmodmap program reads a list of expressions and parses them all be‐
       fore attempting to execute any of them.  This makes it possible to refer
       to keysyms that are being redefined in a natural way without  having  to
       worry as much about name conflicts.

       The   list   of   keysym   names   may  be  found  in  the  header  file
       <X11/keysymdef.h> (without the XK_ prefix).   Keysyms  matching  Unicode
       characters  may  be  specified  as  "U0020"  to  "U007E"  and "U00A0" to
       "U10FFFF" for all possible Unicode characters.

       keycode NUMBER = KEYSYMNAME ...
               The list of keysyms is assigned to the indicated keycode  (which
               may  be specified in decimal, hex or octal and can be determined
               by running the xev program).  Up to eight  keysyms  may  be  at‐
               tached to a key, however the last four are not used in any major
               X server implementation.  The first keysym is used when no modi‐
               fier  key  is  pressed  in conjunction with this key, the second
               with Shift, the third when the Mode_switch key is used with this
               key and the fourth when both the Mode_switch and Shift keys  are
               used.

       keycode any = KEYSYMNAME ...
               If no existing key has the specified list of keysyms assigned to
               it,  a spare key on the keyboard is selected and the keysyms are
               assigned to it.  The list of keysyms may be specified  in  deci‐
               mal, hex or octal.

       keysym KEYSYMNAME = KEYSYMNAME ...
               The KEYSYMNAME on the left hand side is translated into matching
               keycodes  used  to  perform the corresponding set of keycode ex‐
               pressions.  Note that if the same keysym is  bound  to  multiple
               keys, the expression is executed for each matching keycode.

       clear MODIFIERNAME
               This removes all entries in the modifier map for the given modi‐
               fier,  where  valid  name are: Shift, Lock, Control, Mod1, Mod2,
               Mod3, Mod4, and Mod5 (case does not matter  in  modifier  names,
               although  it  does  matter  for  all other names).  For example,
               ‘‘clear Lock'' will remove all any keys that were bound  to  the
               shift lock modifier.

       add MODIFIERNAME = KEYSYMNAME ...
               This adds all keys containing the given keysyms to the indicated
               modifier  map.   The  keysym names are evaluated after all input
               expressions are read to make it easy  to  write  expressions  to
               swap keys (see the EXAMPLES section).

       remove MODIFIERNAME = KEYSYMNAME ...
               This  removes all keys containing the given keysyms from the in‐
               dicated modifier map.  Unlike add, the keysym names  are  evalu‐
               ated  as  the  line  is read in.  This allows you to remove keys
               from a modifier without having to worry  about  whether  or  not
               they have been reassigned.

       pointer = default
               This sets the pointer map back to its default settings (button 1
               generates a code of 1, button 2 generates a 2, etc.).

       pointer = NUMBER ...
               This sets the pointer map to contain the indicated button codes.
               The  list always starts with the first physical button.  Setting
               a button code to 0 disables events from that button.

       Lines that begin with an exclamation point (!) are taken as comments.

       If you want to change the binding of a modifier key, you must  also  re‐
       move it from the appropriate modifier map.

EXAMPLES
       Many  pointers  are designed such that the first button is pressed using
       the index finger of the right hand.  People  who  are  left-handed  fre‐
       quently  find  that  it  is more comfortable to reverse the button codes
       that get generated so that the primary button is pressed using the index
       finger of the left hand.  This could be done on a 3  button  pointer  as
       follows:
       %  xmodmap -e "pointer = 3 2 1"

       Many  applications  support  the notion of Meta keys (similar to Control
       keys except that Meta is held down instead of Control).   However,  some
       servers  do  not  have a Meta keysym in the default keymap table, so one
       needs to be added by hand.  The following command will  attach  Meta  to
       the  Multi-language  key (sometimes labeled Compose Character).  It also
       takes advantage of the fact that applications that need a Meta key  sim‐
       ply  need  to  get the keycode and don't require the keysym to be in the
       first column of the keymap table.  This means that applications that are
       looking for a Multi_key (including the default modifier map)  won't  no‐
       tice any change.
       %  xmodmap -e "keysym Multi_key = Multi_key Meta_L"

       Similarly, some keyboards have an Alt key but no Meta key.  In that case
       the following may be useful:
       %  xmodmap -e "keysym Alt_L = Meta_L Alt_L"

       One  of  the  more simple, yet convenient, uses of xmodmap is to set the
       keyboard's "rubout" key to generate  an  alternate  keysym.   This  fre‐
       quently involves exchanging Backspace with Delete to be more comfortable
       to the user.  If the ttyModes resource in xterm is set as well, all ter‐
       minal emulator windows will use the same key for erasing characters:
       %  xmodmap -e "keysym BackSpace = Delete"
       %  echo "XTerm*ttyModes:  erase ^?" | xrdb -merge

       Some  keyboards do not automatically generate less than and greater than
       characters when the comma and period keys  are  shifted.   This  can  be
       remedied with xmodmap by resetting the bindings for the comma and period
       with the following scripts:
       !
       ! make shift-, be < and shift-. be >
       !
       keysym comma = comma less
       keysym period = period greater

       One of the more irritating differences between keyboards is the location
       of  the  Control  and CapsLock keys.  A common use of xmodmap is to swap
       these two keys as follows:
       !
       ! Swap Caps_Lock and Control_L
       !
       remove Lock = Caps_Lock
       remove Control = Control_L
       keysym Control_L = Caps_Lock
       keysym Caps_Lock = Control_L
       add Lock = Caps_Lock
       add Control = Control_L

       This example can be run again to swap the keys back  to  their  previous
       assignments.

       The  keycode command is useful for assigning the same keysym to multiple
       keycodes.  Although unportable, it  also  makes  it  possible  to  write
       scripts  that  can  reset  the keyboard to a known state.  The following
       script sets the backspace key  to  generate  Delete  (as  shown  above),
       flushes  all  existing  caps  lock bindings, makes the CapsLock key be a
       control key, make F5 generate Escape, and makes Break/Reset be  a  shift
       lock.
       !
       ! On the HP, the following keycodes have key caps as listed:
       !
       !     101  Backspace
       !      55  Caps
       !      14  Ctrl
       !      15  Break/Reset
       !      86  Stop
       !      89  F5
       !
       keycode 101 = Delete
       keycode 55 = Control_R
       clear Lock
       add Control = Control_R
       keycode 89 = Escape
       keycode 15 = Caps_Lock
       add Lock = Caps_Lock

ENVIRONMENT
       DISPLAY to get default host and display number.

X Version 11                     xmodmap 1.0.11                      XMODMAP(1)
```

xmodmap 修改键码到键符的映射 (`xkeycaps` gui);

usage of `xmodmap`:

modifier for X Window. mod1: left Alt, mod2: Num_Lock，mod3: no，mod4: Left Super (windows)，mod5: Shift

```sh
$ xmodmap -pm

xmodmap:  up to 4 keys per modifier, (keycodes in parentheses):

shift       Shift_L (0x32),  Shift_R (0x3e)
lock        Caps_Lock (0x42)
control     Control_L (0x25),  Control_R (0x69)
mod1        Alt_L (0x40),  Alt_R (0x6c),  Alt_L (0xcc),  Meta_L (0xcd)
mod2        Num_Lock (0x4d)
mod3        ISO_Level5_Shift (0xcb),  Hyper_L (0xcf)
mod4        Super_L (0x85),  Super_R (0x86),  Super_L (0xce)
mod5        ISO_Level3_Shift (0x5c)
```

Keycode is what the kernal get, and keysym is what it gives. Each keysym column in the table corresponds to a particular combination of modifier keys:

1. Key
2. Shift+Key
3. Mode_switch+Key
4. Mode_switch+Shift+Key
5. ISO_Level3_Shift+Key
6. ISO_Level3_Shift+Shift+Key

check current keycode map:

```
$ xmodmap -pke

keycode   8 =
keycode   9 = Escape NoSymbol Escape
keycode  10 = 1 exclam 1 exclam
keycode  11 = 2 at 2 at
keycode  12 = 3 numbersign 3 numbersign
keycode  13 = 4 dollar 4 dollar
keycode  14 = 5 percent 5 percent
keycode  15 = 6 asciicircum 6 asciicircum
keycode  16 = 7 ampersand 7 ampersand
keycode  17 = 8 asterisk 8 asterisk
keycode  18 = 9 parenleft 9 parenleft
keycode  19 = 0 parenright 0 parenright
...
keycode  34 = bracketleft braceleft bracketleft braceleft
keycode  35 = bracketright braceright bracketright braceright
keycode  36 = Return NoSymbol Return
keycode  37 = Control_L NoSymbol Control_L
keycode  38 = a A a A
keycode  39 = s S s S
keycode  40 = d D d D
keycode  41 = f F f F
keycode  42 = g G g G
keycode  43 = h H h H
keycode  67 = F1 F1 F1 F1 F1 F1 XF86Switch_VT_1
keycode  68 = F2 F2 F2 F2 F2 F2 XF86Switch_VT_2
...
```

> [!warning]
> xmodmap设置会被setxkbmap重置，它不仅将字母数字键改变为映射表中的值，还将所有其他键重置为启动时的默认值。
> 由于Xorg的限制，xmodmap设置不会自动应用到热插拔设备上。如果在应用了自定义映射表后，系统中添加了一个新的键盘，则必须重新应用自定义映射表

#### setxkbmap

`setxkbmap`: set keyboard layout;

```man

SETXKBMAP(1)                General Commands Manual                SETXKBMAP(1)

NAME
       setxkbmap - set the keyboard using the X Keyboard Extension

SYNOPSIS
       setxkbmap [ args ] [ layout [ variant [ option ...  ] ] ]

DESCRIPTION
       The  setxkbmap command maps the keyboard to use the layout determined by
       the options specified on the command line.

       An XKB keymap is constructed from a number of components which are  com‐
       piled only as needed.  The source for all of the components can be found
       in /usr/share/X11/xkb.

OPTIONS
       -compat name
               Specifies  the  name  of the compatibility map component used to
               construct a keyboard layout.

       -config file
               Specifies the name of an XKB configuration file which  describes
               the keyboard to be used.

       -device device
               Specifies  the  numeric  device id of the input device to be up‐
               dated with the new keyboard layout. If not specified,  the  core
               keyboard device of the X server is updated.

       -display display
               Specifies  the  display to be updated with the new keyboard lay‐
               out.

       -geometry name
               Specifies the name of the geometry component used to construct a
               keyboard layout.

       -help   Prints a message describing the valid input to setxkbmap.

       -I directory
               Adds a directory to the list of directories to be used to search
               for specified layout or rules files.

       -keycodes name
               Specifies the name of the keycodes component used to construct a
               keyboard layout.

       -keymap name
               Specifies the name of the keymap description used to construct a
               keyboard layout.

       -layout name
               Specifies the name of the layout used to  determine  the  compo‐
               nents which make up the keyboard description. The -layout option
               may  only  be  used once. Multiple layouts can be specified as a
               comma-separated list.

       -model name
               Specifies the name of the keyboard model used to  determine  the
               components  which  make  up  the keyboard description.  Only one
               model may be specified on the command line.

       -option name
               Specifies the name of an  option  to  determine  the  components
               which make up the keyboard description;  multiple options may be
               specified,  one  per  -option flag. Note that setxkbmap adds op‐
               tions specified in the command line to the options that were set
               before (as saved in root window properties). If you want to  re‐
               place  all  previously  specified  options, use the -option flag
               with an empty argument first.

       -print  With this option setxkbmap just prints component names in a for‐
               mat acceptable by xkbcomp (an XKB keymap  compiler)  and  exits.
               The option can be used for tests instead of a verbose option and
               in  cases  when one needs to run both the setxkbmap and the xkb‐
               comp in chain (see below).

       -query  With this option setxkbmap just prints the current rules, model,
               layout, variant, and options, then exits.

       -rules file
               Specifies the name of the rules file used  to  resolve  the  re‐
               quested layout and model to a set of component names.

       -symbols name
               Specifies  the name of the symbols component used to construct a
               keyboard layout.

       -synch  Force synchronization for X requests.

       -types name
               Specifies the name of the types component used  to  construct  a
               keyboard layout.

       -variant name
               Specifies which variant of the keyboard layout should be used to
               determine the components which make up the keyboard description.
               The -variant option may only be used once. Multiple variants can
               be  specified as a comma-separated list and will be matched with
               the layouts specified with -layout.

       -verbose|-v [level]
               Specifies level of verbosity in output messages.   Valid  levels
               range  from 0 (least verbose) to 10 (most verbose).  The default
               verbosity level is 5.  If no level  is  specified,  each  -v  or
               -verbose flag raises the level by 1.

       -version
               Prints the program's version number.

USING WITH xkbcomp
       If you have an Xserver and a client shell running on different computers
       and  some  XKB  configuration files on those machines are different, you
       can get problems specifying a keyboard map by model, layout, and options
       names.  This is because setxkbmap converts its arguments to names of XKB
       configuration files according to files that are on the client-side  com‐
       puter,  then  sends  these file names to the server where xkbcomp has to
       compose a complete keyboard map using files which the server has.   Thus
       if the sets of files differ in some way, the names that setxkbmap gener‐
       ates can be unacceptable on the server side.  You can solve this problem
       by  running  the xkbcomp on the client side too.  With the -print option
       setxkbmap just prints the file names in an  appropriate  format  to  its
       stdout  and this output can be piped directly to the xkbcomp input.  For
       example, the command

       setxkbmap us -print | xkbcomp - $DISPLAY

       makes both steps run on the same (client) machine and loads  a  keyboard
       map into the server.

XWAYLAND
       Xwayland  is  an  X  server  that  uses a Wayland Compositor as backend.
       Xwayland acts as translation layer between the X protocol and  the  Way‐
       land protocol but does not manage the keymaps - these are handled by the
       Wayland Compositor.

       Changing  the  keymap  with  setxkbmap is not supported by Xwayland.  In
       most instances, using setxkbmap on Xwayland is indicative of a bug in  a
       shell script and setxkbmap will print a warning. Use the Wayland Compos‐
       itor's native XKB configuration methods instead.

SEE ALSO
       xkbcomp(1), xkeyboard-config(7)

FILES
       /usr/share/X11/xkb

X Version 11                    setxkbmap 1.3.4                    SETXKBMAP(1)
```

simple examples

```sh
setxkbmap -option ctrl:swapcaps # swap ctrl and caps
setxkbmap -option ctrl:nocaps
```

#### xcape

`xcape`: use a modifier as another key.

```man

XCAPE(1)                          xcape Manual                         XCAPE(1)

NAME
       xcape - use a modifier key as another key

SYNOPSIS
       xcape [-d] [-t timeout] [-e map-expression]

DESCRIPTION
       xcape allows a modifier key to be used as another key when it is pressed
       and released on its own. The default behaviour is to generate the Escape
       key in place of Control_L (Left Control).

OPTIONS
       -d     Debug mode.  Will run as a foreground process.

       -t timeout
              Give  a  timeout  in milliseconds.  If you hold a key longer than
              timeout a key event will not be generated.

       -e map-expression
              Use map-expression as the expression(s).

EXPRESSION SYNTAX
       Expression syntax is ´ModKey=Key[|OtherKey]´.  Multiple expressions  can
       be passed, delimited by semi-colons (;).

       A  list  of  keysyms  can  be found in the header file <X11/keysymdef.h>
       (without the XK_ prefix).

       Note that shifted keys must be specified as a shift key followed by  the
       key  to be pressed rather than the actual name of the character. For ex‐
       ample to generate "{" the expression ´ModKey=Shift_L|bracketleft´  could
       be used (assuming that you have a key with ´{´ above ´[´).

       You  can also specify ModKey in decimal (prefix #), octal (#0), or hexa‐
       decimal (#0x). It will be interpreted as a keycode unless no correspond‐
       ing key name is found.

EXAMPLES
       Make Left Shift generate Escape when pressed and released on  it's  own,
       and  Left  Control generate Ctrl-O combination when pressed and released
       on it's own:
              xcape -e 'Shift_L=Escape;Control_L=Control_L|O'

       In conjugation with xmodmap it is possible to make an ordinary  key  act
       as an extra modifier. First map the key to the modifier with xmodmap and
       then the modifier back to the key with xcape. As an example, we can make
       the  space bar work as an additional ctrl key when held with the follow‐
       ing sequence of commands:

       First, map an unused modifier's keysym to  the  spacebar's  keycode  and
       make it a control modifier. It needs to be an existing key so that emacs
       won't spazz out when you press it. Hyper_L is a good candidate.

              spare_modifier="Hyper_L"
              xmodmap -e "keycode 65 = $spare_modifier"
              xmodmap -e "remove mod4 = $spare_modifier"
              # hyper_l is mod4 by default
              xmodmap -e "add Control = $spare_modifier"

       Next,  map  space  to  an unused keycode (to keep it around for xcape to
       use).

              xmodmap -e "keycode any = space"

       Finally use xcape to cause the  space  bar  to  generate  a  space  when
       tapped.

              xcape -e "$spare_modifier=space"
```

example

```sh
xcape -e 'Alt_L=Escape' # Escape when tap, ALt when hold
```

#### example

for example, <kbd>caps</kbd> as <kbd>escape</kbd> and <kbd>ctrl</kbd>

```sh
# I usually use following
setxkbmap -option 'caps:ctrl_modifier' # caps become ctrl
xcape -e 'Caps_Lock=Escape'
```

### wayland

似乎暂时只有KDE Gnome的环境里还有改的方法

### console

[archwiki](https://wiki.archlinux.org/title/Linux_console/Keyboard_configuration)

### keyd

[keyd](https://github.com/rvaiya/keyd)， inspired by via

这个厉害，tty, X11, wayland都能用，而且能实现很强的配置效果。

```tldr

  Remap keys.
  More information: <https://manned.org/keyd>.

  Start and enable the `keyd` service:

      systemctl enable keyd --now

  Display keypress information:

      sudo keyd monitor

  Reset bindings and reload the configuration files in `/etc/keyd`:

      sudo keyd reload

  List all valid key names:

      keyd list-keys

  Create a temporary binding:

      sudo keyd bind "pressed_key = output_key"
```

Other software:

- interception-tools with plugins
  - dual-function-keys (like Mod-Tap feature of zmk and qmk)
  - caps2esc
  - space2meta

## ref

[ref](https://unix.stackexchange.com/questions/55076/what-is-the-mode-switch-modifier-for/55154#55154)
