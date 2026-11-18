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
cat ~/dotfiles/packages.txt # displays the contents of file.

ls -la ~/.config # ls - lists files and directories, -l - detailed listing (permissions, ownership, sizes, dates), -a - include hidden files and directories.

find ~/dotfiles/.config -maxdepth 2 -type d # find - search through directories, -maxdepth 2 - search no more than 2 directories deep, -type d - show directories only.
```
One thing to note is that I'm using waybar whilst 43PR uses Quickshell.

Next we back up our current minimal, rudimentary config using a copy-paste command:

```bash
# 
cp -a ~/.config ~/.config-before-43pr # cp - copy (and paste) command, -a - 'archive' mode; preserves permissions, ownerships, timestamps, symbolic links and directory structure
```
Now we want to check which packages for 43PR aren't currently installed:
```bash
while read -r pkg; do # reads one line at a time and stores it in variable 'pkg'.
    pkg="${pkg%%#*}" # removes everything from the first '#' onwards in a line.
    pkg="${pkg//[[:space:]]/}" # removes whitespace.
    [[ -z "$pkg" ]] && continue # checks empty line, skips line in such case.
    pacman -Q "$pkg" >/dev/null 2>&1 || echo "$pkg" # pacman -Q "$pkg" >/dev/null 2>&1 - checks if pkg is installed and suppresses outputs and error, || echo "$pkg" - if pacman -Q "$pkg" fails (error).
print pkg, done < ~/dotfiles/packages.txt | sort -u # reads the package list for the lines.
```
Now that we know what packages we are missing, we install all of them.

I setup bluetooth since we have installed bluez (Bluetooth service and drivers), bluez-utils (Provides bluetoothctl, the command-line Bluetooth manager) and blueman (GUI Bluetooth manager). To enable the bluetooth service after installing such packages, we write this line:
```bash
sudo systemctl enable --now bluetooth.service
```
Now we being the install. Change to the dotfiles directory and we store some directories as variables:
```bash
 cd ~/dotfiles

REPO_DIR="$(pwd)" # REPO_DIR="$(pwd)" - 'pwd' prints the full path of the current working directory, '$(...)' executes the command and captures the output, 'REPO_DIR =' stores the output as a variable with such variable name. Similarly for the rest.

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
Afterwards, we check if there is a zshrc directory already in Home and if so, back it up before creating a symlink to the repos zshrc setup:
```bash
if [[ -f "$SRC/.zshrc" ]]; then # checks if the repo .zshrc exists on the system.
    if [[ -e "$HOME/.zshrc" && ! -L "$HOME/.zshrc" ]]; then # checks if .zshrc exists in the Home directory and checks if it is not already a symbolic link (-L)
        mkdir -p "$BACKUP_DIR/home" # create a backup directory
        mv "$HOME/.zshrc" "$BACKUP_DIR/home/.zshrc" # moves the existing .zshrc to the backup directory 
        BACKED_UP_COUNT=$((BACKED_UP_COUNT + 1)) # increments the back up count
    fi

    ln -sfn "$SRC/.zshrc" "$HOME/.zshrc" # creates a symbolic link (s), forces replacement of an existing destination (f) and treats an existing symlink to a directory as a link rather than following it 
fi
```
ln -s makes it so it doesn't care whether the destination directory exists (in my case Home/.zshrc doesn't exist yet).

Now the generated files as defined before in the array are going to be the files of the repo that are copied and not symlinked. We loop through the array and create directories when necessary and copy the files of the repo into these created directories in my .config directory. These have more freedom to be edited and changed for personal taste.
```bash
for g in "${GENERATED_FILES[@]}"; do # iterate over the entries in the array, preserving filenames with spaces in them.
    [[ -f "$SRC/$g" ]] || continue # check whether such entry exists as a path directory in the repo.

    dest="$CONFIG_DIR/$g" # sets a variable dest to be that particular path directory, but not from the repo but from the home config directory.

    if [[ ! -e "$dest" || -L "$dest" ]]; then # checks if destination does not exist or if it is a symlink.
        [[ -L "$dest" ]] && rm -f "$dest" # checks if destination is a symlink and if so, then removes the destination.

        mkdir -p "$(dirname "$dest")" # creates a new destination directory
        cp "$SRC/$g" "$dest" $ copies the repo files of the repo directory corresponding to the array entry into the destination.
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
WALLPAPER_DIR="$HOME/Pictures/Wallpapers" # Sets variable as path of Wallpaper directory

mkdir -p "$HOME/Pictures" "$WALLPAPER_DIR" # Creates a Wallpapers and Pictures directory if they don't exist yet.

if [[ -f "$REPO_DIR/wallhaven-k8jvm7.jpg" && ! -e "$WALLPAPER_DIR/wallhaven-k8jvm7.jpg" ]]; then # checks if wallpaper exists in the repo but not in the wallpaper directory
    cp "$REPO_DIR/wallhaven-k8jvm7.jpg" "$WALLPAPER_DIR/" # copies repo wallpaper to home wallpaper directory
fi
```
We then enable pipewire:
```bash
systemctl --user enable --now pipewire.service
systemctl --user enable --now pipewire-pulse.service
systemctl --user enable --now wireplumber.service
```
We make the folders white:
```bash
papirus-folders -C white
```
papirus-folders needs to be installed first.

Then we enable the power-profiles-daemon service:
```bash
if pacman -Qi power-profiles-daemon >/dev/null 2>&1; then
    sudo systemctl enable --now power-profiles-daemon.service
fi
```
Now we turn the files in the repo into executable files now:
```bash
find "$SRC" -type f \( -name "*.sh" -o -name "*.py" \) -exec chmod +x {} \; # recursively searches through the repo directory for regular files (not symlinks or further subdirectories), giving executable permission to .sh or .py files 
```
Now that they are executable files, we can do the heavy lifting of using the python files to use wallpapers and apply themes. Here we apply a default wallpaper (which we already lifted from the repo into our wallpaper directory):
```bash
python3 ~/.config/43pr/bin/theme.py wallpaper \ # uses python to run the theme script in the repo and use the wallpaper argument 
    "$HOME/Pictures/Wallpapers/wallhaven-k8jvm7.jpg" # the wallpaper path is the second argument for the theme script after the wallpaper argument
```
Finally we have to generate and apply theme and generate the files that the configurations and apps depend on:
```bash
python3 ~/.config/43pr/bin/theme.py apply
```
To change to zsh as the default shell, we need to list it under /etc/shells to be able make the change for default shell to zsh:
```bash
which zsh # finds the location of zsh (executable) in the home directory

sudo nvim /etc/shells # we will edit this file and /usr/sbin/zsh or whatever 'which zsh' outputed to the list

chsh -s /usr/sbin/zsh # changes the login shell to a new shell

echo $SHELL # to check your default shell
```
We can do some checks to see if anything went wrong:
```bash
find ~/.config -xtype l -print # finds and prints broken symlinks

ls -la ~/.config/hypr/

ls -l ~/.config/kitty/matugen.conf
ls -l ~/.config/hypr/hyprlock-colors.conf
ls -l ~/.config/rofi/colors.rasi
ls -l ~/.config/quickshell/Theme.qml

cat ~/.local/state/43pr/state.json # displays the content of the repo theme file in the terminal
```

