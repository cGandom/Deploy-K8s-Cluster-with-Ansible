# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 2.0.0"

Vagrant.configure("2") do |config|
  control_plane = 1
  worker = 3

  config.vm.box = "bento/ubuntu-20.04"
  config.vm.provider "virtualbox"
  config.vm.box_check_update = false

  (1..(control_plane + worker)).each do |i|
    name  = (i <= control_plane) ? "c" : "w"
    id    = (i <= control_plane) ? i : (i - control_plane)  
    
    config.vm.define "k8s-#{name}-#{id}" do |node|
      node.vm.hostname = "k8s-#{name}-#{id}"
      ip_addr = "192.168.33.#{10 + i}"
      node.vm.network :private_network, ip: "#{ip_addr}", auto_config: true

      node.vm.provider :virtualbox do |vb|
        vb.name = "#{node.vm.hostname}"
        vb.gui = false
        vb.cpus = 2
        vb.memory = 2048
      end
      
    end
  end
  
  config.vm.provision "shell", inline: <<-SHELL
    echo -e "vagrant\nvagrant" | passwd root
    sed -in 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
    sed -in 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    service ssh restart
  SHELL
end
