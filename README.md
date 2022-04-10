# Readme

## Installing Ruby and Jekyll

### Windows

[RubyInstaller](https://rubyinstaller.org/) is a self-contained Windows-based installer that includes the Ruby language, and execution environment, important documentation and more.

1. Download and install a Ruby+Devkit version from [RubyInstaller Downloads](https://rubyinstaller.org/downloads/).
   Use default options for installation.

2. Run the `ridk install` step on the last stage of the installation wizard.
   This is needed for installing gems with native extensions.

3. Open a new command prompt window from the start menu, so that changes to the `PATH` environment variable becomes effective.
   Install Jekyll and Bundler using `gem install jekyll bundler`.

4. Check if Jekyll has been installed properly: `jekyll -v`

### Arch Linux

Jekyll can be installed in [Arch Linux](https://wiki.archlinux.org/title/jekyll) with the [RubyGems](https://en.wikipedia.org/wiki/RubyGems) package manager or using the applicable packages in the AUR.
Both methods require the Ruby package in the official repositories to be installed.

#### RubyGems (Recommended)

The best way to install Jekyll is with [RubyGems](https://en.wikipedia.org/wiki/RubyGems), which is a package manager for the Ruby programming language. RubyGems comes with the [ruby](https://archlinux.org/packages/?name=ruby) package.
Jekyll can then be installed for all users on the machine using the `gem` command as root.
Alternative installation methods are available on the [Ruby page](https://wiki.archlinux.org/title/Ruby#RubyGems).

Before installing Jekyll make sure to update RubyGems (note that all the following gem commands install for your user only; please do not use gem as root = no sudo here).

```sh
$ gem update
```

Then install Jekyll and bundler using the `gem` command.

```
$ gem install jekyll bundler
```

Update the bundle.

```
$ bundle update
```

#### AUR (Alternate)

Alternately, [jekyll](https://aur.archlinux.org/packages/jekyll/)<sup>AUR</sup> can be installed from the AUR.

## Setup

### Initialize a new Jekyll site

Run `jekyll new . --force` inside the repository folder to initialize a new project.

```
jekyll new . --force
```

### config.yml

### Gemfile

## Troubleshooting

### Live Jekyll Server

#### Issue

`bundle exec jekyll serve --livereload` - [cannot load such file](https://stackoverflow.com/questions/65989040/bundle-exec-jekyll-serve-cannot-load-such-file)

#### Solution

Run the following commands

```
bundle add webrick
gem install github-pages
gem install jekyll
gem install jekyll-feed
bundle install
bundle update
```

Remove `Gemfile.lock`.
