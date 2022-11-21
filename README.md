# Regolith on Wayland Installation Guide
Before starting to follow this guide, please note that the packages you are installing contain code that is yet to be merged. This guide is for testing and experimentation purpose. However, if you encounter any problems following the guide, you can create an issue and I'll try to help you to the best of my ability. 
## Adding the experimental key to apt

Most of the work relating to `regolith on wayland` is packaged and published in the regolith experimental package repository. To add it to the `apt` package manager, use the following command

```
sudo mkdir -p /etc/apt/sources.list.d/
sudo touch /etc/apt/sources.list.d/regolith-experimental.list
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/regolith-archive-keyring.gpg] https://regolith-desktop.org/experimental-debian-testing-amd64 testing main" | sudo tee /etc/apt/sources.list.d/regolith-experimental.list

sudo apt update
sudo apt upgrade
```
## Uninstalling conflicting packages
```
sudo apt remove regolith-session-flashback regolith-look-default
```

**Note**: **DO NOT** reboot or logout at this point. You won't have any regolith-sessions installed. But if you accidentally reboot or logout, you can still continue to follow this guide from a tty (accessible using ctrl+alt+f1) and everything should go back to the way it was after installing the packages in the next step.

## Installing packages
Download the `packages.tar.gz` file from the [release section](https://github.com/SoumyaRanjanPatnaik/regolith_wayland_guide/releases/tag/v0.1-alpha). `cd` into the dierctory with the downloaded file and extract using the following commands
```bash
mkdir regolith_sway 
cd regolith_sway
tar -xvzf ../packages.tar.xz
```

Now you need to install the regolith-look-default and regolith-look-default-loader packages. 
```
sudo dpkg -i  regolith-look-default-loader_0.7.5_amd64.deb regolith-look-default_0.7.5_amd64.deb 
```

Now install all the packages:
```
sudo dpkg -i *.deb
```

At this point, you should get several errors due to unmet dependencies. Install these dependencies using
```
sudo apt install --fix-broken
```

Now, try to install all the packages again. This time you shouldn't get any more errors.
```
sudo dpkg -i *.deb
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

# What doesn't work?
- i3-status (no status icons or buttons on panel)
- gnome-session-quit (works as cli, prompt doesn't work)
- glitches in regolith-control-center / gnome-control-center (timedate panel, accounts panel)
- sway refresh/reload (slow changing of looks)
- You tell me...
