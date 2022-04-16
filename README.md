# The Steam Deck Experience

I am documenting my experience with the Steam Deck thus far here publicly so if
people stumble upon stuff, they can verify it here. Feel free to create a PR to
add your own experiences, I'm mostly treating this like a Wiki.

- [The Steam Deck Experience](#the-steam-deck-experience)
  - [Hardware](#hardware)
    - [Fan noise](#fan-noise)
  - [Recovery image](#recovery-image)
    - [USB disk performance](#usb-disk-performance)
  - [System updates](#system-updates)
  - [Microphone](#microphone)
  - [Touch screen stops working](#touch-screen-stops-working)
  - [Installing software](#installing-software)
    - [OBS](#obs)

## Hardware

I got the 512 GB model as shipped in the EU region. The Steam Deck has the
following hardware:

- A CPU reporting itself as `AMD Custom APU 0405`
  - 4 Zen 2 cores, 8 threads
  - Frequency between 1.6 to 2.8 GHz
  - AMD Van Gogh SoC (codenamed "Aerith"?)
- 16 GB RAMter
- An NVMe disk reporting itself as `Kingston Technology Company, Inc. OM3PDP3 NVMe SSD (rev 01)`
- RTL8822CE 802.11ac wireless network adapter (handles Bluetooth and WLAN)

It overall runs at peak wattage of 40 W.

Depending on where you look on the Internet the hardware seems to be codenamed
either "Jupiter" or "Neptune". Seemingly "Neptune" is the later established
codename when Valve also referred to the Steam Deck as "SteamPal".

### Fan noise

The community is aware of an issue with certain Decks being equipped with a
different fan that has a loud "whining" noise when it spins up as opposed to the
fan used in early units that did not have this. A Discord user's guess was that
the newer "delta" fan is built more symmetrically, causing feedback noise to be
very resonant. Affected Decks often seem to have a serial number starting with
`20`.

My Deck does not have this issue (serial number starts with `1510`). I have made
a recording of the fan noise on my unit.

[Deck fan noise levels (4 levels, each 5 seconds)](audio/noise-levels-deck.ogg)

## Recovery image

### USB disk performance

After initially flashing the recovery image on a USB disk and booting it up, it
may take several minutes or even beyond 10 to boot. It entirely depends on the
USB disk's performance since the recovery image will try to expand a
pre-existing /home partition and initialize a lot of files in the /var partition
right on the disk. Even after it is done with that it may still perform poorly
due to a lot of system logs being written. If you tinker around with the
recovery image make sure that you have the system partition set read-only
(`steamos-readonly enable`), otherwise expect continuous slow downs.

## System updates

SteamOS handles updates to the operating system via [RAUC](https://rauc.io) and
an A/B partition structure. Updates are essentially just images being written to
the root filesystem partition, of which there are two, an "A" and a "B"
partition (EFI, ESP and var partitions both also seem to have an A and B
partition each).

Updates will always be installed to the partset that you are currently *not*
running SteamOS from. It is then marked as the partset to boot from before
reboot. If the newly written image fails to boot, then the bootloader will fall
back to the old partset in order to restore functionality. Otherwise the
operating system on the new partset will mark itself as successfully booted and
the bootloader will continue to use it in the future.

## Microphone

The Steam Deck has a dual microphone array similar to the Valve Index, however
it picks up a bit of the background noise coming from the fan. For this reason,
Valve configures a virtual recording device for software to use by default which
is called the "Filter Chain Source". It adds a few effects to filter out the
noise completely and slightly reduce the volume of input clicks and haptics
audible in the microphone. There is a third device that tries to cancel audible
echo, however I've not used it, and I suspect that the Filter Chain Source makes
use of it internally anyways.

I'm linking a recording I did with Audacity and JACK that I posted on Discord
here:

[Deck Microphone Test (raw and "Filter
Chain")](audio/deck-microphone-test-raw-and-filterchain.ogg)

## Touch screen stops working

At one point, I accidentally drained the battery on the Steam Deck completely.
On the next reboot the touch screen stopped working completely. According to
Google search results, seemingly a reboot can sometimes fix this, but in my case
it just absolutely refused to work properly.

This is actually caused by a firmware bug. As of right now, the touch screen is
not initialized correctly until the Deck is put into "Battery Storage" mode,
hooked up to power and then rebooted. I guess this mode forces a reset of some
configuration that allows the touch screen to be initialized properly. In a
support ticket, Valve's support team did say they were going to fix it in a
future firmware update.

[A reddit user has documented the workaround with the support ticket
quote.](https://www.reddit.com/r/SteamDeck/comments/t7skls/comment/hzr2elr/?utm_source=reddit&utm_medium=web2x&context=3)

> We've identified a firmware initialization issue that causes the touchscreen
> to stop responding. The fix is being tested now, and will roll out in an
> upcoming BIOS update.  
> If your touchscreen stops responding, a simple reboot is not enough to clear
> the issue. However, you can reset the touchscreen by putting the Steam Deck
> into battery storage mode:
>
> - Power the Steam Deck down normally (choose Shut Down from the Power menu)
> - Hold down the Volume+ button and press the power button. The Steam Deck
>   should boot into the BIOS menu.
> - Using the D-Pad and A to select, navigate to "Setup Utility"
> - Navigate to the Power Menu
> - Choose Battery storage mode, and confirm.
>
> You should see the Steam Deck power off, and the power LED will blink three
> times to confirm battery storage mode.  
> Plug the Steam Deck in to the supplied power supply. Turn on the Steam Deck.

## Installing software

Generally, it is considered a bad idea to use `pacman` or `yay` to install
software on the Deck despite the fact it runs on top of Arch Linux. The reason
is the way that SteamOS updates itself: Updates are not incremental but affect
the whole partition, and all system modifications except for a few overlay paths
and the /home partition will get completely overwritten.

Instead, you should use Flatpak or other means to install software to /var or
/home instead.

### OBS

A special case with OBS is the installation of plugins: If you can't find
plugins ready to install in Flatpak, you will need to add them to
`~/.var/app/com.obsproject.Studio/config/obs-studio/plugins/`. Make sure your
plugins have the following folder structure (assuming the plugin you want to
install is obs-ndi):

```
+ ~/.var/app/com.obsproject.Studio/config/obs-studio/plugins/
  + obs-ndi/
    + bin/
      + 64bit/
        - <insert .so files here>
    + data/
      - <insert plugin data/asset files here>
```

If your plugin depends on other libraries (in the case of obs-ndi that would be
the avahi client and NDI itself), you will have to convince Flatpak or the
software to somehow load those libraries as well.

In the case of obs-ndi, you can set the environment variable
`NDI_RUNTIME_V4_DIR` (or `NDI_RUNTIME_V5_DIR` if you tweaked obs-ndi to use NDI
v5 and compiled it yourself) to point to the plugin's directory so you can copy
`libndi.*` files there.

For system libraries such as avahi, there is currently no good way to just
insert libraries into the Flatpak space OBS runs in. You will have to repackage
your plugin as a Flatpak addon if not available as such already.
