# -*- mode: ruby -*-
# vi: set ft=ruby :
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  N = 2
  LAWP_IP = []
  HAPROXY_BE_SERVERS = []

  (1..N).each do |lapw_id|
    LAWP_IP << "192.168.100.#{101+lapw_id}"
    HAPROXY_BE_SERVERS << {name: "lapw#{lapw_id}", address: "192.168.100.#{101+lapw_id}:80"}
  end
   
  # Nodo DB 
  config.vm.define :mysql do |mysql| 
    mysql.vm.box = "ubuntu/trusty64"
    mysql.vm.hostname = "mysql.server"
    mysql.vm.network :private_network, ip: "192.168.100.101"

    mysql.vm.provider :virtualbox do |vb|
      vb.name = "VMmysql"
      vb.memory = 512
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      vb.customize ["modifyvm", :id, "--ioapic", "on"]
      vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    end

    mysql.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/playbook.yml"
      ansible.limit = 'all'
      ansible.sudo = true
      ansible.extra_vars = {lawp_ip: LAWP_IP}
      ansible.verbose = "-v"
    end
  end

  # Nodo apache, php, wordpress
  (1..N).each do |lapw_id|
    config.vm.define "lapw#{lapw_id}" do |lapw|
      lapw.vm.box = "ubuntu/trusty64"
      lapw.vm.hostname = "lapw#{lapw_id}"
      lapw.vm.network "private_network", ip: "192.168.100.#{101+lapw_id}"
      lapw.vm.synced_folder "data/", "/var/www/", owner: "www-data", group: "www-data"

      lapw.vm.provider :virtualbox do |vb|
        vb.name = "VMlapw#{lapw_id}"
        vb.memory = 512
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--ioapic", "on"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
      end
      # Only execute once the Ansible provisioner,
      # when all the machines are up and ready.
      #if lapw_id == N
        lapw.vm.provision "ansible" do |ansible|
          ansible.playbook = "ansible/playbook.yml"
          ansible.limit = 'all'
          ansible.sudo = true
          ansible.groups = {"lapw" => ["lapw#{lapw_id}"]}
          # ansible.verbose = "-v"
        end
      #end
    end
  end 

  # Nodo Balancer
  config.vm.define :haproxy do |haproxy| 
    haproxy.vm.box = "ubuntu/trusty64"
    haproxy.vm.hostname = "haproxy"
    haproxy.vm.network :private_network, ip: "192.168.100.200"

    haproxy.vm.provider :virtualbox do |vb|
      vb.name = "VMhaproxy"
      vb.memory = 512
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      vb.customize ["modifyvm", :id, "--ioapic", "on"]
      vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    end

    haproxy.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/playbook.yml"
      ansible.limit = 'all'
      ansible.sudo = true
      ansible.extra_vars = {haproxy_backend_servers: HAPROXY_BE_SERVERS}
      ansible.verbose = "-v"
    end
  end
end
