# Regolith on Wayland Installation Guide

## Adding the experimental key to apt

Most of the work relating to `regolith on wayland` is packaged and published in the regolith experimental package repository. To add it to the `apt` package manager, use the following command

```
sudo mkdir -p /etc/apt/sources.list.d/
sudo touch /etc/apt/sources.list.d/regolith-experimental.list
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/regolith-archive-keyring.gpg] https://regolith-desktop.org/experimental-debian-testing-amd64 testing main" | sudo tee /etc/apt/sources.list.d/regolith-experimental.list
```

## Installing packages

Download the package from the release section. Extract and install using the following commands
```bash
mkdir regolith_sway 
cd regolith_sway
tar -xvzf ../packages.tar.xz #extract all packages
sudo dpkg -i *.deb #install all packages
```

## Install gdm3 / lightdm-gtk-greeter

Slick greeter crashes when trying to load wayland sessions. You'll may have a better time with `lightdm-gtk-greeter`, but for some reason `gsd-rfkill` doesn't seem to work as the bluetooth panel of gnome-control-center says it cannot find any bluetooth adapters. `gdm3` works the best and has excellent compatibility with everything gnome. 

**Note**: If you don't want to pull in gnome dependencies (including gnome-shell) you might want to use `lightdm-gtk-greeter`.

```bash
# sudo apt install lightdm-gtk-greeter
sudo apt install gdm3
```

## Screen sharing on Wayland

```
sudo apt install xdg-desktop-portal xdg-desktop-portal-wlr
```

## Install regolith-session-sway with apt

```
sudo apt install regolith-session-sway
```

# What doesn't work?
- i3-status (no status icons or buttons on panel)
- gnome-session-quit (works as cli, prompt doesn't work)
- glitches in regolith-control-center / gnome-control-center (timedate panel, accounts panel)
- sway refresh/reload (slow changing of looks)
- You tell me...