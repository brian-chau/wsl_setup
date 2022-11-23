# C++ on WSL2 with VS Code and CMake

This is an example of how to use:
* Install Ubuntu in WSL2
* Setting up Visual Studio Code for GCC compiler for C++ applications

## Setting up WSL2

0. Removing old installations of WSL2 distributions
    1. Open your "Start" menu and search for "Settings" and start the "Settings" app.
    2. Under "Apps", select "Apps & features"
    3. Under "App list", in the "Search apps" field, type the name of the distribution that you installed in the first step (e.g. ubuntu)
    4. Click the three vertical dots icon to the far right of the distribution, and click Uninstall.


1. Install WSL
    1. Start "Command Prompt"
    2. Run `wsl --set-default-version 2`

2. Install Ubuntu 22
    1. In the Start menu, open the Microsoft Store, and search for Ubuntu 22.
    2. Select the item called "Ubuntu 22.04.1 LTS" and click "Install."
    3. Click "Open" to launch the command prompt and start the installation process.
        * If you encounter an error that says `There has been an error`, log out and log back in, and try again.
        * If you encounter an error that says `Failed to attach disk 'C:\Users\....\ext4.vhdx' to WSL2: The system cannot find the file` then do the following:
            1. Launch a new command prompt window by pressing `Ctrl+R`, then typing `cmd`
            2. Type `wsl --list` to find the distribution that used to be installed.
            3. Type `wslconfig /u <name of distribution that used to be installed>` to unlink the distribution.
            4. Return to the Windows Store to try again.
    4. When prompted for the initial account credentials, enter your Windows account username as your username, so that your Linux username will match your Windows username.
    5. Enter a password, and confirm the password.

3. Start WSL
    1. Press Ctrl+R
    2. Type `wsl`
    3. Press Enter
    4. If you see a popup that says this:
        * ![Error message](wsl_error_msg.png)
        * Right click on the title bar of the WSL command window > Settings.
        * In the bottom left hand corner, click "Open JSON file"
        * Scroll down to "profiles" JSON item, and find the "defaults" sub-item in there.
            * This will open the "settings.json" file.
        * Under the "profiles > list" item, copy the settings for "wsl"
            * For example, this might look like
                ```
                "default": {},
                "list":
                [
                    ...
                    {
                        "guid": "{<some-ascii-hex-string-separated-by-underscores>}",
                        "hidden": false,
                        "name": "Ubuntu-22.04",
                        "source": "Windows.Terminal.Wsl"
                    },
                    ...
                ]
                ```
            * Copy it so it looks like
                ```
                "default": {
                    "guid": "{<some-ascii-hex-string-separated-by-underscores>}",
                    "hidden": false,
                    "name": "Ubuntu-22.04",
                    "source": "Windows.Terminal.Wsl"
                },
                "list":
                [
                    ...
                    {
                        "guid": "{<some-ascii-hex-string-separated-by-underscores>}",
                        "hidden": false,
                        "name": "Ubuntu-22.04",
                        "source": "Windows.Terminal.Wsl"
                    },
                    ...
                ]
                ```

4. Fix the command line path name.
    1. Edit the ~/.bashrc file.
        * At the top of the file, paste the following code snippet, where <current_directory> is the name of the
          mount point shown in the console window (e.g. "/mnt/c/Users/`username`")
            ```
            if [[ $PWD == /mnt/c/Users/$USER ]]; then
              cd /home/$USER/user
            else
              case $PWD/ in
                /mnt/c/Users/$USER/*)
                  without_prefix=${PWD#/mnt/c/Users/$USER/}
                  cd /home/$USER/user/user/$without_prefix
                  ;;
              esac
            fi
            ```
    2. Look for the line that says `if [ "$color_prompt" = yes ]; then`
        * In the following line, remove the part that specifies the username "\u" and the host name "\h"
        * Do the same for the "else" block.
        * For example, it might look like this:
            ```
            if [ "$color_prompt" = yes ]; then
                PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
            else
                PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
            fi
            ```
        * Change it to this:
            ```
            if [ "$color_prompt" = yes ]; then
                PS1='${debian_chroot:+($debian_chroot)}\[\033[01;34m\]\w\[\033[00m\]\$ '
            else
                PS1='${debian_chroot:+($debian_chroot)}\w\$ '
            fi
            ```
        * Also remove the `\u@\h` details from the title bar
            ```
            case "$TERM" in
            xterm*|rxvt*)
                PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
                ;;
            *)
                ;;
            esac
            ```
        * Change it to this:
            ```
            case "$TERM" in
            xterm*|rxvt*)
                PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\w\a\]$PS1"
                ;;
            *)
                ;;
            esac
            ```
    3. Save and exit.
    4. In the bash terminal, type the following commands:
        ```
        cd
        mkdir user
        ln -s /mnt/c/Users/$USER ~/user/user
        ```
    5. Restart `wsl`.

