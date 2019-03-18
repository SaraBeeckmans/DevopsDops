# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-18.04"

  #To make the VM show up in the VM-Fusion tool
  config.vm.provider "vmware_desktop" do |v|
    v.gui = false
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
LB_server = '192.168.50.10'

#FrontendServers
(1..N).each do |machine_id|
  config.vm.define "frontend-#{BasecountIP+machine_id}" do |frontend|
    frontend.vm.box = "bento/ubuntu-18.04"
    frontend.vm.network "private_network", ip: "192.168.100.#{BasecountIP+machine_id}"

    #Parallel provisioning
    if machine_id == N
      frontend.vm.provision "shell", path: "./create_users"
      frontend.vm.provision "shell", path: "./change_ssh_config"
      frontend.vm.provision :ansible do |ansible|
        ansible.limit = "all"
        ansible.playbook = "playbooks/frontend.yml"
        ansible.extra_vars = {
          "lb_server_address"=> LB_server
        }
      end
    end
  end
end

# Load balancer
config.vm.define "lb" do |lb|
   lb.vm.box = "bento/ubuntu-18.04"
   lb.vm.network "private_network", ip: LB_server
   lb.vm.provision "shell", path: "./create_users"
   lb.vm.provision "shell", path: "./change_ssh_config"

   lb.vm.provision :ansible do |ansible|
     ansible.playbook = "playbooks/lb.yml"
     FrontendServers = []
     #Build list of front end servers to provision loadbalancer config
     (1..N).each do |machine_id|
       FrontendServers << {"name": "frontend-#{BasecountIP+machine_id}", "ip": "192.168.100.#{BasecountIP+machine_id}", "port": 80, "paramstring": "cookie A check"}
     end
     ansible.extra_vars = {
                "haproxy_backend_servers" => FrontendServers
    }
    end
 end


end
