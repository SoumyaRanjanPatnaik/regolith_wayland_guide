# Regolith on Wayland Installation Guide

Before starting to follow this guide, please note that the packages you are installing contain code that is yet to be merged. This guide is for testing and experimentation purpose. However, if you encounter any problems following the guide, you can create an issue and I'll try to help you to the best of my ability.

# What doesn't work?

- i3-status (no status icons or buttons on panel)
- gnome-session-quit (works as cli, prompt doesn't work)
- glitches in regolith-control-center / gnome-control-center (timedate panel, accounts panel)
- sway refresh/reload (slow changing of looks)
- Ilia keybindings page / cheat-sheet
- Ilia notifications page
- Many X11 based applications might require setting environment variables and may behave differently
- Regolith-inputd needs to be started manually (NEEDS FURTHER DEBUGGING)
- Settings keyboard layouts from gnome-control-center

## Adding the experimental key to apt

Most of the work relating to `regolith on wayland` is packaged and published in the regolith experimental package repository. To add it to the `apt` package manager, use the following command

```bashd set of services that connect users with the network resources they need to get their work d
sudo mkdir -p /etc/apt/sources.list.d/
sudo touch /etc/apt/sources.list.d/regolith-experimental.list
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/regolith-archive-keyring.gpg] https://regolith-desktop.org/experimental-ubuntu-kinetic-amd64 kinetic main" | sudo tee /etc/apt/sources.list.d/regolith-experimental.list

sudo apt update
sudo apt upgrade
sudo apt install --fix-broken
```

## Installing packages
Remove sway if already installed and install `regolith-session-sway` package to install all the wayland related packages.

```bash
sudo apt remove sway # remove sway to avoid conflicts
sudo apt install regolith-sesion-sway
```

## Install lightdm-gtk-greeter

Slick greeter crashes when trying to load wayland sessions. You'll may have a better time with `lightdm-gtk-greeter`.

```bash
sudo apt install lightdm-gtk-greeter
```

# Uninstall Regolith on Wayland 
This will remove the experimental stage and all the packages from wayland.
```bash
sudo apt remove --purge sudo apt remove regolith-session-sway
sudo rm /etc/apt/sources.list.d/regolith-experimental.list
sudo apt update && sudo apt upgrade
```

# Custom configurations

## XResources

While `XResources` are not supported on wayland, `trawl` can parse your `XResources` files and acts as a replacement to `XResources`. You can hence make changes to various aspects of regolith using `XResources`. The default XResources file for regolith should be located in `~/.config/regolith2/XResources`. If you don't have one, create using

```bash
mkdir -p ~/.config/regolith2
touch ~/.config/regolith2/XResources
```

You can use the `XResources` cheetsheet located on the [Regolith Desktop Site](https://regolith-desktop.com/docs/reference/xresources/).

## Window Manager (Sway) configuration

If you need to make any changes to your `sway-regolith` configs and can't use XResources to accomplish it, you can add config partials to `~/.config/regolith2/sway/config.d`. You need to manually create this directory using the following command.

```bash
mkdir -p ~/.config/regolith2/sway/config.d
```

The files you create in here use standard sway configuration format. You may also define variables in the config that use your keys in `XResources` using the `set_from_resource` command.

```
set_from_resource $var_name resource_name [fallback_value]
```

**NOTE 2**: `sway-regolith` is a fork of `sway` that adds `set_from_resource` command to sway configurations allowing for complete compatibility with all of regolith's configs.

**NOTE 2**: If you don't provide a fallback value and the resource couldn't be found in `trawldb`, `sway-regolith` will present you with an error.

# Things to keep in mind

## Keyboard layout

You can set the keyboard layout from `gnome-control-center` or `regolith-control-center`.

## Clamshell Mode

Add the following config to any of the files present in `~/sway/.config/reglith2/sway/config.d`.

```
set $laptop <connector_or_name>
bindswitch --reload --locked lid:on output $laptop disable
bindswitch --reload --locked lid:off output $laptop enable
```

Replace `<connector_or_name>` with the name or connector of your laptop's inbuilt display.

`sway` has no way of knowing which display is the laptop's in-built display. So, if you want to use sway with clamshell mode, i.e. closing the laptop lid disables the laptop display and uses your external display as the primary display, you need to find out the name or connector of your laptop's in-built display.

To list all the available displays, use the follwoing command.

```bash
swaymsg -t get_outputs
```

This should give you the list of all the conected displays. I should look something like this.

```
Output eDP-1 'Najing CEC Panda FPD Technology CO. ltd 0x0050 Unknown'
  Current mode: 1920x1080 @ 120.003 Hz
  Position: 0,0
  Scale factor: 1.100000
  Scale filter: linear
  Subpixel hinting: unknown
  Transform: normal
  Workspace: 4: Music
  Max render time: off
  Adaptive sync: enabled
  Available modes:
    1920x1080 @ 120.003 Hz
    1920x1080 @ 96.032 Hz
    1920x1080 @ 72.024 Hz
    1920x1080 @ 60.002 Hz
    1920x1080 @ 60.002 Hz
    1920x1080 @ 50.014 Hz
    1920x1080 @ 48.001 Hz
    1680x1050 @ 120.003 Hz
    1280x1024 @ 120.003 Hz
    1440x900 @ 120.003 Hz
    1280x800 @ 120.003 Hz
    1280x720 @ 120.003 Hz
    1024x768 @ 120.003 Hz
    800x600 @ 120.003 Hz
    640x480 @ 120.003 Hz

Output HDMI-A-1 'LG Electronics LG ULTRAGEAR 204NTDV9E466' (focused)
  Current mode: 2560x1440 @ 99.946 Hz
  Position: 1745,0
  Scale factor: 1.000000
  Scale filter: nearest
  Subpixel hinting: unknown
  Transform: normal
  Workspace: 8
  Max render time: off
  Adaptive sync: enabled
  Available modes:
    2560x1440 @ 99.946 Hz
    3840x2160 @ 60.000 Hz (16:9)
    3840x2160 @ 59.940 Hz (16:9)
    3840x2160 @ 50.000 Hz (16:9)
    3840x2160 @ 30.000 Hz (16:9)
    3840x2160 @ 29.970 Hz (16:9)
    3840x2160 @ 25.000 Hz (16:9)
    3840x2160 @ 24.000 Hz (16:9)
    3840x2160 @ 23.976 Hz (16:9)
    2560x1440 @ 96.005 Hz
    2560x1440 @ 74.971 Hz
    2560x1440 @ 72.003 Hz
    2560x1440 @ 60.008 Hz
    2560x1440 @ 59.951 Hz
    2560x1440 @ 50.006 Hz
    2560x1440 @ 48.002 Hz
    1920x1200 @ 99.946 Hz
    1920x1080 @ 120.000 Hz (16:9)
    1920x1080 @ 119.880 Hz (16:9)
    1920x1080 @ 60.000 Hz
    1920x1080 @ 60.000 Hz (16:9)
```

In this case the inbuilt display is connected using the connector `eDP-1`. Generally connectors starting `eDP` correspond to the in-built display. To enable clamshell mode for `eDP-1`, add the following to the file `~/.config/regolith2/sway/config.d/clamshell`.

```
set $laptop eDP-1
bindswitch --reload --locked lid:on output $laptop disable
bindswitch --reload --locked lid:off output $laptop enable
```

## Use existing i3 user configs

If you were previously using `Regolith on X11`, there's a chance you had config partials located at `~/.config/regolith2/i3/config.d`. If you want to reuse the same configurations in sway as well, you have three options

1. Copy the config partials to `~/.config/regolith2/sway/config.d` (RECOMMENDED)

   ```bash
   mkdir -p ~/.config/regolith2/sway/config.d
   cp ~/.config/regolith2/i3/config.d/* ~/.config/regolith2/sway/config.d
   ```

2. Symlink config partials to `~/.config/regolith2/sway/config.d` (NOT RECOMMENDED)

   ```bash
   mkdir -p ~/.config/regolith2/sway/config.d
   ln -s $HOME/.config/regolith2/i3/config.d/* ~/.config/regolith2/sway/config.d
   ```

3. Include the i3 config partials directory in one of the files in ~/.config/regolith2/sway/config.d`
   ```bash
   mkdir -p ~/.config/regolith2/sway/config.d
   echo "include ~/.config/regolith2/i3/config.d/*" > ~/.config/regolith2/sway/config.d/i3_include
   ```

**NOTE**: While `sway` serves as a drop-in replacement for `i3`, some configurations may not produce the expected behaviour. So it is advisable for you to copy over your custom configurations so that changing configurations for `i3` doesn't hamper your `sway` experience.
