# -*- mode: ruby -*-
# vi: set ft=ruby :

# Setup Van het netwerk
# Loadsbalancer (lb)      192.168.50.0/24
# Frontend servers        192.168.60.0/24
# Backend servers         192.168.150.0/24
# Database server (db)    192.168.200.0/24


Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-18.04"
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

   N_be=2 #Aantal hosts in backend laag
   N_fe=2    #Aantal hosts in frontend laag

   LB_server = '192.168.50.1'
   DB_server = '192.168.200.1'

   Loadsbalancer_subnet = '192.168.50.0/24'
   FrontendServers_subnet = '192.168.60.0/24'
   BackendServers_subnet = '192.168.150.0/24'
   DatabaseServer_subnet = '192.168.200.0/24'

# DB server
config.vm.define "db" do |db|
   db.vm.box = "bento/ubuntu-18.04"
   db.vm.network "private_network", ip: DB_server
   db.vm.provision "shell", path: "./create_users"
   db.vm.provision "shell", path: "./change_ssh_config"

   db.vm.provision :ansible do |ansible_DB|
     ansible_DB.groups = {
         "dbservers" => ["db"]
       }
     ansible_DB.playbook = "playbooks/db.yml"
     ansible_DB.extra_vars = {
                "BackendServers_subnet" => BackendServers_subnet
    }
    end
 end

#BackendServers
(1..N_be).each do |machine_id_BE|
  config.vm.define "backend-#{machine_id_BE}" do |backend|
    backend.vm.box = "bento/ubuntu-18.04"
    backend.vm.network "private_network", ip: "192.168.150.#{0+machine_id_BE}"

    #Parallel provisioning
    if machine_id_BE == N_be
      backend.vm.provision "shell", path: "./create_users"
      backend.vm.provision "shell", path: "./change_ssh_config"
      backend.vm.provision :ansible do |ansible_be|
        ansible_be.limit = "all"
        ansible_be.playbook = "playbooks/backend.yml"

        BackendServers = []
        #Build list of front end servers for setting groups
        (1..N_be).each do |m_id_BE|
          BackendServers << "backend-#{m_id_BE}"
        end

        ansible_be.groups = {
            "beservers" => BackendServers
          }


        ansible_be.extra_vars = {
          "frontendserver_address"=> "192.168.60.#{0+machine_id_BE}" #Voor firewall rules
        }
      end
    end
  end
end


#FrontendServers
(1..N_fe).each do |machine_id_FE|
  config.vm.define "frontend-#{machine_id_FE}" do |frontend|
    frontend.vm.box = "bento/ubuntu-18.04"
    frontend.vm.network "private_network", ip: "192.168.60.#{0+machine_id_FE}"

    #Parallel provisioning
    if machine_id_FE == N_fe
      frontend.vm.provision "shell", path: "./create_users"
      frontend.vm.provision "shell", path: "./change_ssh_config"
      frontend.vm.provision :ansible do |ansible_fe|
        ansible_fe.limit = "all"
        ansible_fe.playbook = "playbooks/frontend.yml"

        FrontendServers = []
        #Build list of front end servers for setting groups
        (1..N_fe).each do |m_id_FE|
          FrontendServers << "frontend-#{m_id_FE}"
        end

        ansible_fe.groups = {
            "feservers" => FrontendServers
          }

        ansible_fe.extra_vars = {
          "lb_server_address"=> LB_server #Voor Firewall rules
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

   lb.vm.provision :ansible do |ansible_lb|
     ansible_lb.playbook = "playbooks/lb.yml"

     ansible_lb.groups = {
         "lbservers" => ["lb"]
       }

     FrontendServers = []
     #Build list of front end servers to provision loadbalancer config
     (1..N_fe).each do |machine_id_FE|
       FrontendServers << {"name": "frontend-#{machine_id_FE}", "ip": "192.168.60.#{0+machine_id_FE}", "port": 80, "paramstring": "cookie A check"}
     end
     ansible_lb.extra_vars = {
                "haproxy_backend_servers" => FrontendServers
    }
    end
 end


end
