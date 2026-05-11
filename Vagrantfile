Vagrant.configure("2") do |config|
  nodes = {
    "control"  => "192.168.56.10",
    "node1"    => "192.168.56.11",
    "minio"    => "192.168.56.12"
  }

  bridged_ips = {
    "control"  => "192.168.1.50",
    "node1"    => "192.168.1.51",
    "minio"    => "192.168.1.52"
  }

  nodes.each do |name, host_only_ip|
    config.vm.define name do |node|
      node.vm.box = "ubuntu/jammy64"
      node.vm.hostname = name

      # Adapter 2: Host-only network (Kubernetes internal)
      node.vm.network "private_network",
        ip: host_only_ip,
        virtualbox__promiscuous_mode: "allow-all"

      # Adapter 3: Bridged network (LAN access)
      node.vm.network "public_network",
        ip: bridged_ips[name]

      # Adapter 4: MinIO ONLY — Dedicated private network (fixes pod → MinIO traffic)
      if name == "minio"
        node.vm.network "private_network",
          ip: "10.10.10.12",
          virtualbox__promiscuous_mode: "allow-all"
      end

      # VM resources
      node.vm.provider "virtualbox" do |vb|
        if name == "control"
          vb.memory = 2048
          vb.cpus = 2
        elsif name == "minio"
          vb.memory = 2048
          vb.cpus = 2
        else
          vb.memory = 1536
          vb.cpus = 1
        end
      end

      node.vm.synced_folder ".", "/vagrant", disabled: true
    end
  end
end

