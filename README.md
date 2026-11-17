# Ubuntu Setup: My Laptop

Checklist for a fresh **Ubuntu 26.04 LTS** install on this laptop. Do the steps in order. Allow about an hour, plus download time.

| Part               | Model                          | Setup needed                                                                        |
| ------------------ | ------------------------------ | ----------------------------------------------------------------------------------- |
| CPU + graphics     | AMD Ryzen 7 260, Radeon 780M   | Video acceleration check ([step 3](#3-install-codecs-and-check-video-acceleration)) |
| Dedicated graphics | NVIDIA GeForce RTX 5060 Laptop | NVIDIA driver ([step 4](#4-install-the-nvidia-driver))                              |

## Contents

- [Before you start](#before-you-start)

1. [Update the system](#1-update-the-system)
2. [Update firmware](#2-update-firmware)
3. [Install codecs and check video acceleration](#3-install-codecs-and-check-video-acceleration)
4. [Install the NVIDIA driver](#4-install-the-nvidia-driver)
5. [Make it look like GNOME](#5-make-it-look-like-gnome)
6. [Install apps](#6-install-apps)
7. [Set up Git and VS Code](#7-set-up-git-and-vs-code)
8. [Set up backups](#8-set-up-backups)
9. [Maintenance and upgrades](#9-maintenance-and-upgrades)

---

## Before you start

### Using the terminal

- Open it with <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>T</kbd>.
- Paste with <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>V</kbd> (plain <kbd>Ctrl</kbd> + <kbd>V</kbd> doesn't work there).
- When `sudo` asks for your password, nothing appears as you type. That's normal; type it and press <kbd>Enter</kbd>.
- When asked `Do you want to continue? [Y/n]`, press <kbd>Enter</kbd>.

### Rules that keep future upgrades safe

> [!IMPORTANT]
> Every choice in this guide follows these rules. Keep following them when you add things later:
>
> - Install software only from Ubuntu, from the maker's official repository, or from Flathub.
> - Never install the NVIDIA driver from nvidia.com. Ubuntu's driver is rebuilt for every kernel update; NVIDIA's installer isn't.
> - Don't uninstall `ubuntu-desktop` or Snap. Ubuntu's upgrades expect both.
> - Avoid PPAs (unless run by the app's own makers, like OBS's) and `curl ... | bash` install scripts. They're the most common reason an upgrade fails.

---

## 1. Update the system

Installs every update released since the install image was made.

```bash
sudo apt update && sudo apt full-upgrade
```

```bash
sudo reboot
```

## 2. Update firmware

Updates the laptop's BIOS and device firmware.

> [!WARNING]
> Plug in the charger first. A firmware update that loses power halfway can leave the laptop unable to start.

```bash
fwupdmgr refresh --force
```

```bash
fwupdmgr update
```

"No updates available" is normal. If it asks to reboot, do.

## 3. Install codecs and check video acceleration

Adds the codecs needed for common video and audio formats, plus Microsoft's fonts:

```bash
sudo apt install ubuntu-restricted-extras
```

> [!TIP]
> A font licence screen appears partway through. Press <kbd>Tab</kbd> to select `<Ok>` and press <kbd>Enter</kbd>. On the next screen, use the arrow keys to select `<Yes>` and press <kbd>Enter</kbd>.

Ubuntu's AMD driver already decodes video on the Radeon 780M's hardware, which saves battery. Check that it works:

```bash
sudo apt install vainfo
```

```bash
vainfo
```

Look for `VAProfileH264High`, `VAProfileHEVCMain` and `VAProfileAV1Profile0`, each with `VAEntrypointVLD`. That means H.264, H.265 and AV1 video are decoded by the GPU.

## 4. Install the NVIDIA driver

If you ticked **Install third-party software for graphics and Wi-Fi hardware** during installation, the driver may already be there. Check first:

```bash
nvidia-smi
```

If it lists the RTX 5060, skip to [Using the RTX 5060](#using-the-rtx-5060).

Otherwise, see which driver Ubuntu recommends. RTX 50 cards need one whose name ends in `-open`:

```bash
ubuntu-drivers devices
```

Install the recommended one:

```bash
sudo ubuntu-drivers install
```

> [!NOTE]
> **If it asks you to set a Secure Boot password:** invent a short one with letters and numbers only. After the reboot, a blue **MOK Management** screen appears. Choose **Enroll MOK → Continue → Yes**, type the password, then **Reboot**. Usually Ubuntu's driver is already signed and this doesn't come up.

```bash
sudo reboot
```

Check that it works. It should list the RTX 5060:

```bash
nvidia-smi
```

### Using the RTX 5060

Apps run on the AMD GPU by default, which saves battery. To run an app on the RTX 5060, right-click it in the app grid and choose **Launch using Discrete Graphics Card**. While it runs, `nvidia-smi` lists it under **Processes**.

## 5. Make it look like GNOME

Ubuntu's desktop is GNOME with Ubuntu's changes on top: a dock, desktop icons, the Yaru theme, an orange accent and Ubuntu's fonts. Instead of undoing them one by one, install the plain **GNOME** session and pick it at login.

```bash
sudo apt install gnome-session gnome-backgrounds sound-theme-freedesktop gnome-tweaks gnome-shell-extension-manager
```

1. Log out: open the top-right menu, click the power button, then **Log Out**.
2. On the login screen, click your name, then the **gear icon** in the bottom-right corner.
3. Choose **GNOME** and log in. Ubuntu remembers this choice.

You now have GNOME's theme, icons, fonts, top bar and overview. Press <kbd>Super</kbd> to open the overview; the dash of pinned apps is at the bottom. Right-click an app there to pin or unpin it.

> [!TIP]
> Ubuntu's own session stays installed. If GNOME ever misbehaves, pick **Ubuntu** from the same gear icon.

### Finish the look

Ubuntu changes three settings in every session, not just its own: the accent colour, the terminal font and the sounds. Set them back to GNOME's:

```bash
gsettings set org.gnome.desktop.interface accent-color 'blue'
gsettings set org.gnome.desktop.interface monospace-font-name 'Adwaita Mono 11'
gsettings set org.gnome.desktop.sound theme-name 'freedesktop'
```

For GNOME's wallpaper, open **Settings → Appearance** and pick **Adwaita**.

Title bars show only a close button, as in GNOME. For minimize and maximize buttons too, open **Tweaks → Windows → Titlebar Buttons**.

### Turn on tray icons

Discord, Steam, Proton VPN and OBS put an icon in the top bar so you can open or quit them. GNOME needs an extension for that. Open **Extension Manager** and, under **System Extensions**, turn on **Ubuntu AppIndicators**.

### Login screen

The login screen uses Ubuntu's theme even in the GNOME session. To switch it, run this and type the number of the entry **without** `Yaru` in its path:

```bash
sudo update-alternatives --config gdm-theme.gresource
```

Do the same for the login screen's icons. If it says there's only one alternative, or none, there's nothing to change:

```bash
sudo update-alternatives --config gdm-icons.gresource
```

You'll see it at the next restart. Ubuntu's logo stays on the login screen.

### Other settings

In **Settings**, turn on:

- **Power → Show Battery Percentage**
- **Displays → Night Light**

Name the laptop (shown on your network and in the terminal) in **Settings → System → About → Device Name**.

Extension worth trying in **Extension Manager**: Clipboard Indicator.

> [!CAUTION]
> Don't install TLP or other power tools. Ubuntu already manages power, and they conflict. Switch power modes from the top-right menu instead.

## 6. Install apps

Each app comes from the source that keeps it updated automatically:

| App                                     | Installed from                  | Why                                                                           |
| --------------------------------------- | ------------------------------- | ----------------------------------------------------------------------------- |
| VLC, qBittorrent, Fastfetch, Steam, Git | Ubuntu                          | Packaged by Ubuntu, updated with the system                                   |
| OBS Studio                              | OBS's own repository (PPA)      | Official, updated with the system                                             |
| Chrome                                  | Google's repository             | Official, updated with the system                                             |
| Proton VPN                              | Proton's repository             | Official, updated with the system                                             |
| VS Code                                 | Microsoft's repository (step 7) | Official, updated with the system                                             |
| Discord, Obsidian                       | Flathub                         | Their `.deb` files have no repository, so you'd download every update by hand |
| Firefox, Thunderbird (optional)         | Flathub                         | Ubuntu's versions are Snaps                                                   |

### Set up Flathub and GNOME Software

**Software** is GNOME's app store. It installs apps from Flathub and updates them in the background:

```bash
sudo apt install flatpak gnome-software gnome-software-plugin-flatpak
```

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

Restart so Flathub apps appear in the app grid:

```bash
sudo reboot
```

### Keep Snap, but don't use it

Leave Snap installed. Ubuntu's upgrades are tested with it, and some of Ubuntu's own tools are Snaps: Firmware Updater, Security Center and, if you chose it, hardware-backed (TPM) disk encryption. They update themselves.

> [!WARNING]
> Don't add apps from Snap:
>
> - Install apps with this guide's commands or from **Software**, not from **App Center** or with `snap install`.
> - Don't `apt install firefox`, `thunderbird` or `chromium-browser`. On Ubuntu those install the Snap. Get Firefox and Thunderbird from Flathub (below).

Optionally, remove the Snap apps you won't use. Firefox, if Chrome is your browser:

```bash
sudo snap remove firefox
```

App Center, since Software replaces it:

```bash
sudo snap remove snap-store
```

Leave everything else in `snap list` alone; it's Snap's support files and Ubuntu's own tools.

### Install the apps

**Chrome.** Google's package also adds Google's repository, so Chrome updates with the system:

```bash
wget -P /tmp https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb && sudo apt install /tmp/google-chrome-stable_current_amd64.deb
```

**VLC, qBittorrent and Fastfetch:**

```bash
sudo apt install vlc qbittorrent fastfetch
```

**OBS Studio.** Press <kbd>Enter</kbd> when asked to add the repository:

```bash
sudo add-apt-repository ppa:obsproject/obs-studio && sudo apt install obs-studio
```

If adding the repository fails, use OBS's official Flatpak instead:

```bash
flatpak install flathub com.obsproject.Studio
```

**Discord and Obsidian.** Answer `y` when asked:

```bash
flatpak install flathub com.discordapp.Discord md.obsidian.Obsidian
```

**Flatseal** lets you change what Flatpak apps can reach, such as a folder outside your home folder:

```bash
flatpak install flathub com.github.tchx84.Flatseal
```

**Firefox and Thunderbird** (optional, Mozilla's official versions):

```bash
flatpak install flathub org.mozilla.firefox org.mozilla.Thunderbird
```

**Proton VPN.** First add Proton's repository:

```bash
wget -P /tmp https://repo.protonvpn.com/debian/dists/stable/main/binary-all/protonvpn-stable-release_1.0.10_all.deb && sudo apt install /tmp/protonvpn-stable-release_1.0.10_all.deb && sudo apt update
```

> [!NOTE]
> If this fails, Proton has released a newer version of the file. Get the current command from [Proton's guide](https://protonvpn.com/support/official-linux-vpn-ubuntu).

Then install the app:

```bash
sudo apt install proton-vpn-gnome-desktop
```

**Steam.** Steam needs 32-bit libraries, so this turns them on first:

```bash
sudo dpkg --add-architecture i386 && sudo apt update && sudo apt install steam-installer
```

Many games also need the 32-bit NVIDIA library. This picks the version that matches your driver:

```bash
sudo apt install libnvidia-gl-$(nvidia-smi --query-gpu=driver_version --format=csv,noheader | cut -d. -f1):i386
```

**Run Steam games on the RTX 5060 (optional).** Right-click Steam in the app grid. If you see **Launch using Integrated Graphics Card**, Steam already uses the RTX 5060; skip this. Otherwise, make Steam always start on it, so games launched from it do too:

```bash
mkdir -p ~/.local/share/applications && cp /usr/share/applications/steam.desktop ~/.local/share/applications/ && sed -i '/^\[Desktop Entry\]/a PrefersNonDefaultGPU=true' ~/.local/share/applications/steam.desktop
```

Then quit Steam fully (Steam menu → **Exit**) and turn off its start-at-login option. While a game runs, `nvidia-smi` should list it under **Processes**.

### Finding more apps

Search in **Software**. When an app is offered from both Ubuntu and Flathub, use Ubuntu's version if it's recent enough, Flathub's if it's missing or old.

## 7. Set up Git and VS Code

Git is the version-control tool most code projects use:

```bash
sudo apt install git
```

Replace the name and email with yours, then paste:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

**VS Code**, from Microsoft's package. When asked whether to add Microsoft's repository, choose **Yes** so VS Code updates with the system:

```bash
wget -O /tmp/code.deb "https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-x64" && sudo apt install /tmp/code.deb
```

## 8. Set up backups

**Backups** (Déjà Dup) saves your personal files to Google Drive. Install it with `rclone`, which it needs to reach Google Drive:

```bash
sudo apt install deja-dup rclone
```

Open **Backups**, choose **Google Drive** as the storage location, sign in, and turn on automatic backups.

> [!WARNING]
> Keep the encryption password safe. Without it, your backups can't be restored.

- Backups cover your files, not your apps. To rebuild the laptop, reinstall Ubuntu, follow this guide, then restore your files.
- Once the first backup finishes, test it: in **Backups → Restore**, restore one file.

## 9. Maintenance and upgrades

### Weekly updates

Let **Software Updater** and **Software** do it when they pop up, or run:

```bash
sudo apt update && sudo apt full-upgrade
```

```bash
flatpak update
```

### Monthly clean-up

Removes packages and Flatpak runtimes nothing uses any more:

```bash
sudo apt autoremove
```

```bash
flatpak uninstall --unused
```

### Upgrading to the next Ubuntu LTS

A new LTS comes out every two years. **Software Updater** offers it once the first point release is out (for example 28.04.1, around August 2028).

> [!TIP]
> Wait for that offer instead of upgrading on release day. By then the early bugs are fixed and the outside repositories (Chrome, OBS and so on) support the new release.

#### What happens to this setup

Nothing in this guide blocks an upgrade, but a few things need a check afterwards:

| What                                    | After the upgrade                          | What to do                                      |
| --------------------------------------- | ------------------------------------------ | ----------------------------------------------- |
| Ubuntu packages, NVIDIA driver          | Upgraded automatically                     | Check `nvidia-smi` works                        |
| Flatpak apps                            | Unaffected                                 | Nothing                                         |
| Chrome, VS Code, Proton VPN, OBS        | Their repositories are switched off        | Turn them back on ("After", step 3)             |
| GNOME session and settings              | Kept                                       | If it won't start, log in to **Ubuntu** instead |
| Login screen theme                      | Kept                                       | If it looks broken, see "If something breaks"   |
| Extensions you added                    | Often break with a new GNOME               | Turn off before, back on one at a time after    |
| Removed Snap apps (Firefox, App Center) | May come back                              | Remove them again                               |
| 32-bit NVIDIA library                   | Usually upgraded with the driver           | If games stop starting, re-run its command      |
| Steam's RTX 5060 shortcut               | Kept, but Steam's updates don't refresh it | If Steam won't open, delete it and redo it      |

#### Before

- [ ] Plug in the charger and make sure at least 10 GB is free (`df -h /` shows it).
- [ ] Run a backup (step 8).
- [ ] Install all current updates, then restart: `sudo apt update && sudo apt full-upgrade`.
- [ ] In **Extension Manager**, turn off the extensions you added (such as Clipboard Indicator). Leave **Ubuntu AppIndicators** on.

#### During

Click **Upgrade** in **Software Updater**. A **Foreign Packages Installed** message is normal (it means Chrome, VS Code and so on); continue. Approve removing obsolete packages, then restart.

#### After

1. Log in and check that everything works.
2. Check the NVIDIA driver with `nvidia-smi`.
3. Open **Software & Updates → Other Software** and tick the entries for Chrome, VS Code, Proton VPN and OBS. Then run `sudo apt update`. If OBS's line shows an error, its repository doesn't support the new release yet: untick it and try again in a few weeks, or switch to OBS's Flatpak.
4. Turn your extensions back on one at a time. If the desktop misbehaves, turn the last one off and wait for an update to it.
5. Run `snap list` and remove Firefox or App Center again if they came back.

#### If something breaks

<details>
<summary><b>GNOME session won't start or looks wrong</b></summary>

On the login screen, click the gear icon, choose **Ubuntu**, and fix things from there.

</details>

<details>
<summary><b>Login screen looks broken</b></summary>

Reset it to Ubuntu's default:

```bash
sudo update-alternatives --auto gdm-theme.gresource && sudo update-alternatives --auto gdm-icons.gresource
```

</details>

<details>
<summary><b>Steam won't open after an update</b></summary>

Delete the shortcut copy, then redo the RTX 5060 part of step 6:

```bash
rm ~/.local/share/applications/steam.desktop
```

</details>
