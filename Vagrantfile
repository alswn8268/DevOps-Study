# -*- mode: ruby -*-
# vi: set ft=ruby :
vm_image = "nobreak-labs/rocky-9"
vm_subnet = "192.168.56."

Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "user01" do |node|
    node.vm.box = vm_image
    node.vm.provider "virtualbox" do |vb|
      vb.name = "user02"
      vb.cpus = 2
      vb.memory = 1024
    end

    node.vm.network "private_network", ip: vm_subnet + "11", nic_type: "virtio"
    node.vm.hostname = "user02"
  end
end
