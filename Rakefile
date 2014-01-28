require 'rake'
require 'erb'
require 'mysql'
require 'yaml'

$craft_remote_src_url = 'http://download.buildwithcraft.com/craft/1.3/1.3.2473/Craft-1.3.2473.zip'
$craft_local_src_path = "tmp/#{$craft_remote_src_url.split('/').last}"

$craft_db_php_path = 'craft/config/db.php'
$craft_db_yml_path = 'config/database.yml'

$development_server_name = ''

$alert = 31
$notice = 32
$prompt = 33

def mysql_connect
  if File.exists?($craft_db_yml_path)
    config = YAML.load(File.read($craft_db_yml_path))['development']

    begin
      connection = Mysql.new(config['server'], config['user'], config['password'])

      yield connection, config
    rescue Mysql::Error => e
      out "! MySQL Error ##{e.errno}: #{e.error}", $alert
    ensure
      connection.close if connection
    end
  else
    out "! '#{$craft_db_yml_path}' doesn't exist. Aborting database operation...", $alert
  end
end

def out(output, code = 36)
  puts "\033[#{code}m#{output}\033[0m"
end

namespace :config do
  desc 'Configure database for local development'
  task :create do
    if File.exists?($craft_db_php_path)
      out "! '#{$craft_db_php_path}' already exists! Skipping database configuration...", $alert
    else
      out 'At what URL would you like to access your local Craft site (e.g. your-project-name.dev)?', $prompt
      $development_server_name = STDIN.gets.chomp!

      out "\nGenerating database configuration files..."

      out 'What is the server name or IP address of your database (e.g. localhost or 127.0.0.1)?', $prompt
      development_db_server = STDIN.gets.chomp!

      out 'What is the database username?', $prompt
      development_db_user = STDIN.gets.chomp!

      out "What is the database password for user '#{development_db_user}'?", $prompt
      development_db_password = STDIN.gets.chomp!

      out 'What is the name of the database?', $prompt
      development_db_database = STDIN.gets.chomp!

      out "Writing '#{$craft_db_yml_path}'..."
      File.open($craft_db_yml_path, 'w') do |f|
        f.write ERB.new(File.read('config/templates/database.yml.erb')).result(binding)
      end

      out "Writing '#{$craft_db_php_path}'..."
      File.open($craft_db_php_path, 'w') do |f|
        f.write ERB.new(File.read('config/templates/db.php.erb')).result(binding)
      end

      out '✓ Database configuration files created!', $notice
    end
  end

  desc 'Remove custom configuration files'
  task :destroy do
    out 'Removing configuration files...'
    system %Q{rm #{$craft_db_yml_path} #{$craft_db_php_path}}
    out '✓ Configuration files removed!', $notice
  end
end

namespace :craft do
  desc 'Install Craft'
  task :install do
    if (File.directory?('craft') && File.directory?('public'))
      out '! It appears Craft is already installed. Skipping installation...', $alert
    else
      if (!File.directory?('tmp'))
        system %Q{mkdir tmp}
      end

      out 'Downloading and installing Craft...'
      if !File.exists?($craft_local_src_path)
        system %Q{curl -s #{$craft_remote_src_url} -o #{$craft_local_src_path}}
      end
      system %Q{unzip -oq #{$craft_local_src_path}}

      if File.exists?('public/htaccess')
        out 'Creating .htaccess...'
        system %Q{mv public/{,.}htaccess}
      end

      backups_path = 'craft/storage/backups'

      if !File.directory?(backups_path)
        out "Creating backups folder at '#{backups_path}'..."
        system %Q{mkdir #{backups_path}}
        system %Q{echo '*\n!.gitignore\n!db_dump.zip\n' > #{backups_path}/.gitignore}
      end

      out 'Cleaning up installation files...'
      ['readme.txt', 'craft/config/db.php', 'public/web.config'].each do |filepath|
        if File.exists?(filepath)
          system %Q{rm #{filepath}}
        end
      end

      if File.directory?('tmp')
        system %Q{rm -rf tmp}
      end

      out '✓ Craft successfully downloaded and installed!', $notice

      out "\nConfiguring Git..."
      out 'What is your new project\'s clone URL (e.g. git@github.com:vigetlabs/your-project-name.git)?', $prompt

      git_clone_url = STDIN.gets.chomp!

      if git_clone_url.empty?
        system %Q{git remote rm origin}
        out '! Removing remote \'origin\'. Use `git remote add origin` to configure Git at a later time.', $alert
      else
        system %Q{git remote set-url origin #{git_clone_url}}
        out "✓ 'origin' is now set to '#{git_clone_url}'!", $notice
      end

      out "\nWould you like to continue configuring Craft? [Yn]", $prompt
      if STDIN.gets.chomp! == 'Y'
        Rake::Task['craft:setup'].invoke
      else
        out 'Skipping Craft configuration...'
        out 'To continue configuring Craft at a later time, run `rake craft:setup`.'
      end
    end
  end

  desc 'Set folder permissions for Craft'
  task :set_folder_permissions do
    out 'Setting folder permissions (using \'sudo\')...'

    ['craft/app', 'craft/config', 'craft/storage'].each do |folder|
      system %Q{sudo chmod -R 777 #{folder}}
    end

    out '✓ Folder permissions set!', $notice
  end

  desc 'Set up Craft for local development'
  task :setup do
    Rake::Task['config:create'].invoke
    puts "\n"
    Rake::Task['db:create'].invoke
    puts "\n"
    Rake::Task['craft:set_folder_permissions'].invoke

    out "\nAdd the following to your server's virtual hosts file (e.g. '/etc/apache2/extra/httpd-vhosts.conf'):"

    puts %Q{
      <Directory "#{Dir.getwd}/public/">
          Allow From All
          AllowOverride All
          Options +Indexes
      </Directory>
      <VirtualHost *:80>
          ServerName "#{$development_server_name}"
          DocumentRoot "#{Dir.getwd}/public"
      </VirtualHost>
    }

    out "Congratulations! You've installed and configured Craft. Visit http://#{$development_server_name}/admin to complete user registration.", $notice
  end
end

namespace :db do
  desc 'Create database for local development'
  task :create do
    out "Creating database using settings from '#{$craft_db_yml_path}'..."

    mysql_connect do |connection, config|
      if connection.list_dbs.include?(config['database'])
        out "! Database '#{config['database']}' already exists. Skipping database creation...", $alert
      else
        connection.query("CREATE DATABASE #{config['database']}")
        out "✓ Database '#{config['database']}' created!", $notice
      end
    end
  end

  desc 'Drop database'
  task :drop do
    mysql_connect do |connection, config|
      if connection.list_dbs.include?(config['database'])
        out "Are you sure you want to drop the database '#{config['database']}'? [Yn]", $prompt

        if STDIN.gets.chomp! == 'Y'
          out 'Dropping database...'
          connection.query("DROP DATABASE #{config['database']}")
          out '✓ Database dropped!', $notice
        end
      else
        out "! Database '#{config['database']}' doesn't exist. Aborting database operation...", $alert
      end
    end
  end
end

task :install => 'craft:install'