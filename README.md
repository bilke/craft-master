# craft-master

Instructions for installing and configuring a new [Craft](http://buildwithcraft.com/) project using Ruby and Rake. For more on the thinking behind craft-master, check out the [introductory blog post](http://viget.com/extend/kickstarting-your-craft-project-with-ruby-and-rake).


## Table of Contents

1. [Dependencies](#dependencies)
2. [Getting Started](#getting-started)
3. [Clone craft-master](#clone-craft-master)
4. [Install Dependencies](#install-dependencies)
5. [Install and Configure Craft](#install-and-configure-craft)


## Dependencies

- MySQL (5.1.0+ with the InnoDB storage engine installed)
- PHP (5.3.0+ with the mcrypt extension installed)
- Ruby (2.1.0)
- Bundler (1.5.2+)


## Getting Started

Before you dive in, make sure you have a couple of pieces of information handy:

1. Your project repository's Git clone URL (e.g. `git@github.com:vigetlabs/your-project-name.git`). During installation, you'll be asked to provide this so that you can `git push` your work to your own repository.
2. Choose a virtual host name for your local site (e.g. `your-project-name.dev`).
3. Know your database server's accessible address (e.g. `127.0.0.1` or `localhost`) and the user and password that can access it.

Also, these instructions assume you have a passing familiarity with the command line, MySQL, PHP, and Ruby, and that you're using Mac OS X. This setup will likely work on other systems, but the instructions are geared toward Mac OS X.


## Clone craft-master

To get started, issue the following command to clone this repository, substituting `your-project-name` as appropriate:

	git clone https://github.com/vigetlabs/craft-master.git your-project-name


## Install Dependencies

### Install MySQL and PHP with mcrypt extension

[Homebrew](http://brew.sh/) is the easiest (and best!) way to install MySQL and PHP:

	brew tap homebrew/dupes
	brew tap josegonzalez/php
	brew install mysql php54-mcrypt

This will also install PHP 5.4 as a dependency. Be sure to follow the instructions listed in the Caveats to enable the mcrypt extension.

### Install Ruby

Using [rbenv](https://github.com/sstephenson/rbenv) or [RVM](http://rvm.io/), install Ruby 2.1.0.

If you use rbenv, issue the following command to set your local Ruby version:

	rbenv local 2.1.0

If you're using RVM, issue the following command, substituting `your-project-name` as appropriate:

	echo "rvm use 2.1.0@your-project-name --create" > .rvmrc

### Install Bundler

	gem install bundler

### Install gems

Run `bundle` to install all required gems.


## Install and Configure Craft

After starting up your local web and MySQL servers, run `rake install` and follow the prompts.

**Note**: If you use MySQL from [Boxen](https://github.com/boxen) use this socket `/opt/boxen/data/mysql/socket`.

You'll be asked for a clone URL to your project's Git repository (e.g. `git@github.com:vigetlabs/craft-master.git`) and some basic database-related questions. Most everything else is automated.

Craft will download, install, and create configuration files based on your input.

### Set up a Virtual Host

For Craft to work, you'll need to set up a virtual host with a `ServerName` of `http://your-project-name.dev` and a `DocumentRoot` pointed at the `public` folder in your working directory.

For convenience, the installer will output a sample bit of code for you to insert into your server's virtual hosts file (which, depending on your setup, may be located at `/etc/apache2/extra/httpd-vhosts.conf`).

Add an entry to `/etc/hosts`: `127.0.0.1 your-project-name.dev`

Restart apache with `sudo apachectl restart`.

### Finish Craft installation

At this point, you should be able to navigate to `http://your-project-name.dev/admin` and follow the remaining instructions for setting up Craft.

**Congratulations, you're done!**


## Authors

- [@jgarber623](https://github.com/jgarber623)

Of course, craft-master wouldn't be possible without the hard work of the fine folks at [Pixel & Tonic](http://pixelandtonic.com/). Thanks!


## License

Use of these tools indicates that you have read and accepted Craft's [Terms and Conditions](http://buildwithcraft.com/license). Definitely give those a read. craft-master is not affiliated with Craft or Pixel & Tonic.

The source code and documentation for craft-master is licensed under the incredibly permissive [MIT License](http://opensource.org/licenses/MIT).
