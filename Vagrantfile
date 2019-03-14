# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-18.04"

  #To make the VM show up in the VM-Fusion tool
  config.vm.provider "vmware_desktop" do |v|
    v.gui = true
    v.memory = "8192"
    v.cpus = 2
  end

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.hostname = "kibana.nexar.be"

  config.vm.provision "shell", path: "./create_users"
  config.vm.provision "shell", path: "./change_ssh_config"

  config.vm.provision "shell", inline: <<-SHELL
  SHELL

  config.vm.provision :ansible do |ansible|
    ansible.playbook = "provisioning/elk.yml"
  end




end
