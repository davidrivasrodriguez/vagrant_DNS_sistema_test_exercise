# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  config.vm.define "venus" do |slave|
    slave.vm.hostname = "venus.sistema.test "
    slave.vm.network "private_network", ip: "192.168.57.102"
    slave.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get upgrade -y

      apt-get install -y bind9 bind9utils
    SHELL
  end

  config.vm.define "tierra" do |master|
    master.vm.hostname = "tierra.sistema.test "
    master.vm.network "private_network", ip: "192.168.57.103"
    master.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get upgrade -y

    apt-get install -y bind9 bind9utils
  SHELL
  end

end
