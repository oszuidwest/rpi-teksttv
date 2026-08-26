# rpi-texttv

`rpi-texttv` turns a Raspberry Pi into a dedicated narrowcasting player. It starts Chromium full-screen on one or two HDMI outputs and can also run an audio stream through `mpv`.

The installer uses a small X11/Openbox setup, so Raspberry Pi OS Lite is enough; a desktop image is not required.

## Requirements

- Raspberry Pi 4, 5, 400 or 500
- Raspberry Pi OS Trixie (64-bit) Lite
- An internet connection during installation
- A regular user account with `sudo` access

The output is fixed at 1920×1080, 50 Hz interlaced. Make sure the connected display or video chain accepts 1080i50.

## Installation

Log in as the regular user that will run the display, then run:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/oszuidwest/rpi-texttv/main/install.sh)"
```

Do not run the command through `sudo` or from a root shell. The installer asks for `sudo` itself when it needs to change system files.

The script installs the display software, configures console autologin and starts X11 automatically on `tty1`. It also changes the boot video settings, sets the timezone to `Europe/Amsterdam`, limits journald storage, removes CUPS and reboots the Pi when it is done.

## Installer settings

The installer asks these questions. Press Enter to accept the default.

| Setting | Default | Purpose |
| --- | --- | --- |
| `DO_UPDATES` | `y` | Run a full OS upgrade before installing the display software |
| `INSTALL_VNC` | `y` | Install and enable RealVNC for remote access |
| `INSTALL_MPV` | `y` | Play a background audio stream with `mpv` |
| `MPV_URL` | `https://icecast.zuidwest.cloud/zuidwest.stl` | Audio stream used by `mpv` |
| `MPV_VOLUME` | `60` | Audio volume from 0 to 100 |
| `CHROME_URL` | `https://teksttv.zuidwest.cloud/zuidwest-1/` | Page shown on the first display |
| `USE_DUAL_SCREEN` | `n` | Enable the second HDMI output |
| `CHROME_URL_2` | Same as `CHROME_URL` | Page shown on the second display |
| `LIMITED_RGB` | `n` | Send limited-range RGB for an HDMI-to-SDI chain |

With two displays enabled, each screen gets its own Chromium instance. If audio is enabled, `mpv` runs as a systemd user service for each HDMI output and restarts automatically after a failure.

## HDMI and SDI output

The installer forces 1080i50 in three places: the kernel command line, the supplied EDID file and an `xrandr` mode in the Openbox startup script. The relevant defaults in `install.sh` are:

```bash
VIDEO_OPTIONS="video=HDMI-A-1:1920x1080@50D"
BOOT_OPTIONS="drm.edid_firmware=edid/edid.bin vc4.force_hotplug=0x01 consoleblank=1 logo.nologo"
```

Enabling a second display adds the same mode for `HDMI-A-2` and changes `vc4.force_hotplug` to `0x03`.

Leave `LIMITED_RGB` disabled when the Pi is connected directly to a normal TV or monitor. Enable it when an HDMI-to-SDI converter expects video-range RGB (16–235). This prevents the converter from clipping highlights and shadows in a broadcast or vMix setup.

## What runs after boot

The display stack is X11, Openbox and Chromium. `feh` supplies a fallback background and `unclutter` hides the pointer. Chromium runs in kiosk and incognito mode with a separate profile directory for each screen.

The chosen display settings are stored in `~/.config/openbox/display.conf`. Openbox starts everything from `~/.config/openbox/autostart` when the console user logs in.

On Raspberry Pi 5 hardware, the installer also adds active-cooling settings that switch the fan on at 55 °C, off again at 35 °C and run it at full speed while active.

## Contributing

Bug reports and pull requests are welcome. Before submitting a shell change, run:

```bash
bash -n install.sh
shellcheck install.sh
```

## License

Released under the [MIT License](LICENSE).
