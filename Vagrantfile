Vagrant.configure(2) do |config|

 server_configs = [
   {"hostname" => "ci", "ip" => "192.168.30.1", "port" => 1032, "memory_size" => "1024", "cpus" => 2, "execute_script" => true},
   {"hostname" => "web", "ip" => "192.168.30.2", "port" => 1033, "memory_size" => "1024", "cpus" => 2, "execute_script" => false},
   {"hostname" => "db", "ip" => "192.168.30.3", "port" => 1034, "memory_size" => "1024", "cpus" => 2, "execute_script" => false},
 ]

 $script = "
   sudo yum update -y --disablerepo=\* --enablerepo=base,updates ca-certificates
   sudo yum install -y epel-release
   sudo yum install -y --enablerepo=epel sshpass git
   sudo yum install -y --enablerepo=epel-testing ansible-2.4.2.0
   cd ansible-playbook
   cp vagrant/insecure_private_key /home/vagrant/.ssh/id_rsa
   chmod -R og-rwx /home/vagrant/.ssh
   chown -R vagrant.vagrant /home/vagrant/.ssh
 "

 server_configs.each do |server_config|
   config.vm.define "#{server_config['hostname']}" do |server|
     server.vm.hostname = server_config['hostname']
     server.vm.box = "bento/centos-7.4"
     server.vm.network :private_network, ip: server_config['ip']
     server.vm.network :forwarded_port, guest: 22, host: server_config['port'], id: "ssh"
     server.vm.provider "virtualbox" do |v|
       v.customize ["modifyvm", :id, "--cpus", server_config['cpus']]
       v.customize ["modifyvm", :id, "--memory", server_config['memory_size']]
       v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
       v.customize ["setextradata", :id, "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled", 0]
     end
     server.vm.synced_folder ".", "/home/vagrant/ansible-playbook", owner: "root", group: "root", :create => true, :mount_options => ["fmode=777", "dmode=777"]
     if server_config['execute_script'] then
       server.vm.provision :shell, inline: $script
     end
     server.ssh.private_key_path = "./vagrant/insecure_private_key"
     server.ssh.insert_key = false
   end
 end
end
