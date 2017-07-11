# Add Safari Reading List Bookmarks to Pinboard Unread List

## Problem

Pinboard is great, but the integration into iOS and Mac can be annoying. Too many steps: copy and paste a url into the browser, click the Pinboard bookmarklet, enter tags, etc. On mobile, it's even more steps, and even harder to do.

## Solution

This ruby script automates moving your Reading List bookmarks to Pinboard's "to read" list. It is meant to be run via a scheduled job (with launchd). 

The script works like so:

1. read all Safari bookmarks in the "Reading List"
2. check each to see if the url already exists on pinboard
3. add the bookmarks on Pinboard

If a Reading List bookmark exists on Pinboard and has no tags, the script will generate some using Pinboard's suggestions, matching tags you already have, plus some custom rules. 

For bookmarks older than 90 days (adjustable), it will sync the bookmark but disable the "toread" flag on Pinboard.

It should be safe to re-run this script over and over without harm. You can clear out your Safari Reading List bookmarks and the posts will remain on Pinboard.

## How It Works

Safari stores bookmarks in a plist file located in `~/Library/Safari/Bookmarks.plist`. By piping this through [plutil](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/plutil.1.html), a parsable XML file is produced. Bookmarks from the Reading List are extracted, and one-by-one, checked and/or added using the [Pinboard service API](https://pinboard.in/api/).

## Setup

You need RVM, Ruby, and bundler, etc:

### Install RVM

```
    gpg --keyserver hkp://pgp.mit.edu --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
    curl -sSL https://get.rvm.io | bash -s stable --ruby
    echo "source $HOME/.rvm/scripts/rvm" > ~/.profile
    source "$HOME/.rvm/scripts/rvm"
    rvm install ruby 2.2
    gem install bundler
```

### Checkout & prepare project

```
mkdir -p ~/src
cd ~/src
git clone git@github.com:eeichinger/reading-list-sync.git
cd ~/reading-list-sync
bundle
cp ./config/secrets_example.yml ./config/secrets.yml
cp ./launchd/reading-list-sync.example.plist ./launchd/reading-list-sync.plist
```

Edit the file `config/secrets.yml` and replace the value of `pinboard_api_key` with your [actual Pinboard API key](https://pinboard.in/settings/password).


### Schedule with launchd

To schedule this as job with launchd, copy or link the launchd plist file [./launchd/reading-list-sync.plist](./launchd/reading-list-sync.plist) into `/Library/LaunchDaemons`

Edit the file `./launchd/reading-list-sync.plist` and replace all values of `/Users/erich` with your home folder.

```
# create ruby alias
rvm alias create reading-list-sync ruby-2.2.6
# link plist file
sudo ln -s ~/src/reading-list-sync/launchd/reading-list-sync.plist /Library/LaunchDaemons/reading-list-sync.plist
# adjust permissions for launchd
sudo chown root:wheel ~/src/reading-list-sync/launchd/reading-list-sync.plist
# load it
launchctl load /Library/LaunchDaemons/reading-list-sync.plist
```

For more infos see

- [Install RVM](https://rvm.io/rvm/install)
- [Configure ZSH for RVM if necessary](https://stackoverflow.com/questions/22773693/rvm-zsh-rvm-is-not-a-function-selecting-rubies-with-rvm-use-will-not-w)
- [Configure launchd](http://www.splinter.com.au/using-launchd-to-run-a-script-every-5-mins-on/) and [Create ruby launchd task](http://notes.jerzygangi.com/creating-a-ruby-launchd-task-with-rvm-in-os-x/)


## Usage

$ script/sync_bookmarks

## Benefits

When you are using iOS, or any mac, and you "add to reading list", the bookmark will appear on Pinboard, with tags. This allows you to keep using Pinboard (even via RSS) to keep up with your reading.
