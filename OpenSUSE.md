# Installing OpenSUSE 15.3 on Dell Inspiron 3000 (3501)

## Introduction

Overall, OpenSUSE is a better option on this device. Hibernate works out of the box and there's no enabling it as an option in the power menus to do - it's all ready. 

So the following is just a list of post-install tasks, with a lot of overalp with the (X)Ubuntu list.

### Install Chrome

```
sudo zypper ar http://dl.google.com/linux/chrome/rpm/stable/x86_64 Google-Chrome

wget https://dl.google.com/linux/linux_signing_key.pub

sudo rpm --import linux_signing_key.pub

sudo zypper update

sudo zypper install google-chrome-stable
```

### Install Sublime Text

Not available from a third party repo, as with Ubuntu, and must be installed via snap. On the plus side, snap just seems to work with less fuss.

```
sudo zypper addrepo --refresh https://download.opensuse.org/repositories/system:/snappy/openSUSE_Leap_15.3 snappy

sudo zypper --gpg-auto-import-keys refresh

sudo zypper dup --from snappy

sudo zypper install snapd

sudo systemctl enable snapd

sudo systemctl start snapd

sudo snap install sublime-text --classic
```

### Update Python

OpenSUSE 15.3 comes with Python 3.6 which is not ideal. The process for getting a newer version seems to land you with....3.6. So more to be done here.

- Add the repo:

`zypper addrepo https://download.opensuse.org/repositories/devel:languages:python/openSUSE_Leap_15.3/devel:languages:python.repo`

- Get the number of the added repo (from the left hand column of the output):

`sudo zypper lr`

- Set the new repo to have a higher priority than the built-in OpenSUSE repos, refresh and update Python 3:

```
sudo zypper mr -p 50 PYTHON_REPO_NUMBER
sudo zypper ref
sudo zypper update python3
```

- Whichever Python 3 you end up with, install virtualenv with:

`pip3 install virtualenv virtualenvwrapper`

...and add the following to ~/.zshrc or ~/.basrc.

```
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
export WORKON_HOME=$HOME/.virtualenvs
export PROJECT_HOME=$HOME/projects
source $HOME/.local/bin/virtualenvwrapper.sh
```

### Install zsh/oh-my-zsh

- Install

```
zypper in zsh
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

- Remember to change zsh theme

### Install Ruby

sudo zypper install git gcc automake gdbm-devel libyaml-devel ncurses-devel readline-devel zlib-devel libopenssl-devel readline-devel

```
git clone https://github.com/rbenv/rbenv.git ~/.rbenv

cd ~/.rbenv && src/configure && make -C src

mkdir -p "$(rbenv root)"/plugins

wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```

- Add the following to ~.,zshrc

export PATH="$HOME/.rbenv/bin:$PATH"

eval "$(rbenv init -)"

### Install Node

- There may be a more recent version of NVM by the time you are reading this, in which case check the version number:

    ```
    wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

- Add the following to ~/.zshrc

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
