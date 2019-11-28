# -*- mode: ruby -*-
# vi: set ft=ruby :

# load configs
require 'yaml'
current_dir    = File.dirname(File.expand_path(__FILE__))
configs        = YAML.load_file("#{current_dir}/vagrant.config.yml")
vagrant_config = configs['configs']

#require necessary plugins
required_plugins = %w( vagrant-vbguest vagrant-hostmanager )
required_plugins.each do |plugin|
  system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
end

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/bionic64"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = vagrant_config['memory']
    vb.cpus = vagrant_config['cpus']
  end

  # vagrant-hostmanager is necessary to update /etc/hosts on hosts and guests
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true
  config.vm.network "private_network", ip: vagrant_config['ip']
  config.vm.hostname = vagrant_config['domain']
  config.hostmanager.aliases = vagrant_config['aliases']

  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder "ansible_vagrant", "/vagrant/ansible_vagrant", create: true, owner: "vagrant", group: "vagrant", mount_options: ["dmode=775,fmode=775"]

  # auto update guest additions
  config.vbguest.auto_update = true

  # provision the vagrant machine using ansible
  # install ansible from its default repo
  config.vm.provision "ansible_local" do |ansible|
      ansible.install = true
      ansible.install_mode = :default
      ansible.playbook = "ansible_vagrant/playbook.yml"
      ansible.galaxy_role_file = "ansible_vagrant/requirements.yml"
      ansible.extra_vars = {
        ansible_python_interpreter: "/usr/bin/python3",
        GITLAB_IP: "#{vagrant_config['ip']}"
      }
  end
end
