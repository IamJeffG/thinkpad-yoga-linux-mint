This documents how I installed Linux Mint 17.3 on a Lenovo ThinkPad Yoga 14", purchased from Best Buy.

Table of contents
=================

1.  Dual Boot (partition drive and install Linux)
2.  Download files (without working wifi)
3.  Fix wifi by installing new kernel and drivers
4.  Fix graphics with new kernel and drivers
5.  Things that still don't work...

Dual Boot with Windows
======================

[Follow the steps here](https://github.com/longsleep/yoga3pro-linux/blob/master/Yoga%203%20Linux%20HOWTO.md). He uses different hardware but they are similar enough:

1.  Create recovery USB flash drive
2.  Shrink Windows partition
3.  Disable FastStartup
    *   On Windows 10, this is under "Choose what the power buttons do."
4.  BIOS Settings
    *   I checked but nothing needed to be changed
5.  Boot from USB flash drive
    *   Instead of Ubuntu, install Linux Mint 17.3
6.  **Skip** his section on "wifi". You won't be able to fix it here.
7.  Install Linux Mint 17.3
8.  **Stop**. The remaining stuff on the above link don't apply to the Lenovo ThinkPad Yoga. Keep reading below.

Find a way to get large (50MB) files from the Internet to Linux
===============================================================

Wifi doesn't work out of the box, but we're going to need to download a newer kernel

Easiest is an **external USB Wifi Adapter**. However supposedly few work on Linux Mint. I use Realtek 802.11n Adapter.

Otherwise wifi works in Windows so you can boot to Windows, download the files, then transfer to the Linux side. Options include:

- **a big USB flash drive**
- **[read directly from Windows mount](CUSTOMIZATIONS.md#mount-windows-drive-read-only)**

Fix Wifi for Intel Wireless 8260
================================

Confirm you have the `Intel Wireless 8260` card (`$ inxi -Fxz`) and that it's not currently working
(`$ sudo lshw -c network` will show: `*-network UNCLAIMED` for the relevant
card).

Support for this card comes from a set of [Intel drivers called `iwlwifi`](https://wireless.wiki.kernel.org/en/users/drivers/iwlwifi). This card was added in version 8000C-13, and this version requires Linux Kernel >= 4.1.

However, it's not so simple. Just because `iwlwifi` can support it in Linux >= 4.1, the kernel must _also_ be told to look for this driver when it sees your specific wifi card.

[This mapping wasn't added until December 2016](https://git.kernel.org/cgit/linux/kernel/git/iwlwifi/iwlwifi-fixes.git/commit/?id=4ab75944c4b324c1f5f01dbd4c4d122d2b9da187) – meaning it only got added to kernels that were being supported at that time: **4.1.17** and higher microversions, as well as **4.4**, are maintained. But 4.2 and 4.3 were EOL.

### Here's how to proceed:

1.  Wifi can work on 4.1.17, but later on we'll see graphics
    problems on 4.1, so go ahead and install __Kernel 4.4__ now. [Follow Laurent85's instructions for 4.4 here](https://forums.linuxmint.com/viewtopic.php?t=216235#p1132177), including the newer `linux-firmware`.

2.  Also install the right version of [those intel drivers](https://wireless.wiki.kernel.org/en/users/drivers/iwlwifi):

        d /tmp/
        wget https://wireless.wiki.kernel.org/_media/en/users/drivers/iwlwifi-8000-ucode-16.242414.0.tgz
        tar xzf iwlwifi-8000*.tgz
        sudo mv iwlwifi-8000*/iwlwifi-8000C-16.ucode /lib/firmware/

3.  Reboot to the 4.4 kernel by selecting it in Grub.

# Fix graphics (standby and hardware graphics acceleration)

**Kernel 4.1** breaks hardware graphics acceleration by default and also breaks standby.

*   Acceleration can be fixed via grub boot options: include a flag `i915.preliminary_hw_support=1` that enables support for newer hardware, which had been included by default in 3.19 but removed in newer kernel versions.
*   Standby is hopeless in 4.1.

With **Kernel 4.4** both acceleration and standby both work great after installing the nvidia drivers (no grub flags needed):

    sudo apt-get install nvidia-352

If in the future, some package manager update happens to bork graphics (e.g.
[the November 2016 forced upgrade to
`nvidia-367=367.57`](https://forums.linuxmint.com/viewtopic.php?f=49&t=233050
)): You can always add `nomodeset` to the `linux` command through GRUB to get
some half-working graphical desktop.

Things that still don't work...
===============================

Auto-rotate screen when I flip the computer
-------------------------------------------

Not working. There are two sides to this...

### Orientation detection

There are many almost-working scripts (e.g. [^1], [^2], [^3]) out there and they all rely on
`/sys/bus/iio`. Strangely this location doesn't exist on my computer.

* [I've also asked about this on the Linux Mint Forums.](https://forums.linuxmint.com/viewtopic.php?f=49&t=221038)
* They
also all mention systemd as a requirement, which [Linux Mint 17.3 doesn't
support](http://www.pcworld.com/article/2921385/its-optional-for-now-but-linux-mint-expects-to-switch-to-systemd-next-year.html).

[^1]: https://github.com/johanneswilm/thinkpad-yoga-14-s3-scripts/blob/master/rotate/thinkpad-rotate.py
[^2]: https://github.com/pfps/yoga-laptop/blob/79d7db4bef4ebcae12d8ec1d69dc48696796375d/docs/Orientation%20and%20rotation
[^3]: https://github.com/ragtag/spin

### Rotation (manual)

Really the oriendation detection is moot because I cannot even rotate the screen manually.

    $ xrandr --output eDP1 --rotate left
    xrandr: output eDP1 cannot use rotation "left" reflection "none"

Alternately:

    xrandr -o right

reduces screen size significantly and doesn't rotate anyway. (To get out of this, Win+Space, type "Display" and re-Apply the Desktop.)

Perhaps the versions of xrandr and of the NVIDIA driver (nvidia-352)
don't support this together? https://bugs.launchpad.net/nvidia-drivers-ubuntu/+bug/518132


Occasionally standby stops working
----------------------------------

Usually standby works (either through menu or by closing the lid).
But every 10th time or so it stops and standby won't work again til laptop
is rebooted.

Specifically:

* closing the lid does nothing.  When closing the lid the following is emitted to dmesg:

    i915 0000:00:02.0: BAR 6: [??? 0x00000000 flags 0x2] has bogus alignment

* clicking the "Quit" menu item ends up lagging about 1 minute til the Shutdown
  popup appears, and this popup is missing the (usually present) "Standby" option.
