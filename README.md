# Habbo Launcher for Linux (unofficial)

A one-command installer that gets the official **Habbo Launcher** running on Linux through Wine. It adds an app-menu entry and makes the **"Launch and log in"** button on habbo.com open the app.

> **Unofficial.** This project is not affiliated with, endorsed by, or supported by Sulake or Habbo. "Habbo" is a trademark of Sulake. This repository contains no Habbo software. The script downloads the official Windows launcher from Habbo's own servers, or uses a copy you downloaded yourself.

## Requirements

- An x86_64 Linux PC
- **Wine 8 or newer**, installed from your distro or WineHQ. The Flatpak version of Wine isn't supported.
- `curl` or `wget`
- Optional: `icoutils` for the real Habbo icon, and `winetricks` for the `--dxvk` option

Getting a recent Wine:

| Distro | Command |
|---|---|
| Arch / Manjaro / EndeavourOS | `sudo pacman -S wine` |
| Fedora / Nobara | `sudo dnf install wine` |
| openSUSE | `sudo zypper install wine` |
| Ubuntu / Debian / Mint / Pop!_OS | Ubuntu's own package is often too old (Wine 7 is known to crash). Install `winehq-stable` from WineHQ's repository: <https://gitlab.winehq.org/wine/wine/-/wikis/Download> |

## Install

Open a terminal and run:

```bash
curl -fsSLO https://raw.githubusercontent.com/Sentinaut/Habbo-Linux/main/install-habbo.sh
chmod +x install-habbo.sh
./install-habbo.sh
```

Run it as your normal user, not with `sudo`. When it finishes, open **Habbo Launcher** from your app menu, or click **Launch and log in** on habbo.com.

To update or repair an existing install later, run the same three commands again.

If the automatic download fails, log in at habbo.com, choose **Download for Windows**, and point the script at the file:

```bash
./install-habbo.sh --installer ~/Downloads/HabboLauncher-Setup-*.exe
```

### Options

| Option | What it does |
|---|---|
| `--installer FILE` | Use an installer `.exe` you downloaded instead of fetching one |
| `--no-gpu` | Turn off GPU acceleration. Slower, but use it if the launcher crashes on start |
| `--dxvk` | Also install DXVK, which often makes the hotel smoother. Needs `winetricks` and a Vulkan-capable GPU |
| `--force-old-wine` | Try anyway with a Wine older than 8 |
| `--uninstall` | Remove everything the script created |

Re-running the script is safe. It repairs the setup and refreshes the menu entry without reinstalling.

## What it sets up

| Thing | Location |
|---|---|
| Wine prefix (Windows 10 mode) | `~/.local/share/habbo-launcher/prefix` |
| Launch script | `~/.local/bin/habbo-launcher` |
| App-menu entry and `habbo://` link handler | `~/.local/share/applications/habbo-launcher.desktop` |
| Launcher log | `~/.cache/habbo-launcher.log` |

The launch script carries the fixes needed for the launcher to run under Wine:

- **Windows 10 mode.** The Electron launcher refuses to run on the Windows 7 that Wine reports by default.
- **`--no-sandbox`.** Chromium's sandbox doesn't work under Wine.
- **Output goes to a log file.** Started from a menu or browser there's no terminal, and the launcher crashes with *"EINVAL: invalid argument, uv_pipe_open"* when it tries to write to one.
- **Wine's own menu entries are disabled.** Otherwise Wine registers a second `habbo://` handler that skips these fixes and crashes the same way.
- **Login links are handed over after start-up.** A launcher that is just starting ignores the login link from the website, but a running one accepts it. The script offers the link and reads the launcher's log: if a running launcher took it, it's done. If this click started the launcher, the script waits until it logs "Application started", then offers the link again.

## Troubleshooting

**"A JavaScript error occurred in the main process … uv_pipe_open"**
The launcher was started without the launch script. This usually happens through a link handler Wine created earlier. Check which handler your desktop uses:
```bash
xdg-mime query default x-scheme-handler/habbo
```
It should say `habbo-launcher.desktop`. If it doesn't, re-run the installer. If you have older Wine-created files mentioning Habbo in `~/.local/share/applications/`, delete them.

**It crashes or shows a blank window on start.** Re-run with `--no-gpu`.

**It's laggy.** Make sure your GPU driver is working: `glxinfo -B | grep renderer` should name your graphics card, not `llvmpipe`. Then try `--dxvk`.

**The first "Launch and log in" opens the launcher but doesn't log you in.** On slower PCs the login screen may need longer to appear. Open `~/.local/bin/habbo-launcher` in a text editor and raise the number in the line `EXTRA_WAIT="${HABBO_LINK_DELAY:-0}"`, for example to 3.

**"Launch and log in" does nothing.** Your browser may be asking, or may have silently refused, to open the link. Allow it to open with Habbo Launcher. As a fallback, open the launcher from the menu and paste the login code from the **Copy** button on habbo.com.

**Closing the window doesn't fully quit the launcher.** Under Wine some launcher processes can linger. To clear them all out:
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
- the output of `wine --version`
- what you did and what happened, with screenshots of any error windows
- the launcher log: `cat ~/.cache/habbo-launcher.log`

## Uninstall

```bash
./install-habbo.sh --uninstall
```
