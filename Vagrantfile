# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  config.vm.define "venus" do |slave|
    slave.vm.hostname = "venus.sistema.test"
    slave.vm.network "private_network", ip: "192.168.57.102"
    slave.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get upgrade -y

      apt-get install -y bind9 bind9utils
    SHELL
  end

  config.vm.define "tierra" do |master|
    master.vm.synced_folder "./config", "/home/vagrant/shared" 
    master.vm.hostname = "tierra.sistema.test"
    master.vm.network "private_network", ip: "192.168.57.103"
    master.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get upgrade -y

    apt-get install -y bind9 bind9utils
  SHELL

  #vagrant provision master --provision-with config
  master.vm.provision "shell", name: "config", inline: <<-SHELL
    cp /vagrant/config/named /etc/default
    cp /vagrant/config/named.conf.* /etc/bind
    cp /vagrant/config/deaw.test.dns /var/lib/bind
    cp /vagrant/config/192.168.57.dns /var/lib/bind
    systemctl restart named
  SHELL
  end

end
