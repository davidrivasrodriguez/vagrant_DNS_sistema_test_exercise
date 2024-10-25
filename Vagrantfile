# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  config.vm.define "tierra" do |master|
    master.vm.hostname = "tierra.sistema.test"
    master.vm.network "private_network", ip: "192.168.57.103"
    master.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get upgrade -y

    apt-get install -y bind9 bind9utils
  SHELL

  #vagrant provision tierra --provision-with config_tierra
  master.vm.provision "shell", name: "config_tierra", inline: <<-SHELL
    cp /vagrant/config_master/named /etc/default
    cp /vagrant/config_master/named.conf /etc/bind
    cp /vagrant/config_master/named.conf.* /etc/bind
    cp /vagrant/config_master/db.sistema.test /var/lib/bind
    cp /vagrant/config_master/db.57.168.192 /var/lib/bind
    sudo systemctl restart named
  SHELL
  end

  config.vm.define "venus" do |slave|
    slave.vm.hostname = "venus.sistema.test"
    slave.vm.network "private_network", ip: "192.168.57.102"
    slave.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get upgrade -y

      apt-get install -y bind9 bind9utils
    SHELL

    #vagrant provision venus --provision-with config_venus
  slave.vm.provision "shell", name: "config_venus", inline: <<-SHELL
    cp /vagrant/config_slave/named.conf.local /etc/bind
    systemctl restart named
    SHELL
  end

end
