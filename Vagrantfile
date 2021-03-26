require 'fileutils'

NODES = [
  { :hostname => 'node1',      :ip => '192.168.50.101', :box => 'ubuntu/bionic64', :cpu => 1, :memory => 1200 },
  { :hostname => 'node2',   :ip => '192.168.50.102', :box => 'ubuntu/bionic64', :cpu => 4, :memory => 1200 },
  { :hostname => 'node3',   :ip => '192.168.50.103', :box => 'ubuntu/bionic64', :cpu => 4, :memory => 1200 },

]

Vagrant.configure("2") do |config|
  config.vm.network "forwarded_port", guest: 80, host: 7777
  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  config.ssh.insert_key = true


  NODES.each do |node|
    config.vm.define node[:hostname] do |nodeconfig|
      nodeconfig.vm.box = node[:box]
      nodeconfig.vm.hostname = node[:hostname]
      nodeconfig.vm.network :private_network, ip: node[:ip]
      nodeconfig.vm.synced_folder ".", "/vagrant"
      nodeconfig.vm.provision :shell, :inline => "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config; sudo systemctl restart sshd;", run: "once"
      nodeconfig.vm.provider :virtualbox do |vb|
        vb.memory = node[:memory]
        vb.cpus = node[:cpu]
        vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"] 
      end
      config.vm.provision "shell" do |s|
        ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
        s.inline = <<-SHELL
          echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
          echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
        SHELL
      end
    end
  end
end
