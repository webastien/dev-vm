# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.9"

VAGRANT_VERSION = '2'
VAGRANT_PLUGINS = [ 'vagrant-hostmanager', 'vagrant-auto_network' ]

VAGRANT_PLUGINS.each do |plugin|
  if !Vagrant.has_plugin? plugin
    print  "Plugin missing... "
    system "vagrant plugin install #{plugin}"
    abort  "Unable to install '#{plugin}'." unless Vagrant.has_plugin? plugin
  end
end

require 'yaml'
settings = YAML.load_file('provisioning/vars/default-config.yml')

if File.exist?('provisioning/vars/config.yml')
  settings.merge!(YAML.load_file('provisioning/vars/config.yml'))
end

Vagrant.configure(VAGRANT_VERSION) do |config|
  config.vm.box      = settings['machine']['box']
  config.vm.hostname = settings['machine']['hostname']
  config.ssh.forward_agent = true

  config.vm.network :private_network, ip: '0.0.0.0', auto_network: true
  config.vm.synced_folder settings['machine']['localDir'], '/home/vagrant/www', id: 'www', type: settings['machine']['syncType'], create: true

  config.vm.provider :virtualbox do |v|
    v.name         = settings['machine']['hostname']
    v.cpus         = settings['machine']['cpus']
    v.memory       = settings['machine']['memory']
    v.linked_clone = true

    v.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
    v.customize ['modifyvm', :id, '--natdnsproxy1',        'on']
    v.customize ['modifyvm', :id, '--ioapic',              'on']
  end

  if Vagrant.has_plugin?('vagrant-cachier')
    config.cache.auto_detect        = false
    config.cache.synced_folder_opts = { type: settings['machine']['syncType'], mount_options: ['rw', 'vers=3', 'tcp', 'nolock'] }

    config.cache.scope = :box
    config.cache.enable  :apt
  end

  config.vm.provision 'ansible' do |ansible|
    ansible.playbook = 'provisioning/playbook.yml'
  end

  config.hostmanager.manage_host = true
  config.hostmanager.enabled     = true
  config.hostmanager.aliases     = []

  settings['tasks']['sites']['vhosts'].each do |vhost|
    config.hostmanager.aliases.push(vhost['domain'])
    config.hostmanager.aliases.concat(vhost['aliases']) if vhost['aliases']
  end

  config.vm.provision :hostmanager
end

