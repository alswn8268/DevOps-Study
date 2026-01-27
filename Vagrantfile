# -*- mode: ruby -*-
# vi: set ft=ruby :

BOX_NAME = "nobreak-labs/rocky-9"
MEMORY = "2048"
CPUS = 2

NODES = {
  "WP-WEB01" => {
    networks: [
      { ip: "192.168.56.111" }
    ]
  },
  "WP-WEB02" => {
    networks: [
      { ip: "192.168.56.112" }
    ]
  },
  "WP-NFS" => {
    networks: [
      { ip: "192.168.56.113" }
    ]
  },
  "WP-iSCSI" => {
    networks: [
      { ip: "192.168.56.114" }
    ]
  },
  "WP-DB01" => {
    networks: [
      { ip: "192.168.56.115" }
    ]
  },
  "WP-DB02" => {
    networks: [
      { ip: "192.168.56.116" }
    ]
  },
  "WP-LB" => {
    networks: [
      { ip: "192.168.56.117" }
    ]
  },
  "WP-LB02" => {
    networks: [
      { ip: "192.168.56.118" }
    ]
  },
}

Vagrant.configure("2") do |config|
  NODES.each do |node_name, node_config|
    config.vm.define node_name do |node|
      node.vm.box = BOX_NAME

      node_config[:networks].each do |network|
        node.vm.network "private_network", ip: network[:ip]
      end

      node.vm.synced_folder ".", "/vagrant", disabled: true

      node.vm.provider "virtualbox" do |vb|
        vb.memory = MEMORY
        vb.cpus = CPUS
        vb.name = node_name

        if node_name == "WP-iSCSI"
          disk_path = "iscsi-disk.vmdk"
          unless File.exist?(disk_path)
            vb.customize ["createhd", "--filename", disk_path, "--size", 10240]  # 10GB
          end

          vb.customize ["storageattach", :id,
                        "--storagectl", "VirtIO Controller",
                        "--port", 1,
                        "--device", 0,
                        "--type", "hdd",
                        "--medium", disk_path]
        end
      end

      node.vm.hostname = node_name
    end
  end
end