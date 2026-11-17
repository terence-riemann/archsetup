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
# displays the contents of file.
cat ~/dotfiles/packages.txt

# ls - lists files and directories, -l - detailed listing (permissions, ownership, sizes, dates), -a - include hidden files and directories.
ls -la ~/.config

# find - search through directories, -maxdepth 2 - search no more than 2 directories deep, -type d - show directories only.
find ~/dotfiles/.config -maxdepth 2 -type d
```
One thing to note is that I'm using waybar whilst 43PR uses Quickshell.

Next we back up our current minimal, rudimentary config using a copy-paste command:

```bash
# cp - copy (and paste) command.
# -a - 'archive' mode; preserves permissions, ownerships, timestamps, symbolic links and directory structure.
cp -a ~/.config ~/.config-before-43pr
```
Now we want to check which packages for 43PR aren't currently installed:
```bash
# while read -r pkg; do - reads one line at a time and stores it in variable 'pkg'.
# pkg="${pkg%%#*}" - removes everything from the first '#' onwards in a line. 
# pkg="${pkg//[[:space:]]/}" - removes whitespace.
# -z "$pkg" - checks empty line, && continue - skips line in such case.
# pacman -Q "$pkg" >/dev/null 2>&1 - checks if pkg is installed and suppresses outputs and error.
# || echo "$pkg" - if pacman -Q "$pkg" fails (error).
# print pkg, done < ~/dotfiles/packages.txt - reads the package list for the lines.
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
Now we being the install. Change to the dotfiles directory and we store some directories as variables:
```bash
 cd ~/dotfiles

# REPO_DIR="$(pwd)" - 'pwd' prints the full path of the current working directory, '$(...)' executes the command and captures the output, 'REPO_DIR =' stores the output as a variable with such variable name. Similarly for the rest

REPO_DIR="$(pwd)"
SRC="$REPO_DIR/.config"
CONFIG_DIR="$HOME/.config"
BACKUP_ROOT="$HOME/.config-backups"
TIMESTAMP="$(date '+%Y-%m-%d_%H-%M-%S')"
BACKUP_DIR="$BACKUP_ROOT/$TIMESTAMP"

mkdir -p "$CONFIG_DIR"
```
Now we create a Bash array of all the configuration files we are going to use:
```bash
GENERATED_FILES=(
    "kitty/matugen.conf"
    "hypr/hyprlock-colors.conf"
    "gtk-3.0/colors.css"
    "gtk-4.0/colors.css"
    "gtk-3.0/settings.ini"
    "gtk-4.0/settings.ini"
    "rofi/colors.rasi"
    "quickshell/Theme.qml"
)
```
Now we create symlinks between the repo directory and config directory. This makes the config directory read the repo for what config files it should be running.
```bash
is_generated() {
    local rel="$1" g
    for g in "${GENERATED_FILES[@]}"; do
        [[ "$rel" == "$g" ]] && return 0
    done
    return 1
}

LINK_FAILED=0
LINKED_COUNT=0
BACKED_UP_COUNT=0

while IFS= read -r -d '' src; do
    rel="${src#"$SRC"/}"

    is_generated "$rel" && continue

    dest="$CONFIG_DIR/$rel"
    mkdir -p "$(dirname "$dest")"

    if [[ -L "$dest" ]]; then
        ln -sfn "$src" "$dest"

    elif [[ -e "$dest" ]]; then
        mkdir -p "$BACKUP_DIR/config/$(dirname "$rel")"
        mv "$dest" "$BACKUP_DIR/config/$rel"
        BACKED_UP_COUNT=$((BACKED_UP_COUNT + 1))
        ln -s "$src" "$dest"

    else
        ln -s "$src" "$dest"
    fi

    if [[ -L "$dest" && "$(readlink -f "$dest")" == "$(readlink -f "$src")" ]]; then
        LINKED_COUNT=$((LINKED_COUNT + 1))
    else
        LINK_FAILED=1
        echo "Failed to link $rel"
    fi

done < <(find "$SRC" \( -type f -o -type l \) -not -path '*/.git/*' -print0)
```
Afterwards, we check if there is a zshrc directory already in Home and if so, backs it up before creating a symlink to the repos zshrc setup:
```bash
if [[ -f "$SRC/.zshrc" ]]; then
    if [[ -e "$HOME/.zshrc" && ! -L "$HOME/.zshrc" ]]; then
        mkdir -p "$BACKUP_DIR/home"
        mv "$HOME/.zshrc" "$BACKUP_DIR/home/.zshrc"
        BACKED_UP_COUNT=$((BACKED_UP_COUNT + 1))
    fi

    ln -sfn "$SRC/.zshrc" "$HOME/.zshrc"
fi
```
ln -s makes it so it doesn't care whether the destination directory exists (in my case Home/.zshrc doesn't exist yet).
Now the generated files as defined before in the array are going to be the files of the repo that are copied and not symlinked. We loop through the array and create directories when necessary and copy the files of the repo into these created directories in my .config directory. These have more freedom to be edited and changed for personal taste.
```bash
for g in "${GENERATED_FILES[@]}"; do
    [[ -f "$SRC/$g" ]] || continue

    dest="$CONFIG_DIR/$g"

    if [[ ! -e "$dest" || -L "$dest" ]]; then
        [[ -L "$dest" ]] && rm -f "$dest"

        mkdir -p "$(dirname "$dest")"
        cp "$SRC/$g" "$dest"
    fi
done
```
We check if all of this symlinking was successful:
```bash
if [[ "$LINK_FAILED" -eq 1 ]]; then
    echo "Some symlinks failed. Check the errors."
else
    echo "All $LINKED_COUNT configuration files linked successfully."
fi

if [[ "$BACKED_UP_COUNT" -gt 0 ]]; then
    echo "Backups saved to: $BACKUP_DIR"
fi
```
Now we just copy a default wallpaper from the repo into a wallpaper directory:
```bash
WALLPAPER_DIR="$HOME/Pictures/Wallpapers"

mkdir -p "$HOME/Pictures" "$WALLPAPER_DIR"

if [[ -f "$REPO_DIR/wallhaven-k8jvm7.jpg" && ! -e "$WALLPAPER_DIR/wallhaven-k8jvm7.jpg" ]]; then
    cp "$REPO_DIR/wallhaven-k8jvm7.jpg" "$WALLPAPER_DIR/"
fi
```
We then enable pipewire:
```bash
systemctl --user enable --now pipewire.service
systemctl --user enable --now pipewire-pulse.service
systemctl --user enable --now wireplumber.service
```
Now we turn the scripts in the 


Things to run after reboot due to sudo giving me pain:
```bash
papirus-folders -C white

if pacman -Qi power-profiles-daemon >/dev/null 2>&1; then
    sudo systemctl enable --now power-profiles-daemon.service
fi

find "$SRC" -type f \( -name "*.sh" -o -name "*.py" \) -exec chmod +x {} \;

ls -l ~/.config/43pr/bin/theme.py

python3 ~/.config/43pr/bin/theme.py wallpaper \
    "$HOME/Pictures/Wallpapers/wallhaven-k8jvm7.jpg"

python3 ~/.config/43pr/bin/theme.py apply
```
These do checks:
```bash
find ~/.config -xtype l -print

ls -la ~/.config/hypr/

ls -l ~/.config/kitty/matugen.conf
ls -l ~/.config/hypr/hyprlock-colors.conf
ls -l ~/.config/rofi/colors.rasi
ls -l ~/.config/quickshell/Theme.qml

cat ~/.local/state/43pr/state.json
```


asd
sad
asd
asd
asd
