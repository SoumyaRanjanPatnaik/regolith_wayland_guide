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

```bash
sudo mkdir -p /etc/apt/sources.list.d/
sudo touch /etc/apt/sources.list.d/regolith-experimental.list
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/regolith-archive-keyring.gpg] https://regolith-desktop.org/experimental-debian-testing-amd64 testing main" | sudo tee /etc/apt/sources.list.d/regolith-experimental.list

sudo apt update
sudo apt upgrade
```

## Uninstalling conflicting packages

```bash
sudo apt remove regolith-session-flashback regolith-look-default
sudo apt remove sway # remove sway to avoid conflicts
```

**Note**: **DO NOT** reboot or logout at this point. You won't have any regolith-sessions installed. But if you accidentally reboot or logout, you can still continue to follow this guide from a tty (accessible using ctrl+alt+f1) and everything should go back to the way it was after installing the packages in the next step.

## Installing packages

Download the `packages.tar.gz` file from the [release section](https://github.com/SoumyaRanjanPatnaik/regolith_wayland_guide/releases/tag/v0.1.2-alpha) and extract them. You can also do this using the following commands

```bash
mkdir regolith_sway
cd regolith_sway
wget https://github.com/SoumyaRanjanPatnaik/regolith_wayland_guide/releases/download/v0.1.2-alpha/packages.tar.gz.1
tar -xvzf packages.tar.gz
```

**NOTE**: Errors during this part of the installation is expected behaviour as dpkg cannot install the dependencies using apt. Installing unmet dependencies using `sudo apt install --fix-broken` and retrying the installation should be successful.

Now you need to install the regolith-look-default and regolith-look-default-loader packages.

```bash
sudo dpkg -i  regolith-look-default-loader_0.7.5_amd64.deb regolith-look-default_0.7.5_amd64.deb
```

Now install all the packages:

```bash
sudo dpkg -i *.deb
```

At this point, **you should get several errors due to unmet dependencies**. Install these dependencies using

```bash
sudo apt install --fix-broken
```

Now, try to install all the packages again. This time you shouldn't get any more errors.

```bash
sudo dpkg -i *.deb
```

Apply hold on all of the packages we just installed to prevent them from being upgraded (this is important if you don't want to break your system after upgrades).

```bash
sudo apt-mark hold regolith-displayd regolith-inputd ilia regolith-ftue regolith-sway-config "regolith-look-default*" "regolith-session-*"
```

## Install gdm3 / lightdm-gtk-greeter

Slick greeter crashes when trying to load wayland sessions. You'll may have a better time with `lightdm-gtk-greeter`, but for some reason `gsd-rfkill` doesn't seem to work as the bluetooth panel of gnome-control-center says it cannot find any bluetooth adapters. `gdm3` works the best and has excellent compatibility with everything gnome.

**Note**: If you don't want to pull in gnome dependencies (including gnome-shell) you might want to use `lightdm-gtk-greeter`.

```bash
# sudo apt install lightdm-gtk-greeter
sudo apt install gdm3
```

## Reinstalling the looks

While uninstalling `regolith-look-default`, all the installed looks got removed as well. You can either install the looks you want one by one, or you can install them all using

```bash
sudo apt install "regolith-look-*"
```

## Screen sharing on Wayland

```bash
sudo apt install xdg-desktop-portal xdg-desktop-portal-wlr
```

# Undo the changes

**NOTE**: This portion of the guide has not been tested yet.

The following commands should (in theory) undo all the changes made to the regolith environment. If something undesirable happens, you need to follow all the steps you did to install `Regolith on Wayland` and your system should go back to normal.

```bash
sudo apt remove --purge regolith-displayd regolith-inputd regolith-session-sway regolith-sway-config
sudo apt-mark unhold ilia regolith-ftue "regolith-look-default*" "regolith-session-*"
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

The keboard layout will change to EN-US by default. If you wan't to change the keyboard layout, you'll have to add the following snippet in any of the files present in the `~/.config/regolith2/sway/config.d` directory.

```
input type:keyboard {
    xkb_layout <layout_code> [, optional_layout1, optional_layout2]
}
```

The `<layout_code>` needs to be a valid layout code like `us`, `es`, `de`, `eu`. etc. Multiple layouts can be specified by separating them with commas.

For example, if you want to set the layout to `es`, you can create the file `~/.config/regolith2/sway/config.d/kbd_layout_es` and add the following to it.

```
input type:keyboard {
    xkb_layout es
}
```

For more input related information, you can refer to the [sway-input man page](https://man.archlinux.org/man/sway-input.5.

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
