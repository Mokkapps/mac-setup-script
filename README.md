# Setting up a new computer

My personal installation script for a new computer.

Note, this sets up an extremely opinionated and highly personalized installation, with my preferences and dotfiles. Please adjust as needed!

My dotfile repo is currently private, but I plan on creating a public repo before the end of the year.

Some resources borrowed from:

- https://github.com/ruyadorno/installme-osx/
- https://gist.github.com/millermedeiros/6615994
- https://gist.github.com/brandonb927/3195465/

## Install from script:

Open the terminal, then:

```sh
bash -c "`curl -L https://git.io/new-computer`"
```

This command runs the following script:

```shell
#                    _           _        _ _ 
#  ___  _____  __   (_)_ __  ___| |_ __ _| | |
# / _ \/ __\ \/ /   | | '_ \/ __| __/ _` | | |
#| (_) \__ \>  <    | | | | \__ \ || (_| | | |
# \___/|___/_/\_\   |_|_| |_|___/\__\__,_|_|_|


echo "I  ❤️  🍎"
echo "Mac OS Install Setup Script"
echo "By Nina Zakharenko"
echo "Follow me on twitter! https://twitter.com/nnja"

# Some configs reused from:
# https://github.com/ruyadorno/installme-osx/
# https://gist.github.com/millermedeiros/6615994
# https://gist.github.com/brandonb927/3195465/

# Colorize

# Set the colours you can use
black=$(tput setaf 0)
red=$(tput setaf 1)
green=$(tput setaf 2)
yellow=$(tput setaf 3)
blue=$(tput setaf 4)
magenta=$(tput setaf 5)
cyan=$(tput setaf 6)
white=$(tput setaf 7)

# Resets the style
reset=`tput sgr0`

# Color-echo. Improved. [Thanks @joaocunha]
# arg $1 = message
# arg $2 = Color
cecho() {
  echo "${2}${1}${reset}"
  return
}

echo ""
cecho "###############################################" $red
cecho "#        DO NOT RUN THIS SCRIPT BLINDLY       #" $red
cecho "#         YOU'LL PROBABLY REGRET IT...        #" $red
cecho "#                                             #" $red
cecho "#              READ IT THOROUGHLY             #" $red
cecho "#         AND EDIT TO SUIT YOUR NEEDS         #" $red
cecho "###############################################" $red
echo ""

# Set continue to false by default.
CONTINUE=false

echo ""
cecho "Have you read through the script you're about to run and " $red
cecho "understood that it will make changes to your computer? (y/n)" $red
read -r response
if [[ $response =~ ^([yY][eE][sS]|[yY])$ ]]; then
  CONTINUE=true
fi

if ! $CONTINUE; then
  # Check if we're continuing and output a message if not
  cecho "Please go read the script, it only takes a few minutes" $red
  exit
fi

# Here we go.. ask for the administrator password upfront and run a
# keep-alive to update existing `sudo` time stamp until script has finished
sudo -v
while true; do sudo -n true; sleep 60; kill -0 "$$" || exit; done 2>/dev/null &


##############################
# Prerequisite: Install Brew #
##############################

echo "Installing brew..."

if test ! $(which brew)
then
	## Don't prompt for confirmation when installing homebrew
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" < /dev/null
fi

# Latest brew, install brew cask
brew upgrade
brew update
brew tap caskroom/cask


#############################################
### Generate ssh keys & add to ssh-agent
### See: https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/
#############################################

echo "Generating ssh keys, adding to ssh-agent..."
read -p 'Input email for ssh key: ' useremail

echo "Use default ssh file location, enter a passphrase: "
ssh-keygen -t rsa -b 4096 -C "$useremail"  # will prompt for password
eval "$(ssh-agent -s)"

# Now that sshconfig is synced add key to ssh-agent and
# store passphrase in keychain
ssh-add -K ~/.ssh/id_rsa

# If you're using macOS Sierra 10.12.2 or later, you will need to modify your ~/.ssh/config file to automatically load keys into the ssh-agent and store passphrases in your keychain.

if [ -e ~/.ssh/config ]
then
    echo "ssh config already exists. Skipping adding osx specific settings... "
else
	echo "Writing osx specific settings to ssh config... "
   cat <<EOT >> ~/.ssh/config
	Host *
		AddKeysToAgent yes
		UseKeychain yes
		IdentityFile ~/.ssh/id_rsa
EOT
fi

#############################################
### Add ssh-key to GitHub via api
#############################################

echo "Adding ssh-key to GitHub (via api)..."
echo "Important! For this step, use a github personal token with the admin:public_key permission."
echo "If you don't have one, create it here: https://github.com/settings/tokens/new"

retries=3
SSH_KEY=`cat ~/.ssh/id_rsa.pub`

