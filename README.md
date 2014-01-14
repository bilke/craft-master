# craft-master

Instructions for installing and configuring a new project using [Craft](http://buildwithcraft.com/).

Use of these tools indicates that you have read, understand, and accepted Craft's [Terms and Conditions](http://buildwithcraft.com/license). Definitely give those a read before doing anything else.


## Dependencies

- MySQL (5.1.0+ with the InnoDB storage engine installed)
- PHP (5.3.0+ with the mcrypt extension installed)
- Ruby (2.1.0)
- Bundler (1.5.2+)


## Clone this repository

Issue the following command to clone this repository, substituting `your-project-name` as appropriate:

	git clone git@github.com:vigetlabs/craft-master.git your-project-name


## Install Dependencies

### Install MySQL

[Homebrew](http://brew.sh/) is the easiest (and best!) way to install MySQL:

	brew install mysql

### Install PHP with mcrypt extension

Again, using Homebrew, issue the following commands:

	brew tap homebrew/dupes
	brew tap josegonzalez/php
	brew install php54-mcrypt

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

First, start your local web server and MySQL. Second, run `rake install` and follow the prompts. You'll be asked for a clone URL to your project's Git repository (e.g. `git@github.com:vigetlabs/craft-master.git`) and some basic database-related questions. Most everything else is automated.

Craft will download, install, and create configuration files based on your input.

### Set up a Virtual Host

For Craft to work, you'll need to set up a virtual host with a `ServerName` of `http://your-project-name.dev` and a `DocumentRoot` pointed at the `public` folder in this repository.

### Finish Craft installation

At this point, you should be able to navigate to `http://your-project-name.dev/admin` and follow the remaining instructions for setting up Craft.

**Congratulations!**