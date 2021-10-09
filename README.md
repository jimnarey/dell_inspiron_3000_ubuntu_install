# Installing Xubuntu/Ubuntu 20 on Dell Inspiron 3000 (3501)

## Introduction

Many of the Dell Inspiron series, including the 3501 on which this was tested, are [certified as working with Ubuntu](https://ubuntu.com/certified).
When doing a fresh install after wiping Windows, Ubuntu 20 pretty much works out of the box but there are a few additional steps
involved to get everything working nicely.

## You will need...

- An Xubuntu (or other Ubuntu) USB stick (download the image and use dd/Etcher/Rufus/etc to put it on USB).

- A USB to Ethernet adapter (the required wifi driver isn't on the Xubuntu 20.04 ISO image).

## Installation

*This wipes Windows and anything else from the disk. Completely!*

- Disable secure boot in the bios

- Insert a bootable Xubuntu Live USB 

- Select 'Try Xubuntu'. Open Gparted from the menu, select the internal SSD, and 'Create Parition Table' from the Devices
  menu. Choose 'GPT' from the drop-down.
  
- Start the installation program. Either create a swap partition larger than the device's total RAM (I used 18GB for
  a 16GB machine) or do this later in Gparted. Using a swap partition reduces the chances of running into problems with
  hibernate later. Otherwise, no special settings are required.
  
## Post Installation

### General

- Open Window Manager settings and choose one of the HDPI or XHDPI themes. Otherwise finding the right spot with the
  mouse pointer to do things like resize windows is a pain.
  
- With the USB/Ethernet adapter plugged in, run:

    `sudo apt update`

    `sudo apt upgrade`

  (You can optionally have the adapter plugged in during install and choose, when prompted, to have the installer
  download updates but this increases the installation time and often seems to cause the installtion of GRUB to fail).

- Wifi should now be working and the USB/Ethernet adapter no longer required.

- Install the [Ubuntu OEM kernel](https://wiki.ubuntu.com/Kernel/OEMKernel#How_to_install_it.3F), which has some 
  changes to support OEMs such as Dell, Lenovo etc. It's debatable how necessary this is but better safe than sorry. In 
  that spirit, use the oldest of the available OEM kernels:
  
    `sudo apt install linux-oem-20.04b`

  (If you're feeling brave you could try `linux-oem-20.04c` or `linux-oem-20.04d`).

- After rebooting using the newly installed kernel I also removed the other installed kernels, with:

    `sudo apt purge linux-image-XXXXXXX-XXXXXX`

  (You can see all the install kernels with `dpkg --list | grep linux-image`)

- One of the installed kernels would not shift (even though apt reported no errors) due to its counterpart linux-modules
  package being dependent on it. The linux-modules package wouldn't shift either because of the kernel's dependency on 
  it... To get round this:
  
    `sudo dpkg -P linux-image-XXXXXXX-XXXXXX linux-modules-XXXXXXX-XXXXXX`

### Enable hibernate

The one major thing which doesn't work out of the box. To get it working:

- Some more packages are needed:

    `sudo apt install pm-utils hibernate`

- Get the UUID of the swap partition by running the following and looking for the relevant line:

    `sudo blkid`

- Open the grub config file:

    `sudo nano /etc/default/grub`

- Edit the line `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"` to:

    `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash resume=UUID=<YOUR_COPIED_UUID>"`

- Save the file and run:

    `sudo update-grub`

- Hibernate can now be initiated with `sudo systemctl hibernate` or `sudo hibernate` but does it appear in the Xubuntu
  power menu? Does it Hell. To fix this:
  
    `sudo nano /etc/polkit-1/localauthority/50-local.d/com.ubuntu.enable-hibernate.pkla`

- Add the following to the (in my case) new/blank file:

```
[Re-enable hibernate by default in upower]
Identity=unix-user:*
Action=org.freedesktop.upower.hibernate
ResultActive=yes

[Re-enable hibernate by default in logind]
Identity=unix-user:*
Action=org.freedesktop.login1.hibernate;org.freedesktop.login1.handle-hibernate-key;org.freedesktop.login1;org.freedesktop.login1.hibernate-multiple-sessions;org.freedesktop.login1.hibernate-ignore-inhibit
ResultActive=yes
```

- Reboot.

### Power management settings

- Open Power Management settings from the Xubuntu menu. You can now select what you want to happen when, for example,
  the laptop lid is closed and I suggest something more sane than the default 'Blank Screen' option.
  
- Either via the Power Management settings or straight from the main menu open screensaver options and turn it off
  completely. It causes the desktop/menu/everything to be completely unresponsive after suspend (though oddly, not
  hibernate). You can always set the screen to go blank after a set time in the Power Management Settings.

## Setting up a basic development environment

This is entirely optional but is why I wanted a laptop which plays nicely with Linux.

### Install packages:

    `sudo apt install build-essential git gparted python3-pip zsh openssh-server wget curl vim terminator fonts-powerline tmux code`

### Install Sublime Text:


    ```
    wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
    sudo apt-get install apt-transport-https
    echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
    apt-get update
    apt-get install sublime-text
    ```

### Install zsh/oh-my-zsh:

- Running the following will give you the option to change your shell to zsh. Choose yes.

    ```
    sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
    ```

- Change your theme, in my case changing: `ZSH_THEME="robbyrussell"`

    `ZSH_THEME="bullet-train"` or

    `ZSH_THEME="bira"`

### Install Ruby

- Run the following:


    ```
    apt-get install -y libssl-dev libreadline-dev zlib1g-dev ruby
    wget -q https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-installer -O- | bash
    ```

- Open `~/.zshrc` and add `$HOME/.rbenv/bin` to your path. You may have an existing line beginning `export PATH=` which
can be adapted or adding the following line will work:
  
    `export PATH=$HOME/.rbenv/bin:$PATH`

- Add the following line somewhere in the same file, after the line setting the PATH variable:

    `eval "$(rbenv init -)"`

- Reload the shell:

    `exec $SHELL`

- Install ruby-build with the following lines:


    ```
    mkdir -p "$(rbenv root)"/plugins
    git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
    ```

- You can now install your preferred verison of Ruby with something like:

    `rbenv install 2.7.2`

### Install Node

- There may be a more recent version of NVM by the time you are reading this, in which case check the version number:


    ```
    wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
    ```

- NVM should give a set of lines to add to `~/.zshrc` but it's usually these:


    ```
    export NVM_DIR="$HOME/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
    [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
    ```

### Virtualenv

    `pip3 install virtualenv virtualenvwrapper`

- Add the following lines to `~/.zshrc`. Adapt them as required to suit where you keep your projects and where
  you want to keep your virtualenvs.


    ```
    export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
    export WORKON_HOME=$HOME/.virtualenvs
    export PROJECT_HOME=$HOME/projects
    source $HOME/.local/bin/virtualenvwrapper.sh
    ```

### Go

- This doesn't require much other than the download linked from [here](https://golang.org/doc/install).

- Setting up your GOPATH properly makes life easier. See [this](https://hackersandslackers.com/create-your-first-golang-app/).

## References

- [Enable hibernate](https://askubuntu.com/questions/1240123/how-to-enable-the-hibernate-option-in-ubuntu-20-04). You'll
  see some debate here and elsewhere over whether a swap partion (rather than a swapfile) are needed but there were
  more than enough reports of problems with a swapfile for me to opt for a partition.
  
- [Enable hibernate options in power menu](https://ubuntuforums.org/showthread.php?t=2432961)

- [Install OEM Kernel](https://wiki.ubuntu.com/Kernel/OEMKernel#How_to_install_it.3F)

- Setting a default kernel in GRUB: [this](https://askubuntu.com/questions/970969/how-does-one-use-grub-default-to-select-a-default-os-for-boot)
  and [this](https://askubuntu.com/questions/1308901/setting-older-kernel-version-as-default). I could not get
  various combinations of `GRUB_SAVEDEFAULT=true` and `GRUB_DEFAULT=saved` to work and didn't want to hardcode a
  specific kernel option so opted for removing the other kernels, as above.

  
