Github Page
==============

***Github page with Jekyll and Ruby***

**Author:** *Markus Rathgeb*

# Setup Requirements

Jekyll 3.9.0/3.9.2/... used for github pages does not work with Ruby 3.0, we need to switch do ruby 2.7

We will use Ruby Version Manager (RVM) from now as we do net depend on distribution and its version (e.g. Ruby 2.7 is not available on Fedora 37 anymore in the official repositories).

## Cleanup

If you ever set the GEM_HOME environment variable as we did before remove it.
E.g. remove that lines from your `.bashrc`

```
export GEM_HOME="/home/maggu2810/.gem_home"
export PATH="${GEM_HOME}/bin:${PATH}"
```

## Install RVM

```sh
gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

curl -sSL https://get.rvm.io | bash -s stable
```

```text
Downloading https://github.com/rvm/rvm/archive/1.29.12.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.12/1.29.12.tar.gz.asc
gpg: Signature made Fri 15 Jan 2021 07:46:22 PM CET
gpg:                using RSA key 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
gpg: Good signature from "Piotr Kuczynski <piotr.kuczynski@gmail.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 7D2B AF1C F37B 13E2 069D  6956 105B D0E7 3949 9BDB
GPG verified '/home/maggu2810/.rvm/archives/rvm-1.29.12.tgz'
Installing RVM to /home/maggu2810/.rvm/
    Adding rvm PATH line to /home/maggu2810/.profile /home/maggu2810/.mkshrc /home/maggu2810/.bashrc /home/maggu2810/.zshrc.
    Adding rvm loading line to /home/maggu2810/.profile /home/maggu2810/.bash_profile /home/maggu2810/.zlogin.
Installation of RVM in /home/maggu2810/.rvm/ is almost complete:

  * To start using RVM you need to run `source /home/maggu2810/.rvm/scripts/rvm`
    in all your open shell windows, in rare cases you need to reopen all shell windows.

Thanks for installing RVM 🙏
Please consider donating to our open collective to help us maintain RVM.

👉  Donate: https://opencollective.com/rvm/donate
```

Now **reboot**

## Install Ruby 2.7

See https://github.com/rvm/rvm/issues/5216#issuecomment-1206488598

```
rvm pkg install openssl
rvm install ruby-2.7 --with-openssl-dir=/home/maggu2810/.rvm/usr/
```

## Use Ruby 2.7

```
. "$HOME/.rvm/scripts/rvm"
rvm use ruby-2.7.2
```

## Install Dependencies - Jekyll

Create a temporary directory and change to that directory.

Create a file named `Gemfile` with the following content:

```text
source "https://rubygems.org"
gem "github-pages", "~> GITHUB-PAGES-VERSION", group: :jekyll_plugins
```

Replace GITHUB-PAGES-VERSION with the latest supported version of the github-pages gem. You can find this version here: "[Dependency versions](https://pages.github.com/versions/)."

The correct version Jekyll will be installed as a dependency of the `github-pages` gem.

```sh
bundle install
```

Remove temporary directory again
```

# Create new page

Now follow the instructions here to create a new site:

https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll

# Build site

```sh
bundle install
```

# Test site

https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll

```sh
bundle exec jekyll serve
```