for ((i=0; i<retries; i++)); do
      read -p 'GitHub username: ' ghusername
      read -p 'Machine name: ' ghtitle
      read -sp 'GitHub personal token: ' ghtoken

      gh_status_code=$(curl -o /dev/null -s -w "%{http_code}\n" -u "$ghusername:$ghtoken" -d '{"title":"'$ghtitle'","key":"'"$SSH_KEY"'"}' 'https://api.github.com/user/keys')

      if (( $gh_status_code -eq == 201))
      then
          echo "GitHub ssh key added successfully!"
          break
      else
			echo "Something went wrong. Enter your credentials and try again..."
     		echo -n "Status code returned: "
     		echo $gh_status_code
      fi
done

[[ $retries -eq i ]] && echo "Adding ssh-key to GitHub failed! Try again later."


##############################
# Install via Brew           #
##############################

echo "Starting brew app install..."

### Developer Tools
brew cask install hyper

### Development
brew install node
brew install yarn

### Command line tools - install new ones, update others to latest version
brew install git  # upgrade to latest
brew install git-lfs # track large files in git https://github.com/git-lfs/git-lfs
brew install wget
brew install zsh # zshell

### Dev Editors 
brew cask install visual-studio-code
brew cask install webstorm

### Writing

### Conferences, Blogging, Screencasts

### Productivity
brew cask install google-chrome
brew cask install timing  # time and project tracker

### Keyboard & Mouse

### Chat / Video Conference
brew cask install franz

### Music and Video
brew cask install spotify

### Other tools
brew cask install enpass
brew cask install ccleaner

### Run Brew Cleanup
brew cleanup

#############################################
### Fonts
#############################################

echo "Installing fonts..."

brew tap caskroom/fonts

### programming fonts
brew cask install font-fira-mono-for-powerline
brew cask install font-fira-code

### SourceCodePro + Powerline + Awesome Regular (for powerlevel 9k terminal icons)
cd ~/Library/Fonts && { curl -O 'https://github.com/Falkor/dotfiles/blob/master/fonts/SourceCodePro+Powerline+Awesome+Regular.ttf?raw=true' ; cd -; }


#############################################
### Installs from Mac App Store
#############################################

echo "Installing apps from the App Store..."

### find app ids with: mas search "app name"
brew install mas

### Mas login is currently broken on mojave. See:
### Login manually for now.

cecho "Need to log in to App Store manually to install apps with mas...." $red
echo "Opening App Store. Please login."
open "/Applications/App Store.app"
echo "Is app store login complete.(y/n)? "
read response
if [ "$response" != "${response#[Yy]}" ]
then
	# mas install 907364780  # Tomato One - Pomodoro timer
	# mas install 485812721  # Tweetdeck
	# mas install 668208984  # GIPHY Capture. The GIF Maker (For recording my screen as gif)
	# mas install 1351639930 # Gifski, convert videos to gifs
	# mas install 414030210  # Limechat, IRC app.
else
	cecho "App Store login not complete. Skipping installing App Store Apps" $red
fi

#############################################
### Set OSX Preferences - Borrowed from https://github.com/mathiasbynens/dotfiles/blob/master/.macos
#############################################

# Close any open System Preferences panes, to prevent them from overriding
# settings we’re about to change
osascript -e 'tell application "System Preferences" to quit'


##################
### Finder, Dock, & Menu Items
##################

# Finder: show all filename extensions
defaults write NSGlobalDomain AppleShowAllExtensions -bool true

# Avoid creating .DS_Store files on network or USB volumes
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool true
defaults write com.apple.desktopservices DSDontWriteUSBStores -bool true

# Minimize windows into their application’s icon
defaults write com.apple.dock minimize-to-application -bool true

# Don’t show recent applications in Dock
#    defaults write com.apple.dock show-recents -bool false

###############################################################################
# Trackpad, mouse, keyboard, Bluetooth accessories, and input                 #
###############################################################################

# Disable “natural” (Lion-style) scrolling
# Uncomment if you don't use scroll reverser
# defaults write NSGlobalDomain com.apple.swipescrolldirection -bool false


###############################################################################
# Mac App Store                                                               #
###############################################################################

# Enable the automatic update check
defaults write com.apple.SoftwareUpdate AutomaticCheckEnabled -bool true

# Download newly available updates in background
defaults write com.apple.SoftwareUpdate AutomaticDownload -int 1

# Install System data files & security updates
defaults write com.apple.SoftwareUpdate CriticalUpdateInstall -int 1

###############################################################################
# Photos                                                                      #
###############################################################################

# Prevent Photos from opening automatically when devices are plugged in
defaults -currentHost write com.apple.ImageCapture disableHotPlug -bool true

