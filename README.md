
#VAGRANT MASTER SLAVE EJERCICIO

Primero configuramos el `Vagrantfile`:

```ruby
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
Luego configuramos cada uno de los archivos necesarios para los DNS en tierra según los requerimientos que se nos pide.

/etc/default/named

```bash
#
# run resolvconf?
RESOLVCONF=no

# startup options for the server | ONLY LISTEN IPV4
OPTIONS="-u bind -4"

```

/etc/bind/named.conf.local

```bash
zone "sistema.test" {
    type master;
    file "/var/lib/bind/db.sistema.test";
};

zone "57.168.192.in-addr.arpa" {
    type master;
    file "/var/lib/bind/db.57.168.192";
};

```

/etc/bind/named.conf.options

```bash
options {
    directory "/var/cache/bind";
    listen-on { 192.168.57.103; };
    listen-on-v6 { none; };
    dnssec-validation yes;

    allow-recursion { trusted; };
    forwarders {
        208.67.222.222;
    };
    forward only;
    allow-query { trusted; };
};

acl "trusted" {
    127.0.0.1;
    192.168.57.0/24;
};

```
/var/lib/bind/db.sistema.test

```bash
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
mercurio   IN  A   192.168.57.101

; Alias
ns1     IN  CNAME   tierra.sistema.test.
ns2     IN  CNAME   venus.sistema.test.
mail    IN  CNAME   marte.sistema.test.

```

/var/lib/bind/db.57.168.192

```bash
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
101     IN  PTR mercurio.sistema.test.

```

Configuramos este archivo del slave para enlazarlos
/etc/bind/named.conf.local

```bash
zone "sistema.test" {
    type master;
    file "/var/lib/bind/db.sistema.test";
};

zone "57.168.192.in-addr.arpa" {
    type master;
    file "/var/lib/bind/db.57.168.192";
};

```

Traemos los archivos al repositorio local para posteriormente enlazarlos en el provision para que se implementen automáticamente sin tener que hacerlo de forma manual. Por ejemplo, podemos copiar los archivos desde la máquina a la carpeta de nuestro repositorio local:

cp {archivoDeseado} ./vagrant/{carpetaDestino}

Agregamos a la configuración de tierra en el Vagrantfile estas líneas para provisionar cuando queramos automáticamente todos los ficheros que habíamos modificado anteriormente:

```ruby
#vagrant provision tierra --provision-with config
master.vm.provision "shell", name: "config", inline: <<-SHELL
  cp /vagrant/config_master/named /etc/default
  cp /vagrant/config_master/named.conf /etc/bind
  cp /vagrant/config_master/named.conf.* /etc/bind
  cp /vagrant/config_master/db.sistema.test /var/lib/bind
  cp /vagrant/config_master/db.57.168.192 /var/lib/bind
  sudo systemctl restart named
SHELL

```

Finalmente, el archivo Vagrantfile tendría que quedar tal que así:

```ruby
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

    #vagrant provision venus --provision-with config
    slave.vm.provision "shell", name: "config", inline: <<-SHELL
      cp /vagrant/config_slave/named.conf.local /etc/bind
      systemctl restart named
    SHELL
  end

  config.vm.define "tierra" do |master|
    master.vm.hostname = "tierra.sistema.test"
    master.vm.network "private_network", ip: "192.168.57.103"
    master.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get upgrade -y
      apt-get install -y bind9 bind9utils
    SHELL

    #vagrant provision tierra --provision-with config
    master.vm.provision "shell", name: "config", inline: <<-SHELL
      cp /vagrant/config_master/named /etc/default
      cp /vagrant/config_master/named.conf /etc/bind
      cp /vagrant/config_master/named.conf.* /etc/bind
      cp /vagrant/config_master/db.sistema.test /var/lib/bind
      cp /vagrant/config_master/db.57.168.192 /var/lib/bind
      sudo systemctl restart named
    SHELL
  end
end

```
