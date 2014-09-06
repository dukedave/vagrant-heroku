This is a [veewee](https://github.com/jedi4ever/veewee) template for building a
[Vagrant](http://vagrantup.com/) box that closely mirrors the heroku Cedar stack. You can build it
yourself by following the directions below or install a prebuilt version from [here](http://dl.dropbox.com/u/1906634/heroku.box).

## Easy Install

Add the following to your `Vagrantfile`.

```ruby
Vagrant::Config.run do |config|
  config.vm.box = "heroku"
  config.vm.box_url = "https://dl.dropboxusercontent.com/s/rnc0p8zl91borei/heroku.box"
end
```

And run `vagrant up`. The box will be downloaded and imported for you.

This box was last updated 8/19/13.  For the latest changes, please follow the instructions below.

## Building From Scratch

First, clone the repo and install gems with bundler.

```bash
$ git clone https://github.com/ejholmes/vagrant-heroku.git
$ cd vagrant-heroku
$ bundle install
```

Next, build the box with veewee. Go grab a cup of coffee because this is gonna
take a while.

```bash
$ bundle exec veewee vbox build heroku
```

There is also a 2x dyno box available, just substitute every instance of `heroku` with `heroku-2x`.

And finally, install the box for use with Vagrant.

```bash
$ bundle exec veewee vbox export heroku
$ vagrant box add heroku heroku.box
```

Now all you have to do is setup vagrant in your project.

```bash
$ vagrant init heroku
$ vagrant up
$ vagrant ssh
```

## Included Packages

The packages that are included are carefully selected to closely match those on
the Celadon Cedar stack.

* Ubuntu 10.04 64bit
* Ruby 2.0.0-p247 MRI
* RubyGems 2.0.3
* Python with pip, virtualenv, and virtualenvwrapper
* PostgreSQL 9.2.4
* NodeJS 0.4.7
* Foreman https://github.com/ddollar/foreman

# Dave's tips:

## Postgres

All Postgres stuff is installed as the `postgres` user, in to `/var/pgsql/` (see [here](https://github.com/dukedave/vagrant-heroku/blob/cfe14b102c306c358e8b63b52b2330edb1c2bca6/definitions/heroku/postinstall.sh#L66)).

To connect from host use: `$ psql -h localhost -p 3432 vagrant vagrant`, that's *connect to the forwarded port on localhost, to a DB called vagrant, as a user (role) called vagrant*.

To restart Postgres:

1. `vagrant ssh` on to the box
1. `sudo su` to become root
1. `sudo -u postgres pg_ctl -D /var/pgsql/data/ restart`, this (as user `postgres`) tells `pg_ctl` to restart the DB at `/var/pgsql/data/`.

# Setup 

## Console

In `~/.bashrc`:

```
set -o vi
set editing-mode vi
cd /vagrant
```

In `~/.inputrc`:

```
set editing-mode vi
```

In `~/.editrc`:

```
bind -v
```

In `~/.irbrc`:

```
require 'irb/completion'
IRB.conf[:PROMPT_MODE] = :SIMPLE

require 'irb/ext/save-history'
ARGV.concat [ "--readline", "--prompt-mode", "simple" ]
IRB.conf[:SAVE_HISTORY] = 100
IRB.conf[:HISTORY_FILE] = "#{ENV['HOME']}/.irb-save-history" 
```

## Postgres

* Add port forward to `Vagrantfile`:
  `config.vm.network :forwarded_port, guest: 5432, host: 3432`
  (I chose `3432` to stick with the convention of having Vagrant's forwarded ports in the 3000s).

* Need to change listen address in `/var/pgsql/data/postgresql.conf` to find remote connection:
  `listen_addresses = '*'`.

* Need to change `IPv4` permissions in `/var/pgsql/data/pg_hba.conf` to allow remote connection:

  `host    all             all             all                     trust`

* Find `ERROR REPORTING AND LOGGING` in `/var/pgsql/data/postgresql.conf` and set:

  `log_destination = 'stderr'` and `logging_collector = on`

* When starting Unicorn from a fresh box you may see:

      ERROR -- : FATAL:  role "vagrant" does not exist

  Fix it with `sudo su` then `su -c 'createuser vagrant -s' postgres`.


## Redis

* Grab bz2 from [here](http://redis.io/download), `make`, `sudo make install`, then `./utils/install_server.sh` and follow instructions.

* Add port forward to `Vagrantfile`:
  `  config.vm.network :forwarded_port, guest: 6379, host: 3379`
  (I chose `3379` to stick with the convention of having Vagrant's forwarded ports in the 3000s).

* Change `bind` address per [this answer](http://stackoverflow.com/a/13928537/21115) and remember to restart Redis.

## Bundler

Tell bundler to prefer local gem git repos for `prop_store` and `places` with [a local override](http://stackoverflow.com/a/14167368/21115), e.g:
