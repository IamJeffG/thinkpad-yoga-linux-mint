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
