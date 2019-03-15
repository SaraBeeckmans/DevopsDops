# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-18.04"

  #To make the VM show up in the VM-Fusion tool
  config.vm.provider "vmware_desktop" do |v|
    v.gui = true
    v.memory = "8192"
    v.cpus = 1
  end

  config.vm.provider "virtualbox" do |vb|
     vb.gui = false
     vb.memory = "512"
     vb.cpus = 1
   end

N=1
BasecountIP = 9


  config.vm.define "lb" do |lb|
     lb.vm.box = "bento/ubuntu-18.04"
     lb.vm.network "private_network", ip: "192.168.50.10"
     lb.vm.provision "shell", path: "./create_users"
     lb.vm.provision "shell", path: "./change_ssh_config"

     lb.vm.provision :ansible do |ansible|
       ansible.playbook = "playbooks/lb.yml"
      end
   end

(1..N).each do |machine_id|
  config.vm.define "frontend-#{BasecountIP+machine_id}" do |frontend|
    frontend.vm.box = "bento/ubuntu-18.04"
    frontend.vm.network "private_network", ip: "192.168.100.#{BasecountIP+machine_id}"

    if machine_id == N
      frontend.vm.provision "shell", path: "./create_users"
      frontend.vm.provision "shell", path: "./change_ssh_config"
      frontend.vm.provision :ansible do |ansible|
        ansible.limit = "all"
        ansible.playbook = "playbooks/frontend.yml"
      end
    end
  end
end


#  config.vm.provision "shell", path: "./create_users"
#  config.vm.provision "shell", path: "./change_ssh_config"

#  config.vm.provision "shell", inline: <<-SHELL
#  SHELL

#  config.vm.provision :ansible do |ansible|
#    ansible.playbook = "provisioning/elk.yml"
#  end




end
