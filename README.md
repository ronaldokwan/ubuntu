# Ubuntu Setup: My Laptop

My checklist for a fresh Ubuntu 26.04 LTS install on this laptop. Do the steps in order; the folded "Optional" parts can be skipped. It takes about an hour, plus downloads.

| Part               | Model                          | Setup needed                                                                        |
| ------------------ | ------------------------------ | ----------------------------------------------------------------------------------- |
| CPU + graphics     | AMD Ryzen 7 260, Radeon 780M   | Video acceleration check ([step 3](#3-install-codecs-and-check-video-acceleration)) |
| Dedicated graphics | NVIDIA GeForce RTX 5060 Laptop | NVIDIA driver ([step 4](#4-install-the-nvidia-driver))                              |

## Before you start

<details>
<summary>New to the terminal? Read this first</summary>

- Open it with <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>T</kbd>.
- Paste with <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>V</kbd> (plain <kbd>Ctrl</kbd> + <kbd>V</kbd> doesn't work there).
- When `sudo` asks for your password, nothing appears as you type. That's normal; type it and press <kbd>Enter</kbd>.
- When asked `Do you want to continue? [Y/n]`, press <kbd>Enter</kbd>.

</details>

> [!IMPORTANT]
> Every choice in this guide follows these rules, so that future Ubuntu upgrades go smoothly. Keep following them when you add things later:
>
> - Install software only from Ubuntu, from the maker's official repository, or from Flathub.
> - Never install the NVIDIA driver from nvidia.com. Ubuntu's driver is rebuilt for every kernel update; NVIDIA's installer isn't.
> - Don't uninstall `ubuntu-desktop` or Snap. Ubuntu's upgrades expect both.
> - Avoid PPAs (unless run by the app's own makers, like OBS's) and `curl ... | bash` install scripts. They're the most common reason an upgrade fails.

## 1. Update the system

Install everything released since the install image was made, then restart:

```bash
sudo apt update && sudo apt full-upgrade && sudo reboot
```

## 2. Update firmware

This updates the BIOS and device firmware.

> [!WARNING]
> Plug in the charger first. Losing power halfway through a firmware update can leave the laptop unable to start.

```bash
fwupdmgr refresh --force && fwupdmgr update
```

"No updates available" is normal. If it asks to reboot, do.

## 3. Install codecs and check video acceleration

Codecs for common video and audio formats, plus Microsoft's fonts:

```bash
sudo apt install ubuntu-restricted-extras
```

Partway through, a font licence screen appears. Press <kbd>Tab</kbd> to select `<Ok>` and <kbd>Enter</kbd>, then on the next screen use the arrow keys to select `<Yes>` and press <kbd>Enter</kbd>.

Then check that the Radeon 780M decodes video in hardware:

```bash
sudo apt install vainfo && vainfo
```

The list should include `VAProfileH264High`, `VAProfileHEVCMain` and `VAProfileAV1Profile0`, each with `VAEntrypointVLD`.

## 4. Install the NVIDIA driver

If you ticked "Install third-party software" during installation, the driver may already be there. Check:

```bash
nvidia-smi
```

If it lists the RTX 5060, skip to [Using the RTX 5060](#using-the-rtx-5060). If not, install the driver Ubuntu recommends (for RTX 50 cards, its name ends in `-open`) and restart:

```bash
sudo ubuntu-drivers install && sudo reboot
```

With Secure Boot on, the install may ask you to set a password. Make up a short one with only letters and numbers. After the restart, a blue MOK Management screen appears: choose Enroll MOK → Continue → Yes, type the password, then Reboot.

Run `nvidia-smi` again. It should now list the RTX 5060.

### Using the RTX 5060

Apps run on the AMD GPU by default, which saves battery. To run one on the RTX 5060, right-click it in the app grid and choose **Launch using Discrete Graphics Card**. While it runs, `nvidia-smi` lists it under Processes.

## 5. Make it look like GNOME

Ubuntu's desktop is GNOME plus Ubuntu's own dock, desktop icons, Yaru theme, orange accent and fonts. The plain GNOME session drops all of that at once:

```bash
sudo apt install gnome-session gnome-backgrounds sound-theme-freedesktop gnome-tweaks gnome-shell-extension-manager
```

1. Log out: top-right menu → power button → Log Out.
2. On the login screen, click your name, then the gear icon in the bottom-right corner.
3. Choose **GNOME** and log in. Ubuntu remembers the choice.

If GNOME ever misbehaves, pick Ubuntu from the same gear icon; Ubuntu's session stays installed.

Ubuntu changes three settings in every session, not just its own: the accent colour, the terminal font and the sounds. Put them back to GNOME's:

```bash
gsettings set org.gnome.desktop.interface accent-color 'blue'
gsettings set org.gnome.desktop.interface monospace-font-name 'Adwaita Mono 11'
gsettings set org.gnome.desktop.sound theme-name 'freedesktop'
```

A few more things to set:

- Discord, Steam, Proton VPN and OBS need an extension to show their tray icon. In Extension Manager, under System Extensions, turn on **Ubuntu AppIndicators**.
- For GNOME's wallpaper, go to Settings → Appearance and pick Adwaita.
- GNOME windows only have a close button. To add minimize and maximize, open Tweaks → Windows → Titlebar Buttons.
- Press <kbd>Super</kbd> for the overview. Right-click an app in the dash at the bottom to pin or unpin it.

Don't install TLP or other power tools; they conflict with Ubuntu's own power management. Switch power modes from the top-right menu.

<details>
<summary>Optional: GNOME-style login screen</summary>

The login screen keeps Ubuntu's theme. To switch it, run each command and type the number of the entry without `Yaru` in its path. If one says there's only one alternative, or none, there's nothing to change.

```bash
sudo update-alternatives --config gdm-theme.gresource
```

```bash
sudo update-alternatives --config gdm-icons.gresource
```

The change shows after the next restart. Ubuntu's logo stays.

</details>

<details>
<summary>Optional: other settings worth changing</summary>

- Settings → Power → Show Battery Percentage
- Settings → Displays → Night Light
- Settings → System → About → Device Name (the laptop's name on your network and in the terminal)
- The Clipboard Indicator extension, from Extension Manager

</details>

## 6. Install apps

| App                                     | From                       | Why                                            |
| --------------------------------------- | -------------------------- | ---------------------------------------------- |
| VLC, qBittorrent, Fastfetch, Steam, Git | Ubuntu                     | Packaged and kept up to date by Ubuntu         |
| Chrome, Proton VPN, VS Code, OBS Studio | The maker's own repository | Official builds, delivered by Ubuntu's updater |
| Discord, Obsidian, Bitwarden            | Flathub                    | Their `.deb` files don't update themselves     |
| Firefox, Thunderbird (optional)         | Flathub                    | Ubuntu's versions are Snaps                    |

### Set up Flathub

Software is GNOME's app store; it installs Flathub apps and updates them in the background. This installs it, adds Flathub, and restarts so Flathub apps show up in the app grid:

```bash
sudo apt install flatpak gnome-software gnome-software-plugin-flatpak && flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo && sudo reboot
```

Leave Snap installed: Ubuntu's upgrades are tested with it, and Firmware Updater and Security Center are Snaps. Just don't add apps from it. Use this guide's commands or Software instead of App Center or `snap install`, and don't `apt install firefox`, `thunderbird` or `chromium-browser`, because on Ubuntu those install the Snap.

### Install the apps

**Chrome.** Google's package also adds Google's repository:

```bash
wget -P /tmp https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb && sudo apt install /tmp/google-chrome-stable_current_amd64.deb
```

**VLC, qBittorrent and Fastfetch:**

```bash
sudo apt install vlc qbittorrent fastfetch
```

**Discord, Obsidian, Bitwarden and Flatseal.** Flatseal controls which folders Flatpak apps can reach. Answer `y` when asked:

```bash
flatpak install flathub com.discordapp.Discord md.obsidian.Obsidian com.bitwarden.desktop com.github.tchx84.Flatseal
```

To fill in passwords on websites, also add the Bitwarden extension to Chrome from the Chrome Web Store.

**OBS Studio.** Press <kbd>Enter</kbd> when asked to add the repository:

```bash
sudo add-apt-repository ppa:obsproject/obs-studio && sudo apt install obs-studio
```

If adding the repository fails, use the Flatpak instead: `flatpak install flathub com.obsproject.Studio`.

**Proton VPN.** First Proton's repository:

```bash
wget -P /tmp https://repo.protonvpn.com/debian/dists/stable/main/binary-all/protonvpn-stable-release_1.0.10_all.deb
sudo apt install /tmp/protonvpn-stable-release_1.0.10_all.deb && sudo apt update
```

Then the app:

```bash
sudo apt install proton-vpn-gnome-desktop
```

If the download fails, Proton has released a newer file. [Proton's guide](https://protonvpn.com/support/official-linux-vpn-ubuntu) has the current command.

**Steam.** It needs 32-bit packages turned on first:

```bash
sudo dpkg --add-architecture i386 && sudo apt update && sudo apt install steam-installer
```

Many games also need the 32-bit NVIDIA library. This installs the one matching your driver:

```bash
sudo apt install libnvidia-gl-$(nvidia-smi --query-gpu=driver_version --format=csv,noheader | cut -d. -f1):i386
```

From now on, `sudo apt update` may print `doesn't support architecture 'i386'` for Chrome, VS Code or OBS. You can ignore it.

<details>
<summary>Optional: run all Steam games on the RTX 5060</summary>

Right-click Steam in the app grid. If you see "Launch using Integrated Graphics Card", it already uses the RTX 5060 and you can skip this. Otherwise:

```bash
mkdir -p ~/.local/share/applications && cp /usr/share/applications/steam.desktop ~/.local/share/applications/ && sed -i '/^\[Desktop Entry\]/a PrefersNonDefaultGPU=true' ~/.local/share/applications/steam.desktop
```

Quit Steam fully (Steam menu → Exit) and turn off its start-at-login option. While a game runs, `nvidia-smi` should list it.

</details>

<details>
<summary>Optional: Firefox and Thunderbird</summary>

Mozilla's official versions:

```bash
flatpak install flathub org.mozilla.firefox org.mozilla.Thunderbird
```

</details>

<details>
<summary>Optional: remove Snap apps you won't use</summary>

Firefox's Snap (if you use Chrome or the Flatpak) and App Center (Software replaces it):

```bash
sudo snap remove firefox snap-store
```

Leave everything else in `snap list` alone. It's Snap's support files and Ubuntu's own tools.

</details>

For anything else, search in Software. If an app is offered from both Ubuntu and Flathub, take Ubuntu's version when it's recent enough and Flathub's when it's missing or old.

## 7. Set up Git and VS Code

Install Git, then set your own name and email:

```bash
sudo apt install git
```

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

For VS Code, answer Yes when it asks to add Microsoft's repository, so it keeps getting updates:

```bash
wget -O /tmp/code.deb "https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-x64" && sudo apt install /tmp/code.deb
```

## 8. Set up backups

Backups (Déjà Dup) copies your personal files to Google Drive, and needs `rclone` to reach it:

```bash
sudo apt install deja-dup rclone
```

Open Backups, choose Google Drive, sign in, and turn on automatic backups. Once the first one finishes, restore a single file from the Restore tab to make sure it works.

> [!WARNING]
> Keep the encryption password somewhere safe. Without it, the backups can't be restored.

Backups cover your files, not your apps. To rebuild the laptop, reinstall Ubuntu, follow this guide, then restore your files.

## 9. Maintenance and upgrades

Once a week, let Software Updater and Software install their updates when they pop up, or run:

```bash
sudo apt update && sudo apt full-upgrade && flatpak update
```

Every month or so, clear out what nothing uses any more:

```bash
sudo apt autoremove && flatpak uninstall --unused
```

### Upgrading to the next Ubuntu LTS

Software Updater offers the next LTS once its first point release is out (28.04.1, around August 2028). Wait for that instead of upgrading on release day.

#### Before

- [ ] Plug in the charger and check there's at least 10 GB free (`df -h /`).
- [ ] Run a backup.
- [ ] Install all current updates and restart.
- [ ] In Extension Manager, turn off the extensions you added yourself. Leave Ubuntu AppIndicators on.

#### During

Click Upgrade in Software Updater. A "Foreign Packages Installed" message is expected; continue. Approve removing obsolete packages, then restart.

#### After

1. Check `nvidia-smi` still lists the RTX 5060.
2. In Software & Updates → Other Software, tick the entries for VS Code, Proton VPN and OBS. Chrome's turns itself back on. Then update everything:

   ```bash
   sudo apt update && sudo apt full-upgrade && flatpak update
   ```

   If OBS's line shows an error, its repository doesn't support the new release yet. Untick it and try again in a few weeks, or switch to OBS's Flatpak.
3. Turn your extensions back on one at a time. If the desktop misbehaves, turn the last one off again.
4. Run `snap list` and remove Firefox or App Center again if they came back.

<details>
<summary>What happens to each part of this setup</summary>

| What                                    | After the upgrade                          | What to do                                   |
| --------------------------------------- | ------------------------------------------ | -------------------------------------------- |
| Ubuntu packages, NVIDIA driver          | Upgraded automatically                     | Check `nvidia-smi` works                     |
| Flatpak apps                            | Unaffected                                 | `flatpak update` to match the new driver     |
| Chrome                                  | Switched off, then Chrome turns it back on | Nothing                                      |
| VS Code, Proton VPN, OBS                | Their repositories are switched off        | Turn them back on ("After", step 2)          |
| GNOME session and settings              | Kept                                       | If it won't start, log in to Ubuntu instead  |
| Login screen theme                      | Kept                                       | If it looks broken, see below                |
| Extensions you added                    | Often break with a new GNOME               | Turn off before, back on one at a time after |
| Removed Snap apps (Firefox, App Center) | May come back                              | Remove them again                            |
| 32-bit NVIDIA library                   | Usually upgraded with the driver           | If games stop starting, re-run its command   |
| Steam's RTX 5060 shortcut               | Kept, but Steam's updates don't refresh it | If Steam won't open, see below               |

</details>

<details>
<summary>If something breaks</summary>

The upgrade stops because of foreign packages: uninstall the apps that came from outside Ubuntu (OBS, VS Code, Proton VPN, Chrome), run the upgrade again, then reinstall them with steps 6 and 7.

`nvidia-smi` fails: reinstall the recommended driver and restart.

```bash
sudo ubuntu-drivers install && sudo reboot
```

The GNOME session won't start or looks wrong: click the gear icon on the login screen, choose Ubuntu, and fix things from there.

The login screen looks broken: reset it to Ubuntu's default.

```bash
sudo update-alternatives --auto gdm-theme.gresource && sudo update-alternatives --auto gdm-icons.gresource
```

Steam won't open: delete the shortcut copy, then redo the optional RTX 5060 part of step 6.

```bash
rm ~/.local/share/applications/steam.desktop
```

</details>
