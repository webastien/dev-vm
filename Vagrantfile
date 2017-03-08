# -*- mode: ruby -*-
# vi: set ft=ruby :
# Little hack: add "symbolize_keys" method on Hash class (exists in ROR but not in pure Ruby)
class Hash; def symbolize_keys; Hash[self.map{ |k, v| [k.to_sym, v] }]; end; end
# Include YAML library to read config
require 'yaml'
# Init the main variables used in this file
current_dir   = File.dirname(File.expand_path(__FILE__))
configuration = Hash.new
# Inject default configuration, eventually overriden by user values...
Dir["#{current_dir}/configuration/default/settings/*.yml"].each do |file|
  # Extract the filename without the YAML extension to use it as "key"
  key = File.basename(file, '.yml')
  # Store the settings of the file in the keyed index of "configuration"
  configuration.store(key, YAML.load_file(file))
  # If user has override this settings, merge its defined values
  if File.exist?("#{current_dir}/configuration/yours/settings/#{key}.yml")
    configuration[key].merge!(YAML.load_file("#{current_dir}/configuration/yours/settings/#{key}.yml"))
  end
end
# Inject default files path, eventually overriden by user ones...
['templates', 'starters'].each do |section|
  configuration[section] = {} # Init the section in configuration variable
  # Check all files for this section
  Dir["#{current_dir}/configuration/default/#{section}/*"].each do |file|
    fileName = File.basename(file)
    userFile = "#{current_dir}/configuration/yours/#{section}/#{fileName}"
    # Add the path in the config, default if not overriden
    configuration[section][fileName] = File.exist?(userFile)? userFile: file
  end
end
# Fix the minimal version of vagrant to execute this script
Vagrant.require_version ">= #{configuration['vagrant']['minimal_version']}"
# Ensure required plugins are present, try to install if not
configuration['vagrant']['required_plugins'].each do |plugin|
  if !Vagrant.has_plugin? plugin
    print  "Plugin #{plugin} is missing, try to install it..."
    system "vagrant plugin install #{plugin}"
    abort  "Unable to install '#{plugin}'." unless Vagrant.has_plugin? plugin
  end
end
# Check suggested plugins are present, ask for install if not
configuration['vagrant']['suggested_plugins'].each do |plugin|
  if !Vagrant.has_plugin? plugin
    print "Plugin #{plugin} is missing, it's not required but suggested. Install it? [y/n] "
    prompt = STDIN.gets.chomp
    system "vagrant plugin install #{plugin}" if prompt == 'y'
  end
end
# Configure the virtual machine
Vagrant.configure(configuration['vagrant']['config_version']) do |config|
  # Set the machine name (visible in vargant status)
  config.vm.define configuration['vm']['machineName']
  # Allow SSH connection to the VM
  config.ssh.forward_agent = true
  # Change the SSH port to prevent collisions
  config.ssh.port = configuration['vm']['sshPort']
  # Configure the synchronized folders
  configuration['vm']['syncFolders'].each do |folder|
    config.vm.synced_folder folder['local'], folder['remote'], folder['options'].symbolize_keys
  end
  # Basic box informations
  config.vm.box      = configuration['vm']['box']
  config.vm.hostname = configuration['vm']['hostname']
  # Virtualbox specific settings...
  config.vm.provider :virtualbox do |v|
    v.name         = configuration['vm']['hostname']
    v.cpus         = configuration['vm']['cpus']
    v.memory       = configuration['vm']['memory']
    v.linked_clone = configuration['vm']['clone']
    # Tell Virtualbox using specifics modifiers
    configuration['vm']['virtualBoxModifiers'].each do |key, value|
      v.customize ['modifyvm', :id, "--#{key}", value]
    end
  end
  # Networking configuration...
  config.vm.network :forwarded_port,  id: 'ssh', guest: 22, host: configuration['vm']['sshPort']
  config.vm.network :private_network, ip: configuration['vm']['private_IP'], auto_network: Vagrant.has_plugin?('vagrant-auto_network')
  # Vagrant hostmanager plugin settings...
  if Vagrant.has_plugin?('vagrant-hostmanager')
    config.hostmanager.manage_host = true
    config.hostmanager.enabled     = true
    config.hostmanager.aliases     = []
    # Build the list of domains / aliases we need on the VM
    configuration['vhosts'].each do |domain,vhost|
      config.hostmanager.aliases.push(domain)
      config.hostmanager.aliases.concat(vhost['aliases']) if vhost['aliases']
    end
    # Tell vagrant to use Vagrant Hostmanager
    config.vm.provision :hostmanager
  end
  # Vagrant cachier plugin settings...
  if configuration['vm']['cachier']['is_activated'] = Vagrant.has_plugin?('vagrant-cachier')
    config.cache.synced_folder_opts = configuration['vm']['cachier']['syncFolderOpts'].symbolize_keys
    config.cache.auto_detect        = (configuration['vm']['cachier']['enabledBuckets'].size == 0)
    # In case of auto-detect has been disabled (buckets given), configure them...
    if !config.cache.auto_detect
      # Generic bucket is the only bucket with options. YAML loaded in Ruby convert keys in string but we need symbols here...
      def bucketizeOptions(options) # <-- Custom function to convert "cache_dir" key to Ruby symbol
        if options.key?('cache_dir'); options = options.symbolize_keys; else
           options.each do |key, option|; options[key] = bucketizeOptions(option); end; end
        options
      end
      # Activate the buckets
      configuration['vm']['cachier']['enabledBuckets'].each do |bucket, options|
        config.cache.enable :"#{bucket}", bucketizeOptions(options)
      end
    end
    # Restrict the cache to this vm, or to the base box if we use clone
    config.cache.scope = configuration['vm']['clone']? :box : :machine
  end
  # Configure ansible...
  config.vm.provision 'ansible' do |ansible|
    ansible.playbook   = "#{current_dir}/provisioning/playbook.yml"
    ansible.extra_vars = { configuration: configuration }
  end
end

