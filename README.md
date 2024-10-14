# vagrant_DNS_sistema_test_exercise

Primero configuramos el Vagrantfile 
```
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
  end

end

```


Luego configuramos cada uno de los archivos necesarios para los DNS

/etc/bind/named.conf.options
```
options {
    directory "/var/cache/bind";
    listen-on { 192.168.57.103; };
    listen-on-v6 { none; };
    dnssec-validation yes;

    acl "trusted" {
        127.0.0.1;
        192.168.57.0/24;
    };

    allow-recursion { trusted; };
    allow-query { trusted; };
    forwarders {
        208.67.222.222;
    };
    forward only;
};

```

/var/lib/bind/db.sistema.test
```
$TTL 7200
@   IN  SOA tierra.sistema.test. root.sistema.test. (
    2024101401  ; Serial
    3600        ; Refresh
    1800        ; Retry
    604800      ; Expire
    7200        ; Negative Cache TTL
)

@   IN  NS  tierra.sistema.test.
@   IN  NS  venus.sistema.test.

; Registros A
tierra  IN  A   192.168.57.103
venus   IN  A   192.168.57.102
marte   IN  A   192.168.57.104

; Alias
ns1     IN  CNAME   tierra.sistema.test.
ns2     IN  CNAME   venus.sistema.test.
mail    IN  CNAME   marte.sistema.test.

```


/var/lib/bind/db.57.168.192
```
$TTL 7200
@   IN  SOA tierra.sistema.test. root.sistema.test. (
    2024101401  ; Serial
    3600        ; Refresh
    1800        ; Retry
    604800      ; Expire
    7200        ; Negative Cache TTL
)

@   IN  NS  tierra.sistema.test.

; Registros PTR
103     IN  PTR tierra.sistema.test.
102     IN  PTR venus.sistema.test.
104     IN  PTR marte.sistema.test.

```

Traemos los archivos al repositorio local para posteriormente enlazarlos en el provision para que 
se implementen automaticamente sin tener que hacerlo de forma manual.
Por ejemplo podemos sincronizar una carpeta de la maquina con una de nuestro repositorio local
```
    master.vm.synced_folder "./config", "/home/vagrant/shared" 

```


Agregamos a la configuracion de tierra en el Vagrantfile estas lineas para provisionar cuando queramos
automaticamente todos los ficheros que habiamos modificado anteriormente
```
  #vagrant provision master --provision-with config
  master.vm.provision "shell", name: "config", inline: <<-SHELL
    cp /vagrant/config/named /etc/default
    cp /vagrant/config/named.conf.* /etc/bind
    cp /vagrant/config/deaw.test.dns /var/lib/bind
    cp /vagrant/config/192.168.57.dns /var/lib/bind
    systemctl restart named
  SHELL
  end

```


Finalmente el archivo Vagrantfile tendria que quedar tal que
```
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

```