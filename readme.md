# dotfiles

Here are some settings for the various tools that I
use.


## Quickly Get Set Up with These Dotfiles

I'm going to try my best to provide beginning to end installation instructions
for Ubuntu (native or WSL) along with macOS. Fortunately setting up
most of these tools will be the same on any OS since they're not OS specific.

### OS / distro specific installation steps

```sh
sudo apt-get update && sudo apt-get install -y \
  curl \
  git \
  gnupg \
  htop \
  python3-pip \
  shellcheck \
  tmux \
  zsh \
  unzip
```

```sh
chsh -s $(which zsh)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

```

#### Install these dotfiles and various tools on your system
```sh

# Clone down this dotfiles repo to your home directory. Feel free to place
# this anywhere you want, but remember where you've cloned things to.
git clone https://github.com/mr4torr/dotfiles ~/dotfiles

# Create symlinks to various dotfiles. If you didn't clone it to ~/dotfiles
# then adjust the ln -s symlink source (left side) to where you cloned it.
#
# NOTE: The last one is WSL 1 / 2 specific. Don't do it on native Linux / macOS.
mkdir -p ~/.local/bin && mkdir -p ~/.vim/spell \
  && ln -s ~/dotfiles/.bashrc ~/.bashrc \
  && ln -s ~/dotfiles/.profile ~/.profile \
  && ln -s ~/dotfiles/.gitconfig ~/.gitconfig \
  && ln -s ~/dotfiles/.aliases ~/.aliases \
  && ln -s ~/dotfiles/.tmux.conf ~/.tmux.conf \
  && ln -s ~/dotfiles/.vimrc ~/.vimrc \
  && ln -sf ~/dotfiles/.zshrc ~/.zshrc \
  && ln -s ~/dotfiles/.vim/spell/en.utf-8.add ~/.vim/spell/en.utf-8.add \
  && ln -s ~/dotfiles/.local/bin/set-theme ~/.local/bin/set-theme \
  && sudo ln -s ~/dotfiles/etc/wsl.conf /etc/wsl.conf

# Create your own personal ~/.gitconfig.user file. After copying the file,
# you should edit it to have your name and email address so git can use it.
cp ~/dotfiles/.gitconfig.user ~/.gitconfig.user

# Install Plug (Vim plugin manager).
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
  https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

# Install TPM (Tmux plugin manager).
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# Install FZF (fuzzy finder on the terminal and used by a Vim plugin).
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf && ~/.fzf/install

# Install ASDF (version manager which I use for non-Dockerized apps).
git clone https://github.com/asdf-vm/asdf.git ~/.asdf

# Install Node through ASDF. Even if you don't use Node / Webpack / etc., there
# is one Vim plugin (Markdown Preview) that requires Node and Yarn.
asdf plugin-add nodejs https://github.com/asdf-vm/asdf-nodejs.git
bash ~/.asdf/plugins/nodejs/bin/import-release-team-keyring
asdf install nodejs 14.10.1
asdf global nodejs 14.10.1

# Install Yarn.
npm i -g yarn
npm i -g typeorm

# Install system dependencies for Ruby on Debian / Ubuntu.
#
# Not using Debian or Ubuntu? Here's alternatives for macOS and other Linux distros:
#   https://github.com/rbenv/ruby-build/wiki#suggested-build-environment
sudo apt-get install -y autoconf bison build-essential libssl-dev libyaml-dev \
  libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm6 libgdbm-dev libdb-dev

```

#### Install plugins for Vim and tmux

```sh
# Open Vim and install the configured plugins. You would type in the
# :PlugInstall command from within Vim and then hit enter to issue the command.
vim .
:PlugInstall
# OR
vim +PlugInstall +qall

# Start a tmux session and install the configured plugins. You would type in
# ` followed by I (capital i) to issue the command.
tmux
`I
```

#### Optionally confirm that a few things work after closing and re-opening your terminal

```sh
# Sanity check to see if you can run some of the tools we installed.
node --version
npm --version
yarn --version

# Check to make sure git is configured with your name, email and custom settings.
git config --list

# If you're using Docker Desktop with WSL 2, these should be accessible too.
docker info
docker-compose --version
```


## FAQ

### How to add custom themes to the set-theme script?

Prefer a video? [Here's a video](https://www.youtube.com/watch?v=h509rn2xIyU&t=191s)
that demonstrates performing the steps below.

After installing these dotfiles you'll have a `~/.local/bin/set-theme` script.
It's a [zero dependency Python 3
script](https://github.com/mr4torr/dotfiles/blob/master/.local/bin/set-theme).

1. Open the above file
2. Check out the `THEMES` dictionary near the top of the file
3. Copy one of the existing themes' dictionary items, such as `gruvbox` or `one`
4. Rename the dictionary's key to whatever your new theme's colorscheme name is in Vim
5. Adjust all of the colors and additional values in your new dictionary item as you see fit
6. Run `set-theme cooltheme`, replacing `cooltheme` with whatever name you used in step 4

Your terminal and tmux colors will update automatically, but if you have Vim
already open you'll need to manually run this command from within Vim to reload
your config `:so $MYVIMRC`.

If you added a theme with good contrast ratios please open a pull request to
get it added to the script.

### How to use a different terminal in the set-theme script?

I'm using the Microsoft Terminal but if you're using something else then your
terminal's colors won't get updated by this script because the script looks for
strings that are in MS terminal's config, but it's not painful to change.

*By the way, if you're using the Microsoft Terminal Preview edition you'll
still need to do step 1 below because the path of your MS terminal config file
will be different than the non-preview edition.*

You'll want to adjust the `set-theme` script by doing this:

1. Change the `TERMINAL_CONFIG` variable to reference your terminal config's path
2. Change the `terminal` attributes in the `THEMES` dictionary to use your terminal's config option names
3. Change the regex in the `change_terminal_theme` function based on your terminal's config option formatting rules
4. Optionally install Gruvbox, One or any other themes (the MS Terminal config in this repo includes them)

### How to fix Vim taking a long time to open when inside of WSL?

It primarily comes down to either VcXsrv not running or a firewall tool
blocking access to VcXsrv and it takes a bit of time for the connection to time
out.

You can verify this by starting Vim with `vim -X` instead of `vim`. This
will prevent Vim from connecting to an X server. This also means clipboard
sharing to your system clipboard won't work, but it's good for a test.

Vim will try to connect to that X server by default because `DISPLAY` is
exported in the `.bashrc` file. Installing and configuring VcXsrv as per these
dotfiles will fix that issue.

If it still persists, it might be a software firewall issue. You can open TCP
port 6000 and also restrict access to it from only WSL 2. This will depend on
which tool you're using to configure that but that should do the trick.

## Ref / Credit
[https://nickjanetakis.com](https://nickjanetakis.com/)
[blog posts](https://nickjanetakis.com/blog/)