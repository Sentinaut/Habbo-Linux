# Habbo Launcher for Linux (unofficial)

A one-command installer that gets the official **Habbo Launcher** running on Linux through Wine. It adds an app-menu entry and makes the **"Launch and log in"** button on habbo.com open the app and sign you in.

> **Unofficial.** This project is not affiliated with, endorsed by, or supported by Sulake or Habbo. "Habbo" is a trademark of Sulake. This repository contains no Habbo software. The script downloads the official Windows launcher from Habbo's own servers, or uses a copy you downloaded yourself.

## Requirements

- An x86_64 Linux PC
- **Wine 8 or newer**, from your distro or WineHQ. The Flatpak version of Wine isn't supported.
- `curl` or `wget`
- Optional: `icoutils` for the real Habbo icon in your app menu, and `winetricks` for the `--dxvk` option

Getting Wine:

| Distro | Command |
|---|---|
| Ubuntu / Kubuntu / Debian / Mint / Pop!_OS | `sudo apt install wine`. Recent releases (e.g. Ubuntu 24.04+, Debian 12+) are new enough. On older ones such as Ubuntu 22.04, install `winehq-stable` from [WineHQ](https://gitlab.winehq.org/wine/wine/-/wikis/Download) instead. |
| Arch / Manjaro / EndeavourOS | `sudo pacman -S wine` |
| Fedora / Nobara | `sudo dnf install wine` |
| openSUSE | `sudo zypper install wine` |

Check your version with `wine --version`.

## Install

Open a terminal and run:

```bash
curl -fsSLO https://raw.githubusercontent.com/Sentinaut/Habbo-Linux/main/install-habbo.sh
chmod +x install-habbo.sh
./install-habbo.sh
```

Run it as your normal user, not with `sudo`. When it finishes, open **Habbo Launcher** from your app menu, or click **Launch and log in** on habbo.com.

To update to a newer version of this installer, or to repair an install, run the same three commands again. `./install-habbo.sh --version` shows which version you have.

If the automatic download fails, log in at habbo.com, choose **Download for Windows**, and point the script at the file:

```bash
./install-habbo.sh --installer ~/Downloads/HabboLauncher-Setup-*.exe
```

### Options

| Option | What it does |
|---|---|
| `--installer FILE` | Use an installer `.exe` you downloaded instead of fetching one |
| `--no-gpu` | Turn off GPU acceleration. Slower, but use it if the launcher crashes on start |
| `--dxvk` | Also install DXVK, which can make the hotel smoother. Needs `winetricks` and a Vulkan-capable GPU |
| `--force-old-wine` | Try anyway with a Wine older than 8 |
| `--uninstall` | Remove everything the script created |
| `--version` | Show the installer's version |

Re-running the script is safe. It repairs the setup and refreshes the menu entry without reinstalling the launcher.

## What it sets up

| Thing | Location |
|---|---|
| Wine prefix (Windows 10 mode) | `~/.local/share/habbo-launcher/prefix` |
| Launch script | `~/.local/bin/habbo-launcher` |
| App-menu entry and `habbo://` link handler | `~/.local/share/applications/habbo-launcher.desktop` |
| Launcher log | `~/.cache/habbo-launcher.log` |

The launch script carries the fixes the launcher needs to run under Wine:

- **Windows 10 mode.** The launcher won't run on the Windows 7 that Wine reports by default.
- **`--no-sandbox`.** Chromium's sandbox doesn't work under Wine.
- **Output goes to a log file.** Started from a menu or browser there's no terminal, and the launcher crashes with *"EINVAL: invalid argument, uv_pipe_open"* when it tries to write to one.
- **Wine's own menu entries are turned off.** Otherwise Wine registers a second `habbo://` handler that skips these fixes and crashes the same way.
- **Login links work on the first click.** A launcher that's still starting up ignores the login link from the website. If your click is what started it, the script waits until the launcher logs "Application started", then passes the link again.
- **Leftovers get cleaned up.** Under Wine, closing the launcher window can leave some of its processes running. The next time the launcher starts, the script closes any leftovers from that install. Other Wine installs are left alone.

## Troubleshooting

**"A JavaScript error occurred in the main process … uv_pipe_open"**
The launcher was started without the launch script, usually through a link handler Wine created earlier. Check which handler your desktop uses:
```bash
xdg-mime query default x-scheme-handler/habbo
```
It should say `habbo-launcher.desktop`. If it doesn't, re-run the installer. If you have older Wine-created files mentioning Habbo in `~/.local/share/applications/`, delete them.

**It crashes or shows a blank window on start.** Re-run the installer with `--no-gpu`.

**It's laggy.** Make sure your graphics driver is working: `glxinfo -B | grep renderer` should name your graphics card, not `llvmpipe`. Then try re-running the installer with `--dxvk`.

**The first "Launch and log in" opens the launcher but doesn't sign you in.** Your PC may need a moment longer before the login screen accepts the code. Open `~/.local/bin/habbo-launcher` in a text editor, find the line `EXTRA_WAIT="${HABBO_LINK_DELAY:-0}"` and change the `0` to `3`. Re-running the installer resets this, so make the change again afterwards.

**"Launch and log in" does nothing.** Your browser may be asking, or may have quietly refused, to open the link. Allow it to open with Habbo Launcher. As a fallback, open the launcher from the menu and paste the login code from the **Copy** button on habbo.com.

**The launcher won't open, or opens nothing.** Close everything Wine is running for it, then try again:
```bash
WINEPREFIX=~/.local/share/habbo-launcher/prefix wineserver -k
```

**Anything else.** Check `~/.cache/habbo-launcher.log`, or ask for help (see below).

## Getting help

If something isn't working, there are two ways to get in touch:

- **GitHub:** [open an issue](https://github.com/Sentinaut/Habbo-Linux/issues). You'll need a free GitHub account.
- **Discord:** message [**Sentinaut**](https://discord.com/users/1536053675808792680).

It helps a lot if you include:

- your distro and version, e.g. Kubuntu 26.04
- the output of `wine --version` and `./install-habbo.sh --version`
- what you did and what happened, with screenshots of any error windows
- the launcher log: `cat ~/.cache/habbo-launcher.log`

## Uninstall

```bash
./install-habbo.sh --uninstall
```

This removes the Wine prefix (including the launcher and anything it downloaded), the launch script, the menu entry and the log.

## Tested on

- Kubuntu with KDE Plasma 6 (Wayland), Wine 10.0 from Ubuntu's own repositories, NVIDIA RTX 3070

If it works (or doesn't) on your setup, let us know in an issue so this list can grow.

## For maintainers

The installer downloads a fixed version of Habbo's Windows installer (set in `INSTALLER_URL` near the top of the script). The launcher updates itself on first run, so an older version is fine. If Habbo ever removes that file, point `INSTALLER_URL` at a newer one and bump `SCRIPT_VERSION`.

## License

The installer script is released under the [MIT License](LICENSE). This covers only the script, not Habbo or any of Sulake's software.
