hosts = [{ name: 'ceph-node1', ip: '192.168.57.101', port: '2222', disk1: 'ceph-node1disk1.vdi', disk2: 'ceph-node1disk2.vdi', disk3: 'ceph-node1disk3.vdi' },
         { name: 'ceph-node2', ip: '192.168.57.102', port: '2223', disk1: 'ceph-node2disk1.vdi', disk2: 'ceph-node2disk2.vdi', disk3: 'ceph-node2disk3.vdi' },
         { name: 'ceph-node3', ip: '192.168.57.103', port: '2224', disk1: 'ceph-node3disk1.vdi', disk2: 'ceph-node3disk2.vdi', disk3: 'ceph-node3disk3.vdi' }]

Vagrant.configure("2") do |config|
  # Use the same key for each machine
  config.ssh.insert_key = false

  config.vm.provider :virtualbox do |v|
    v.customize ['modifyvm', :id, '--memory', 2048]
    v.customize ['modifyvm', :id, '--cpus', 2]
  end

  hosts.each do |host|
    config.vm.define host[:name] do |node|
      node.vm.hostname = host[:name]
      #node.vm.box = 'centos/7'
      node.vm.box = 'geerlingguy/ubuntu1604'
      node.vm.network :forwarded_port, guest: 22, host: host[:port], id: 'ssh'
      node.vm.network :private_network, ip: host[:ip]
      node.vm.provider :virtualbox do |vb|
        vb.name = host[:name]
        if not File.exists?(host[:disk1])
          vb.customize ['storagectl', :id, '--name', 'SATA Controller', '--add', 'sata', '--portcount', 4]
          vb.customize ['createhd', '--filename', host[:disk1], '--size', 10 * 1024]
          vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', "1", '--device', 0, '--type', 'hdd', '--medium', host[:disk1]]
        end
        if not File.exists?(host[:disk2])
          vb.customize ['createhd', '--filename', host[:disk2], '--size', 10 * 1024]
          vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', "2", '--device', 0, '--type', 'hdd', '--medium', host[:disk2]]
        end
        if not File.exists?(host[:disk3])
          vb.customize ['createhd', '--filename', host[:disk3], '--size', 10 * 1024]
          vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', "3", '--device', 0, '--type', 'hdd', '--medium', host[:disk3]]
        end
      end

      # Only execute once the Ansible provisioner,
      # when all the machines are up and ready.
      if host[:name] == 'ceph-node3'
        config.vm.provision "ansible" do |ansible|
          ansible.limit = 'all'
          ansible.playbook = "ansible/playbook.yml"
          ansible.verbose = "-v"
          ansible.inventory_path = "ansible/hosts"
        end
      end
    end
  end
end
