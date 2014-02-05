require 'rake'
require 'erb'
require 'mysql'
require 'yaml'

module Conversable
  ALERT_COLOR = 31
  PROMPT_COLOR = 33
  NOTICE_COLOR = 32

  def out(msg, code=36)
    puts "\033[#{code}m#{output}\033[0m"
  end

  def alert(msg)
    out(msg, ALERT_COLOR)
  end

  def notify(msg)
    out(msg, NOTICE_COLOR)
  end

  def ask?(msg)
    out(msg, PROMPT_COLOR)
    STDIN.gets.chomp!
  end

  def confirm?(msg)
    case ask?
    when 'Y', 'y', 'yes', 'YES'
      true
    else
      false
    end
  end
end

class Database
  include Conversable

  def self.create
    new.create
  end

  def self.destroy
    new.destroy
  end

  def create
    out "Creating database using settings from '#{craft_db_yml_path}'..."

    mysql_connect do |connection, config|
      if connection.list_dbs.include?(config['database'])
        alert "! Database '#{config['database']}' already exists. Skipping database creation..."
      else
        connection.query("CREATE DATABASE #{config['database']}")
        notify "✓ Database '#{config['database']}' created!"
      end
    end
  end

  def destroy
    mysql_connect do |connection, config|
      if connection.list_dbs.include?(config['database'])
        msg = "Are you sure you want to drop the database '#{config['database']}'? [yes/no]"

        if confirm?(msg)
          out 'Dropping database...'
          connection.query("DROP DATABASE #{config['database']}")
          notify '✓ Database dropped!'
        end
      else
        alert "! Database '#{config['database']}' doesn't exist. Aborting database operation..."
      end
    end
  end

  private
  def craft_db_yml_path
    Configuration::CRAFT_DB_YML_PATH
  end

  def mysql_connect
    if File.exists?(craft_db_yml_path)
      config = YAML.load(File.read(craft_db_yml_path))['development']

      begin
        connection = Mysql.new(config['server'], config['user'], config['password'])

        yield connection, config
      rescue Mysql::Error => e
        alert "! MySQL Error ##{e.errno}: #{e.error}"
      ensure
        connection.close if connection
      end
    else
      alert "! '#{craft_db_yml_path}' doesn't exist. Aborting database operation..."
    end
  end
end

