# Howto

## Jekyll and Ruby

Jekyll 3.9.0 used for github pages does not work with Ruby 3.0, we need to switch do ruby 2.7.0

On Fedora (35):

```sh
dnf module list | grep ruby

sudo dnf module install ruby:2.7/default

sudo dnf install ruby ruby-devel openssl-devel redhat-rpm-config @development-tools
```

##  bashrc

```
export GEM_HOME="/home/maggu2810/.gem_home"
export PATH="${GEM_HOME}/bin:${PATH}"
```

## setup

https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll

```sh
gem install bundler
```

## Build site

```sh
bundle install
```

## Test site

https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll

```sh
bundle exec jekyll serve
```
