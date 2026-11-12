# Manual Installation of 43PR Hyperland Setup
We started with a relatively fresh installation of archlinux with a basic configuration of hyprland and waybar (https://www.youtube.com/watch?v=TS1ghG3c3xI, https://www.youtube.com/watch?v=PEgDssV0nW0&t=1898s&pp=0gcJCUkMAYcqIYzv by Josean Martinez).

First we copy the githup repo onto our home directory: https://github.com/43PR/dotfiles.

```bash
cd ~
git clone https://github.com/43PR/dotfiles.git
```
The second line uses the git package (which we have installed onto our system using pacman) to copy the whole repo into a new directory within the home directory which is named the same as the repo (in my case home/GeneTakavich/dotfiles).

We check the packages that the repo needs, check every current configuration that we have and check what configurations are featured in the repo:

```bash
# displays the contents of file
cat ~/dotfiles/packages.txt

# ls - lists files and directories, -l - detailed listing (permissions, ownership, sizes, dates), -a - include hidden files and directories
ls -la ~/.config

# find - search through directories, -maxdepth 2 - search no more than 2 directories deep, -type d - show directories only
find ~/dotfiles/.config -maxdepth 2 -type d
```
One thing to note is that I'm using waybar whilst 43PR uses Quickshell.

Next we back up our current minimal, rudimentary config using a copy-paste command:

```bash
# cp - copy (and paste) command, -a - 'archive' mode; preserves permissions, ownerships, timestamps, symbolic links and directory structure
cp -a ~/.config ~/.config-before-43pr
```
Now we want to check which packages for 43PR aren't currently installed:
```bash
# while read -r pkg; do - reads one line at a time and stores it in variable 'pkg', pkg="${pkg%%#*}" - removes everything from the first '#' onwards in a line, pkg="${pkg//[[:space:]]/}" - removes whitespace, -z "$pkg" - checks empty line, && continue - skips line in such case, pacman -Q "$pkg" >/dev/null 2>&1 - checks if pkg is installed and suppresses outputs and error, || echo "$pkg" - if pacman -Q "$pkg" fails (error), print pkg, done < ~/dotfiles/packages.txt - reads the package list for the lines
while read -r pkg; do
    pkg="${pkg%%#*}"
    pkg="${pkg//[[:space:]]/}"
    [[ -z "$pkg" ]] && continue
    pacman -Q "$pkg" >/dev/null 2>&1 || echo "$pkg"
done < ~/dotfiles/packages.txt | sort -u
```
Now that we know what packages we are missing, we install all of them.

I setup bluetooth since we have installed bluez (Bluetooth service and drivers), bluez-utils (Provides bluetoothctl, the command-line Bluetooth manager) and blueman (GUI Bluetooth manager). To enable the bluetooth service after installing such packages, we write this line:
```bash
sudo systemctl enable --now bluetooth.service
```


asd
sad
asd
asd
asd