5. Update the system
    ```
    sudo apt update
    sudo apt upgrade
    sudo apt autoremove --purge
    ```

6. Add custom aliases to `~/.bash_aliases`
    ```
    alias cls="printf '\ec'; history -c"
    alias nanos="nano -c -ET4"
    alias ii="explorer.exe"
    ```

7. Install gcc-12 and g++-12.
    1. Restart WSL.
    2. Run `sudo apt update && sudo apt upgrade`
    3. Run `sudo apt install g++-12 gcc-12 build-essential`

8. Install `cmake` from source.
    1. Download `cmake-3.25.0.tar.gz` from this link: https://cmake.org/download/
    2. Unzip it with the command `tar -xzf cmake-3.25.0.tar.gz`.
    3. Run the following commands:
        ```
        sudo ./configure
        sudo make
        sudo make install
        ```

9. Add GitHub settings
    1. Restart WSL
    2. Run `mkdir repos`
    3. Connect to GitHub
        1. Sign in to GitHub
        2. Go to this path: https://github.com/settings/keys
        3. Delete any old SSH keys if any.
        4. Start creating SSH keys
            1. Run the following commands:
                ```
                ssh-keygen -t ed25519 -C "your_email@example.com"
                eval "$(ssh-agent -s)"
                ssh-add ~/.ssh/id_ed25519
                ```
            2. Run `cat ~/.ssh/id_ed25519.pub`
            3. Copy the result and go to this page: https://github.com/settings/ssh/new
            4. For "Title," put "Linux".
            5. For "Key," paste the copied key.
            6. Click "Add SSH Key"
            7. Run `ssh -T git@github.com`, then type "yes"

## Setting up VSCode
1. Right-click on the project folder, select `Open in Linux` to open the application in WSL2, then type `code .`
2. If prompted to answer "Do you trust the authors of the files in this folder?" check the box "Trust the authors of all files in the parent folder" and click "Yes, I trust the authors."
3. Press Ctrl+Shift+X to open the "Extensions" window.
4. Install the following extensions
    1. C/C++ Themes
    2. C/C++ Extension Pack
    3. Makefile Tools
    4. Remote - WSL
    5. Better C++ Syntax
    6. Clang Format by xaver
5. Close VSCode
6. Install Clang Format with `sudo apt install clang-format`
7. Type `code .` in the terminal to restart it.
8. Press Ctrl+Shift+X again to open the "Extensions" window.
9. If any of the above extensions say "Install in WSL: Ubuntu-22.04", then click that button.
10. Configure editor settings
    1. Click the gear icon in the lower left corner.
    2. Click Settings.
    3. In the search bar, type "minimap" and uncheck "Editor > Minimap: Enabled" where the checkbox says "Controls whether the minimap is shown"
    4. In the search bar, type "Trim Trailing Whitespace" and check the box for "Files: Trim Trailing Whitespace"
    5. In the search bar, type "Detect Indentation" and uncheck the box for "Editor: Detect Indentation"
    6. In the search bar, type "Format On Save" and check the box for "Editor: Format On Save"
    7. In the search bar, type `C_Cpp.clang_format_fallbackStyle`
        1. In the field that appears, change `Visual Studio` to `{ BasedOnStyle: Google, IndentWidth: 4 }`
    8. In the search bar, type "Clang-format: Executable".
        1. In the WSL terminal, type `which clang-format`
        2. Copy that path and paste it into the field.
    9. In the search bar, type "Default Formatter" and select from the dropdown menu `C/C++`.
    10. In the search bar, type `C_Cpp.default.cppStandard` and change the value to `gnu++20`

11. Set the key bindings to build and clean the solution.
    1. Press Ctrl+K Ctrl+S
    2. In the keybindings search box, type "makefile: build clean the target ALL"
        1. Double-click the keybinding and replace it with Ctrl+Shift+B.
            * There is already a keybinding for that in VSCode, but you can remove the other keybinding to replace it with this.
        2. Press Enter to save the keybinding.
    3. In the keybindings search box, type "Makefile: Build clean the current target"
        1. Double-click the keybinding and replace it with Ctrl+Shift+Z.
