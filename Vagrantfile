# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  
  config.vm.define "vpnserver", primary: true do |s|
    s.vm.box = "centos/7"
    s.vm.provider :virtualbox do |sb|
        sb.customize ["modifyvm", :id, "--memory", "1024"]
        sb.customize ["modifyvm", :id, "--cpus", "1"]
    end
    s.vm.hostname = 'vpnserver'
    s.vm.network "private_network", ip: "192.168.20.10"
    s.vm.network "private_network", ip: "10.200.254.2"
    s.vm.provision "shell", inline: <<-SHELL
	sudo echo -e "[main]\ndns=none" > /etc/NetworkManager/conf.d/no-dns.conf
	sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
	sudo yum install epel-release -y
	sudo yum install wget ifconfig openvpn net-tools unzip -y
	sudo mkdir /etc/openvpn/keys
	sudo cd /etc/openvpn/keys
	wget https://github.com/OpenVPN/easy-rsa/archive/master.zip
	sudo unzip master.zip
	sudo setenforce 0
	sudo cp /vagrant/server.conf /etc/openvpn/
	SHELL
#    s.vm.provision "ansible_local" do |ansiblesrv|
#    ansiblesrv.playbook = "ansible/ipaserver.yml"
 #   ansiblecl.playbook = "ansible/ipaclient.yml"
    end
 end
