# -*- mode: ruby -*-
# vim: set ft=ruby :

ENV["LC_ALL"] = "en_US.UTF-8"
ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'


MACHINES = {
  :inetRouter1 => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "inetRouter1",
        :net => [   
                    #ip, adpter, netmask, virtualbox__intnet
                    ["192.168.255.1", 2, "255.255.255.252",  "router-net1"], 
                    ["192.168.56.10", 8, "255.255.255.0"],
                ],
        :node_port_fwd => {
                          }
                
  },

  :inetRouter2 => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "inetRouter2",
        :net => [   
                    #ip, adpter, netmask, virtualbox__intnet
                    ["192.168.255.5", 2, "255.255.255.252",  "router-net2"],
                    ["192.168.56.11", 8, "255.255.255.0"],
                ],
        :node_port_fwd => {
                  :http => {
                        :guest => 8085,
                        :host => 8085
                },
        }      

  },

  :centralRouter => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "centralRouter",
        :net => [
                   ["192.168.255.2",  2, "255.255.255.252",  "router-net1"],
                   ["192.168.255.6",  3, "255.255.255.252",  "router-net2"],
                   ["192.168.0.1",    4, "255.255.255.240",  "serv-net"],
                   ["192.168.56.12",  8, "255.255.255.0"],
                ],
        :node_port_fwd => {
                          }
  },

  :centralServer => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "centralServer",
        :net => [
                   ["192.168.0.2",    2, "255.255.255.240",  "serv-net"],
                   ["192.168.56.13",  8, "255.255.255.0"],
                ],
        :node_port_fwd => {
                          },
  },

}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxconfig[:vm_name]
      
      box.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 1
        
       end
   
       boxconfig[:net].each do |ipconf|
        box.vm.network("private_network", ip: ipconf[0], adapter: ipconf[1], netmask: ipconf[2], virtualbox__intnet: ipconf[3])
      end


      boxconfig[:node_port_fwd].each do |port, port_config|
        box.vm.network "forwarded_port", guest: port_config[:guest], host: port_config[:host]
      end

      if boxconfig.key?(:public)
        box.vm.network "public_network", boxconfig[:public]
      end

      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
      SHELL

   #   Запуск ansible-playbook
     if boxconfig[:vm_name] == "centralServer"
        box.vm.provision "ansible" do |ansible|
         ansible.playbook = "ansible/provision.yaml"
         ansible.inventory_path = "ansible/hosts"
         ansible.host_key_checking = "false"
         ansible.limit = "all"
        end
      end
 
    end
  end
end
