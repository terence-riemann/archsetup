# Manual Installation
First we copy the githup repo onto our home directory: https://github.com/43PR/dotfiles.

```bash
cd ~
git clone https://github.com/43PR/dotfiles.git
```
The second line uses the git package (which we have installed onto our system using pacman) to copy the whole repo into a new directory within the home directory which is named the same as the repo (in this case home/GeneTakavich/dotfiles).

We check the packages that the repo needs, check every current configuration that we have and check what configurations are featured in the repo:

```bash
-- displays the contents of file
cat ~/dotfiles/packages.txt

-- ls - lists files and directories, -l - detailed listing (permissions, ownership, sizes, dates), -a - include hidden files and directories
ls -la ~/.config

-- find - search through directories, -maxdepth 2 - search no more than 2 directories deep, -type d - show directories only
find ~/dotfiles/.config -maxdepth 2 -type d
```
Next we back up 
asd
sad
asd
asd
asd
