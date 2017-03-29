Here are some customizations to Linux Mint that I like to do.

Mount Windows Drive, read-only
------------------------------

We mount the Windows drive in read-only mode, just to be safe.

This mounts one time, but you'd have to re-run it on every boot. Therefore,
don't do this, but instead skip to the subsequent code block that uses
`/etc/fstab`.

    sudo mkdir /windows
    sudo fdisk -l
    sudo mount -t ntfs -o nls=utf8,umask=0222 /dev/sda3 /windows
    sudo mount /windows -o remount,ro,noatime

Here's how to automatically mount on every boot.

1. `sudo mkdir /windows`
2. `sudo nano /etc/fstab`
3. Append the following line
    ```
    UUID=01D1720#########  /windows ntfs defaults,ro,uid=1000,umask=022,gid=100 0 0
    ```

    where

    * UUID was identified by `sudo blkid`: the one with `LABEL="WINDOWS"`.
    * `ro` means "read-only"
    * `uid=1000,umask=022,gid=100` from stackoverflow and `$ id`

4. Reboot to prove it worked. If not, you'll see an error during startup.

Enable Compose Keys
--------------------------

Linux Mint makes it easy to turn on Compose keys in keyboard options, but this
computer's physical keyboard has very few "superfluous" keys that I don't regularly
use.  My hacky solution is to define `Shift+Win` as the Compose key, without changing
the `Win` key's primary function of opening the Cinnamon menu.

I'm on a U.S. English keyboard.  I do not enable a 3rd level.

[The solution is described here.](https://forums.linuxmint.com/viewtopic.php?f=208&t=221606&p=1169670#p1169670)

This allows things like

    Shift+LeftWin  e  =   →  €
    Shift+LeftWin  /  c   →  ¢
    Shift+LeftWin  o  c   →  ©
    Shift+LeftWin  1  4   →  ¼
    Shift+LeftWin  `  e   →  è
    Shift+LeftWin  -  >   →  →


Keyboard shortcuts for missing audio keys
-----------------------------------------

The Thinkpad Yoga has no built-in keys to adjust audio playback.
Emulate this using `Win+Left` or `Win+Right` for previous/next track,
and `Win+Down` to toggle play.
(I had initially tried `Alt+Left` or `Alt+Right`, but this interferes
with web browser shortcuts for prev/next page.)

This is found in: Keyboard → Shortcuts → Sound and Media

When setting "Previous track", "Next track", and "Play", Cinnamon will warn
you that it's changing the shortcuts from "Push tile left/right/down".
I never used this anyway.


Reset screenshot directory
--------------------------

([source](https://forums.linuxmint.com/viewtopic.php?t=182801#p947856))

    mkdir ~/Downloads/screenshots
    gsettings set org.gnome.gnome-screenshot auto-save-directory ~/Downloads/screenshots


Install latest QGIS
-------------------

Follow _all_ the steps [on this page](http://www.qgis.org/en/site/forusers/alldownloads.html#debian-ubuntu).

* repository: `http://qgis.org/ubuntugis`
* on Mint 17.3, the codename is `trusty`


Display "save your tabs?" warning when quitting Firefox
-------------------------------------------------------

([source](http://ccm.net/faq/13072-display-the-save-and-quit-message-when-closing-firefox))

* in `about:config`, set `browser.showquitwarning` to True.

MTP for Samsung A3 phone
------------------------

This is not a problem with the Thinkpad, but a workaround here makes up
for Samsung's shortcoming.  My Galaxy A3 phone has poor support for USB Mass
Storage, so I have to use MTP instead.  There's a Banshee plugin that
tries to hijack your new MTP connections, so it's best left disabled:

* in Banshee's Preferences → Extensions, disable "Mass Storage Media Player"
  and "MTP Media Player".

Also, if your phone has too many files, MTP will crash nemo. Use [this
transfer client](https://github.com/whoozle/android-file-transfer-linux)
instead (and disable Nemo's automount).


Touchscreen Scroll in Firefox
-----------------------------

This enables **two-finger scrolling** in Firefox. Use one finger and it will still
select text, as always.

* In Firefox's `about:config`, [set `dom.w3c_touch_events.enabled=1`](https://bbs.archlinux.org/viewtopic.php?id=216569) (default was 2)
* Start firefox with `env MOZ_USE_XINPUT2=1`, [as instructed here.](http://askubuntu.com/a/886914/547689)