echo ""
cecho "Done!" $cyan
echo ""
echo ""
cecho "################################################################################" $white
echo ""
echo ""
cecho "Note that some of these changes require a logout/restart to take effect." $red
echo ""
echo ""
echo -n "Check for and install available OSX updates, install, and automatically restart? (y/n)? "
read response
if [ "$response" != "${response#[Yy]}" ] ;then
    softwareupdate -i -a --restart
fi
```


----


# Manual Configuration

These apps need to be configured manually.

For OSX settings, I'm still looking for the command line way to change this preference.

#### Set Scroll Reverser preferences

##### Scrolling Section

Checked:

- Reverse Scrolling
- Reverse vertical
- Reverse horizontal
- Reverse Mouse

Unchecked:
- Reverse Trackpad

##### App Section

Checked:

- Start at login

Unchecked:

- Show in menu bar

#### Iterm2

* Iterm2 -> Preferences -> General
	* Check: Load preferences from custom folder /Users/nina/dotfiles/iterm-profiles
	* Check: Save changes to folder when Iterm2 quits

#### System Preferences Configuration

**Users & Groups**

* make sure guest account is turned off

**Keyboard**

* Use F1, F2 as standard function keys: ON
* Turn off spotlight (use Alfred instead)
	* Keyboard -> Shortcuts -> Spotlight -> Deselect all
* Customize control strip -> Delete siri from touchbar
* Touch Bar Shows -> F1, F2, etc. Keys

**General**

* Recent items: None
* Appearance: Graphite
* Highlight color: Pink

**Mission Control**

* Dashboard: As Overlay

**Desktop & Screen Saver**

* Screensaver
	* Hot Corners
		* Top right -> Dashboard
		* Bottom right -> Start screen saver

**Trackpad**

* Tap to click: ON
* Secondary click: ON

**Display**

* Night Shift (flux) -> Schedule -> Sunrise to Sunset

**Security & Privacy**

* Firewall -> on
* Firewall options -> Enable stealth mode
* FileVault -> Turn On FileVault (encrypt harddrive)

**Sharing**

* Ensure everything is unchecked

**Finder Preferences**

* General
	* New finder window show: home folder
* Sidebar
	* Show home, remove Recents
* Advanced
	* Show all filename extensions: ON
	* Show warning before changing an extension: OFF
	* Show warning before emptying the trash: OFF

**TrackPad**

* Point & Click
	* Silent clicking -> On

#### Chrome

* Sign into chrome to sync profile & bookmarks
* Chrome -> Warn before quitting: ON

#### Alfred

* Set Double Cmd to Alfred hotkey

#### Configure VPN

#### SizeUp

* configure license
* settings -> automatically start at login

#### Karabiner Elements

* Simple Modifications
	* From key caps_lock to key left_control (Remap caps lock to control for emacs)
	* From key right_control to key `b` (custom setting for my ergonomic keyboard)
* Devices
	* For all devices -> Caps Lock LED Check Manipulate LED

#### Arduino IDE (Set up to use with Adafruit boards)

* Preferences -> Additional Boards Manager URLS -> https://adafruit.github.io/arduino-board-index/package_adafruit_index.json
* Tools -> Board -> Board Manager. Install:
	* Arduino SAMD Boards
	* Adafruit AVR Boards
* Sketch -> Include Library -> Manage Libraries. Install:
	* Adafruit NeoPixel
	* FastLED

#### Dash

* Download
* docs for:
	* python2
	* python3
	* django
	* flask
	* arduino
	* processing
	* man pages
* cheatsheets for:
	* emacs
	* git
	* python debugger
	* python unittest assertions

### Limechat

* log into freenode

### Widgets

Useful [shipping status tracker widget](https://junecloud.com/software/dashboard/delivery-status.html)

#### Optional

Apps:

* [Send to kindle for Mac](https://www.amazon.com/gp/sendtokindle/mac)
* [Delivery Status Dashboard Widget](https://www.apple.com/downloads/dashboard/shopping/deliverystatus.html)
* [Flip Clock Screensaver](https://fliqlo.com/#about)
* [FlyCut Clipboard Manager](https://github.com/TermiT/Flycut)
* Visit [this page](http://optout.aboutads.info/) to opt out of ads (turn off adblock first)

#### Todos

- Configure touchbar?
	- See: https://blog.eriknicolasgomez.com/2016/11/28/managing-or-setting-the-mini-touchbar-control-strip/
- Set up wallpaper
	- osascript -e 'tell application "Finder" to set desktop picture to "/Users/wilsonmar/Pictures/wallpapers/macOS-High-Sierra-Wallpaper-2880x1494.jpg" as POSIX file'
- Set up alfred powerpack
