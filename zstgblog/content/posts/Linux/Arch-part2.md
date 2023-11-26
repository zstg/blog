+++
title = "Configuring Arch post-install"
author = "ZeStig"
authorTwitter = "" #do not include @
cover = ""
tags = ["Linux", "Arch"]
keywords = ["Arch", "Linux"]
description = "This blog serves as a how-to on setting up Arch post-install. This involves configuring things that are usually installed on 'normie' distros."
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

I'm using Arch with Hyprland, and so here's a quick reference for getting things up-to-speed.
## Initial
```bash
sudo pacman -S waybar mako libnotify starship wl-clipboard hyprland xdg-desktop-portal-hyprland git ripgrep fd bat blueman brave-bin librewolf-bin \
     eza fuse2 git-delta gpm grim slurp swappy hugo keepassxc kitty ly nano-syntax-highlighting zsh-syntax-highlighting neofetch  \
     networkmanager-applet noto-fonts oh-my-zsh-git openssh pacman-contrib pinentry pipewire-pulse playerctl qt5-wayland qt6-wayland tree wireplumber man-pages # provides man pages for some C functions/headers
usermod -aG input,video,dialout stig # it's fine if this gives an error, modify accordingly
chsh -s /usr/bin/zsh 
# Change to ZSH
git clone https://gitlab.com/zstg/dotfiles ~/.dotfiles
syncer # linker not reqd, stuff is in PATH
~/.dotfiles/misc/setup-yay
paru -S hyprpaper eww-tray-wayland-git
~/dotfiles/misc/setup-rofi
~/.dotfiles/misc/setup-doom-emacs
~/.dotfiles/misc/setup-nvchad
systemctl --user enable --now xdg-desktop-portal-hyprland wireplumber pipewire-pulse pipewire gnome-keyring-daemon emacs 
```
## GTK settings
```bash
paru -S arc-gtk-theme gtk3 papirus-icon-theme bibata-cursor-theme-bin 
# Syncer would have copied the gtk settings as well
```
## Fonts
```bash
paru -S ttf-ms-fonts # For LibreOffice Flatpak
paru -S ttf-jetbrains-mono-nerd ttf-nerd-fonts-symbols-common ttf-nerd-fonts-symbols-mono
sudo fc-cache -fv
fc-cache -fv
```
## Flatpak
```bash
paru -S flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak --user remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```
## Polkit
```bash
sudo pacman -S polkit-gnome gnome-keyring libgnome-keyring
# /lib/polkit-gnome/polkit-gnome-authentication-agent-1 is part of Hyprland config
```
## Swap 
```bash 
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile 
sudo mkswap /swapfile 
echo '/swapfile    none    swap    sw    0    0' | sudo tee -a /etc/fstab 
# sudo reboot to use the swapfile 
```
## Hibernate settings
Edit `/etc/systemd/logind.conf:`

```bash
# InhibitDelayMaxSec=5
HandlePowerKey=ignore
HandlePowerKeyLongPress=poweroff
# DefaultTimeoutStopSec=5s
```
Edit `/etc/systemd/sleep.conf`:

```bash
HibernateDelaySec=3600
```