class Configuration
  include Conversable

  attr_accessor :options

  CRAFT_DB_PHP_PATH = 'craft/config/db.php'
  CRAFT_DB_YML_PATH = 'config/database.yml'

  def self.create
    new.build.save
  end

  def self.destroy
    new.destroy
  end

  def initialize
    self.options = {}
  end

  def build
    if configuration_exists?
      alert "! '#{$craft_db_php_path}' already exists! Skipping database configuration..."
      return self
    end

    prompt_development_server_name
    prompt_development_db_settings

    self
  end

  def save
    write_db_settings unless configuration_exists?
    self
  end

  def destroy
    alert 'Removing configuration files...'
    system %Q{rm #{CRAFT_DB_YML_PATH} #{CRAFT_DB_PHP_PATH}}
    nofity '✓ Configuration files removed!'
  end

  def development_server_name
    options[:development_server_name]
  end

  private
  def configuration_exists?
    File.exists?(CRAFT_DB_PHP_PATH)
  end

  def prompt_development_server_name
    options[:development_server_name] = ask?('At what URL would you like to access your local Craft site (e.g. your-project-name.dev)?')
  end

  def prompt_development_db_settings
    notify "Generating database configuration files..."

    options[:development_db_server] = ask?('What is the server name or IP address of your database (e.g. localhost or 127.0.0.1)?')
    options[:development_db_user] = ask?('What is the database username?')
    options[:development_db_password] = ask?("What is the database password for user '#{options[:development_db_user]}'?")
    options[:development_db_name] = ask?('What is the name of the database?')
  end

  def write_db_settings
    out "Writing '#{CRAFT_DB_YML_PATH}'..."
    File.open(CRAFT_DB_YML_PATH, 'w') do |f|
      f.write ERB.new(File.read('config/templates/database.yml.erb')).result(binding)
    end

    out "Writing '#{CRAFT_DB_PHP_PATH}'..."
    File.open(CRAFT_DB_PHP_PATH, 'w') do |f|
      f.write ERB.new(File.read('config/templates/db.php.erb')).result(binding)
    end

    notify "✓ Database configuration files created!"
  end
end

class CraftInstall
  include Conversable

  REMOTE_SOURCE_URL = 'http://download.buildwithcraft.com/craft/1.3/1.3.2473/Craft-1.3.2473.zip'
  LOCAL_SOURCE_PATH = "tmp/#{REMOTE_SOURCE_URL.split('/').last}"

  BACKUPS_PATH = 'craft/storage/backups'

  def self.perform
    new.perform_install
  end

  def perform_install
    if craft_installed?
      alert("! It appears Craft is already installed. Skipping installation...")
      return
    end

    make_temp
    download_craft
    configure_htaccess
    create_backup
    cleanup

    notify '✓ Craft successfully downloaded and installed!'

    configure_git

    if continue_with_setup?
      CraftSetup.perform
    else
      out 'Skipping Craft configuration...'
      out 'To continue configuring Craft at a later time, run `rake craft:setup`.'
    end
  end

  private
  def craft_installed?
    File.directory?('craft') && File.directory?('public')
  end

  def continue_with_setup?
    confirm?("Would you like to continue configuring Craft? [yes/no]")
  end

  def make_temp
    `mkdir tmp` unless File.directory?('tmp')
  end

  def download_craft
    unless File.exists?(LOCAL_SOURCE_PATH)
      `curl -s #{REMOTE_SOURCE_URL} -o #{LOCAL_SOURCE_PATH}`
    end

    `unzip -oq #{LOCAL_SOURCE_PATH}`
  end

  def configure_htaccess
    if File.exists?('public/htaccess')
      out 'Creating .htaccess...'
      `mv public/{,.}htaccess`
    end
  end

  def backups_path
    BACKUPS_PATH
  end

  def create_backup
    if !File.directory?(backups_path)
      out "Creating backups folder at '#{backups_path}'..."
      system %Q{mkdir #{backups_path}}
      system %Q{echo '*\n!.gitignore\n!db_dump.zip\n' > #{backups_path}/.gitignore}
    end
  end

  def cleanup
   out 'Cleaning up installation files...'
    ['readme.txt', 'craft/config/db.php', 'public/web.config'].each do |filepath|
      system `rm #{filepath}` if File.exists?(filepath)
    end

    `rm -rf tmp` if File.directory?('tmp')
  end

  def configure_git
    out "\nConfiguring Git..."

    msg = 'What is your new project\'s clone URL (e.g. git@github.com:vigetlabs/your-project-name.git)?'
    git_clone_url = ask?(msg)

    if git_clone_url.empty?
      system %Q{git remote rm origin}
      alert '! Removing remote \'origin\'. Use `git remote add origin` to configure Git at a later time.'
    else
      system %Q{git remote set-url origin #{git_clone_url}}
      notify "✓ 'origin' is now set to '#{git_clone_url}'!"
    end
  end
end

class CraftSetup
  include Conversable

  attr_accessor :database_configuration

  def self.perform
    new.perform_setup
  end

  def perform_setup
    configure_database
    create_database
    set_folder_permissions

    display_apache_config

    notify "Congratulations! You've installed and configured Craft. Visit http://#{configuration.development_server_name}/admin to complete user registration."
  end

  def set_folder_permissions
    out 'Setting folder permissions (using \'sudo\')...'

    ['craft/app', 'craft/config', 'craft/storage'].each do |folder|
      `sudo chmod -R 777 #{folder}`
    end

    out '✓ Folder permissions set!', $notice
  end

  def display_apache_config
    out "Add the following to your server's virtual hosts file (e.g. '/etc/apache2/extra/httpd-vhosts.conf'):"

    out %Q{
      <Directory "#{Dir.getwd}/public/">
          Allow From All
          AllowOverride All
          Options +Indexes
      </Directory>
      <VirtualHost *:80>
          ServerName "#{configuration.development_server_name}"
          DocumentRoot "#{Dir.getwd}/public"
      </VirtualHost>
    }
  end

  private
  def configure_database
    self.configuration = Configuration.create
  end

  def create_database
    Database.create
  end
end

namespace :config do
  desc 'Configure database for local development'
  task :create do
    Configuration.create
  end

  desc 'Remove custom configuration files'
  task :destroy do
    Configuration.destroy
  end
end

namespace :craft do
  desc 'Install Craft'
  task :install do
    CraftInstall.perform
  end

  desc 'Set folder permissions for Craft'
  task :set_folder_permissions do
    CraftSetup.new.set_folder_permissions
  end

  desc 'Set up Craft for local development'
  task :setup do
    CraftSetup.perform
  end
end

namespace :db do
  desc 'Create database for local development'
  task :create do
    Database.create
  end

  desc 'Drop database'
  task :drop do
    Database.destroy
  end
end

task :install => 'craft:install'

task :default => :install
