---
title: My first steps after installing Xubuntu
layout: post
categories:
    - blog
    - tech
---

Every time a new Ubuntu version is released you see blog posts popping up about
the first things to do after installing it and every time I read one of those
it's mostly a waste of time to me. I guess I just have different requirements.
So here is the list of things I do after installing Ubuntu. Xubuntu 16.04 that
is in this case. First a few things that will be relevant for common users too,
before getting to the geeky stuff.


## Setup Supported Languages

The dialog pops up into your face after the first boot, so why not do it right
away. Otherwise you can find it under `Language Support`. You don't need to
change the languages for menus and windows assuming you made the right choice
during installation. However, you should install all the languages you could
possibly care about as they are important for spell checking in Libre Office
for example.


## Setup Touchpad Properly

Under `Mouse and Touchpad` you should either disable the touchpad altogether (I
prefer using the pointing stick) or tick the `Disable touchpad while typing`
box while reducing the delay (unless you have really slow fingers). This is to
avoid having your mouse cursor jumping around or changin focus while you type.


## Setup Window Panel

It can be configured by doing a right click and then choosing `Panel` -> `Panel
Preferences`. I prefer to have the panel at the bottom, to get it there you go
to the `Display` tab, untick `lock`, drag the panel to the bottom of the screen
and tick `lock` again.
There are several things to do under the `Items` tab:

 - First configure the window buttons. I disable `Switch windows using the
   mouse wheel`, set `Sorting order` to `None` (the default order is weird and
   like this I can drag and drop) and change the `Middle click action` to `Close
   Window` (so it's the same thing as in Firefox)
 - Add CPU Graph, configure it to not have a frame and make the background
   color the same as the panel using the colour picker.
 - Change the format of the clock to `Week %V, %A %d %B, %H:%M:%S`

## Setup External Hardware

I have got a printer and a NAS both configured with static IPs.

### NAS

I mount the nfs shares of the NAS using autofs:

```bash
sudo apt-get install autofs
sudo vi /etc/auto.master
```

Add the following line (`--ghost` creates empty directories for the shares):

```
/nas /etc/auto.nas --ghost
```

Then create /etc/auto.nas with the following content:

```
Documents    192.168.1.2:/data/Documents
Music        192.168.1.2:/data/Music
Pictures     192.168.1.2:/data/Pictures
Transmission 192.168.1.2:/data/Transmission
Videos       192.168.1.2:/data/Videos
```

And restart (reload does not bring up the ghost directories) the autofs
service:

```bash
sudo /etc/init.d/autofs restart
```

Unfortunately there is a problem with the wireless that prevents it from
locating local IPs for a few minutes after starting it (and thus access to the
NAS fails) and I could not figure out why. However, disabling the wireless n
protocol serves as a workaround:

```bash
sudo tee /etc/modprobe.d/iwlwifi-opt.conf <<< "options iwlwifi 11n_disable=1"
# restart wireless
sudo modprobe --remove iwlwifi
sudo modprobe iwlwifi
```

### Printer

Every time I tried to add a network printer through the dialogues, I was left
with now printer installed and a broken package system which was quite hard to
fix. Most likely culprit: `gutenprint` Luckily `Brother` provides a `Driver
Install Tool` on their homepage. So I download it under http://www.brother.com
-> Software Downloads -> Printers and do the following:

```bash
gunzip linux-brprinter-installer-2.0.0-1.gz
sudo bash linux-brprinter-installer-2.0.0-1
Input model name -> DCP-L2560DW
```

After confirming a few things and typing in the IP (192.168.1.3) printing and
scanning (using `Simple Scan`) work like a charm. But for the best quality, you
still need to change resolution to `HQ1200` under `Printer Properties` ->
`Printer Options`

### Bluetooth

I listen to music over bluetooth and discovering and pairing of my receiver
works without problems. However, connecting to it as audio sink generally fails
with `Connection Failed: blueman.bluez.errors.DBusFailedError: Protocol not
available...` Installing `pulseaudio-module-bluetooth` solves this problem.

## Setup Keyboard Shortcuts

 - Set preferred terminal emulator to `X Term` in `Preferred Applications`
 - Add keyboard shortcuts for `xbacklight -dec 1` and `xbacklight -inc 1` (the
   default hardware keys do not allow fine enough steps for backlight) and
   install xbacklight `sudo apt-get install xbacklight`
 - Change application for keyboard shortcut `Print Screen` to
   `xfce4-screenshooter --fullscreen --save /home/sebastian/Desktop`
 - Change keyboard shortcut for xfce4-popup-whiskermenu to `Super+X` (can't use
   only `Super`, as shortcuts using `Super+something` will not work if you do)
 - Change key for drag-and-drop of windows to `Super` in `Window Manager Tweaks`
 - Change `Show Desktop` shortcut to `Super+D` in `Window Manager`

## Install Useful Apps

XnView and Skype have to be downloaded and installed manually all other are
available through the package manager:

```bash
sudo apt-get install \
    entr \
    gmusicbrowser \
    gnome-font-viewer \
    gparted \
    inkscape \
    pdftk \
    pinta \
    puddletag \
    silversearcher-ag \
    texlive
    tree \
    usb-creator-gtk \
    vlc \

```

## Uninstall Apps I Don't Use

```bash
sudo apt-get purge \
    gnome-mines \
    gnome-sudoku \
    parole `# I prefer vlc and gmusicbrowser` \
    pidgin \
    ristretto `# I prefer xnview` \

```

## Enable support for watching HBO

Unfortunately HBO does not use HTML5 and does use DRM, so all you see is a
black screen when you try to watch Game of Thrones. However, there is a great
[tutorial](https://ubuntuforums.org/showthread.php?t=2274467) of how to get
around this problem. As pipelight kept on stealing focus on several websites, I
found it necessary to enable it only for watching something and to disable it
when done:

```bash
sudo pipelight-plugin --create-mozilla-plugins
sudo pipelight-plugin --remove-mozilla-plugins
```

## Enable support for playing DVDs

Playing DVDs won't work by default either and apparently there might be legal
issues when doing something about it. So here is just a [link with more
information](https://help.ubuntu.com/community/RestrictedFormats/PlayingDVDs)

## Setup Terminal Environment

### Switch To ZSH

```bash
sudo apt-get install zsh
chsh -s /usr/bin/zsh
```

Logout and login again

### Install Dotfiles

My dotfiles are on Github along with all my other public projects, so lets get
all of them at the same time. First access has to be sorted, so either existing
ssh keys (~/.ssh folder) need to be recovered or new ones generated and
registered with Github
(https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)
Then the repositories can be cloned:

```bash
# a few things are required
sudo apt-get install curl jq git subversion
mkdir ~/Clones
cd ~/Clones
curl --silent https://api.github.com/users/sblask/repos \
    | jq --raw-output ".[] | .ssh_url" \
    | xargs -L 1 git clone
cd ~/Clones/dotfiles
# retrieves a few things that are not part of my dotfiles repository
./clone.sh
./install.py # a few folders have to be created manually, but install will tell you about them
```

Side note: I also have private repositories on Bibucket, they can be cloned
like this:

```bash
echo "Bitbucket password:" \
    && read -s password \
    && curl --silent --user "sblask:${password}" https://api.bitbucket.org/2.0/repositories/sblask \
    | jq --raw-output ".values | .[] | .links.clone | .[] | select(.name==\"ssh\") | .href" \
    | xargs -L 1 git clone
```

There is a lot of stuff in the dotfiles, so a few more programs need to be
installed that are either configured in the dotfiles or are required in some
other ways:

```bash
sudo apt-get install \
    baobab `# to analyze disk usage from Thunar context menu` \
    compton \
    devilspie2 \
    maximus \
    meld `# to compare files and directories from Thunar context menu` \
    puddletag `# to edit mp3 tags from Thunar context menu` \
    redshift \
    tmux `# use `CTRL-B` + `I` to install the plugins` \
    vim-gtk `# run BundleInstall the first time you run it to install all the plugins` \
    xclip \

```

## Setup Programming Language Environments

The following environments are mostly for programming, where it's important to
avoid version conflicts between dependencies of different projects. They also
help to keep your system clean. I think it's preferable to only install stuff
system wide if it comes with the system package manager. But there is for
example [Jekyll][jekyll] which I use for this blog that comes as a Ruby gem.
Using [rvm][rvm] allows me to do `gem install jekyll` without installing it
system wide.

### Javascript/Node

Really the easiest of the lot, all you need is the following which will also
change you shell configuration file.

```bash
curl -L http://git.io/n-install | bash
```

### Ruby

```bash
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
curl -sSL https://get.rvm.io | bash -s stable
rvm list known
rvm install 2.3.1 # the latest version from `list known`
```

Rvm has to be in your [PATH][rvm_path] and be [loaded][rvm_zsh] by your shell.

[jekyll]:   http://jekyllrb.com/
[rvm]:      https://rvm.io/
[rvm_path]: https://github.com/sblask/dotfiles/blob/40cccd3fc918059e5da160151af2bde7ce73b515/zshrc.dotfile#L227
[rvm_zsh]:  https://github.com/sblask/dotfiles/blob/40cccd3fc918059e5da160151af2bde7ce73b515/zshrc.dotfile#L406

### Python

There are certainly other ways to do this, but when the goal is to keep the
system clean, I can even get away without installing virtualenv system wide:

```bash
cd /tmp
# get the current virtualenv package
curl https://pypi.python.org/pypi/virtualenv \
    |  pup 'a[href]:contains("tar.gz") attr{href}' | xargs -L 1 wget

# install virtualenv and virtualenvwrapper
tar --extract --strip-components 1 --file *.tar.gz
./virtualenv.py ~/opt/virtualenv
~/opt/virtualenv/bin/pip install virtualenvwrapper

```

Now virtualenvwrapper can be loaded from your [shell
configuration][virtualenvwrapper_zsh] which requires your
[PATH][virtualenv_path] to contain the path to the virtualenv in `~/opt`.
To make it even handier and to be able to easily handle the difference between
Python 2 and 3, I've got [few aliases][virtualenvwrapper_aliases].

[virtualenvwrapper_aliases]: https://github.com/sblask/dotfiles/blob/48d47a96aa174b92b808f5b3faff94e05b8bb5f0/zshrc.dotfile#L293
[virtualenvwrapper_zsh]:     https://github.com/sblask/dotfiles/blob/40cccd3fc918059e5da160151af2bde7ce73b515/zshrc.dotfile#L233
[virtualenv_path]:           https://github.com/sblask/dotfiles/blob/40cccd3fc918059e5da160151af2bde7ce73b515/zshrc.dotfile#L227
