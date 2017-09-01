# -*- mode: ruby -*-
# vi: set ft=ruby :

$base_script = <<SCRIPT
# Update apt and get dependencies
sudo apt-get update
sudo apt-get install -y unzip curl wget vim

# Install Docker
sudo curl -sSL https://get.docker.com/ | sh
SCRIPT

$consul_script = <<SCRIPT
# Download Consul
echo Fetching Consul...
cd /tmp/
wget https://releases.hashicorp.com/consul/0.9.2/consul_0.9.2_linux_amd64.zip -O consul.zip

echo Installing Consul...
unzip consul.zip
sudo chmod +x consul
sudo mv consul /usr/bin/consul

sudo mkdir /etc/consul.d
sudo chmod a+w /etc/consul.d

cat << EOF | sudo tee /etc/init/consul.conf
description "Consul"

start on runlevel [2345]
stop on runlevel [!2345]

respawn
respawn limit 10 5

exec consul agent -config-dir /etc/consul.d
EOF

sudo apt-get install -y dnsmasq
cat << EOF | sudo tee /etc/dnsmasq.d/10-consul
server=/consul/127.0.0.1#8600
EOF

sudo service dnsmasq restart
SCRIPT

$nomad_script = <<SCRIPT
# Download Nomad
echo Fetching Nomad...
cd /tmp/
wget https://releases.hashicorp.com/nomad/0.6.2/nomad_0.6.2_linux_amd64.zip -O nomad.zip

echo Installing Nomad...
unzip nomad.zip
sudo chmod +x nomad
sudo mv nomad /usr/bin/nomad

sudo mkdir /etc/nomad.d
sudo chmod a+w /etc/nomad.d

cat << EOF | sudo tee /etc/init/nomad.conf
description "Nomad"

start on runlevel [2345]
stop on runlevel [!2345]

respawn
respawn limit 10 5

exec nomad agent -config /etc/nomad.d
EOF
SCRIPT

Vagrant.configure(2) do |config|
  config.vm.box = "puphpet/ubuntu1404-x64"
  config.vm.provision "shell", inline: $base_script, privileged: true
  config.vm.provision "shell", inline: $consul_script, privileged: true
  config.vm.provision "shell", inline: $nomad_script, privileged: true

  # config.vm.network "forwarded_port", guest: 4646, host: 4646
  # config.vm.network "forwarded_port", guest: 4647, host: 4647
  # config.vm.network "forwarded_port", guest: 4648, host: 4648

  # Increase memory for Virtualbox
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
  end

  config.vm.define "master-eu-east-1" do |c|
    c.vm.hostname = "master.eu-east-1.example.com"
    c.vm.network "private_network", ip: "192.168.50.10"

server_script = <<SCRIPT
cat << EOF | sudo tee /etc/consul.d/server.json
{
  "node_name": "master-eu-east-1",
  "data_dir": "/tmp/consul",
  "log_level": "DEBUG",
  "server": true,
  "bootstrap_expect": 1,
  "advertise_addr": "192.168.50.10",
  "bind_addr": "0.0.0.0",
  "client_addr": "0.0.0.0",
  "datacenter": "eu-east-1",
  "ui": true
}
EOF

sudo service consul start

cat << EOF | sudo tee /etc/nomad.d/server.hcl
name = "master-eu-east-1"
log_level = "DEBUG"
data_dir = "/tmp/nomad"
addresses {
  http = "0.0.0.0"
  rpc = "192.168.50.10"
  serf = "192.168.50.10"
}
advertise {
  rpc = "192.168.50.10:4647"
}
datacenter = "eu-east-1"
region = "eu"
http_api_response_headers { Access-Control-Allow-Origin = "*" }
consul {}
server {
  enabled = true
  bootstrap_expect = 1
}
EOF

sudo service nomad start

SCRIPT

    c.vm.provision "shell", inline: server_script, privileged: false
  end

  config.vm.define "slave-1-eu-east-1" do |c|
    c.vm.hostname = "slave-1.eu-east-1.example.com"
    c.vm.network "private_network", ip: "192.168.50.11"

client_script = <<SCRIPT
cat << EOF | sudo tee /etc/consul.d/client.json
{
  "node_name": "slave-1-eu-east-1",
  "data_dir": "/tmp/consul",
  "log_level": "DEBUG",
  "advertise_addr": "192.168.50.11",
  "bind_addr": "0.0.0.0",
  "client_addr": "0.0.0.0",
  "start_join": ["192.168.50.10"],
  "datacenter": "eu-east-1"
}
EOF

sudo service consul start

cat << EOF | sudo tee /etc/nomad.d/client.hcl
name = "slave-1-eu-east-1"
log_level = "DEBUG"
data_dir = "/tmp/nomad"
addresses {
  http = "0.0.0.0"
}
advertise {
  rpc = "192.168.50.11:4647"
}
datacenter = "eu-east-1"
region = "eu"
http_api_response_headers { Access-Control-Allow-Origin = "*" }
consul {}
client {
  enabled = true
  network_interface = "eth1"
}
EOF

sudo service nomad start

SCRIPT

    c.vm.provision "shell", inline: client_script, privileged: false
  end

  config.vm.define "slave-2-eu-east-1" do |c|
    c.vm.hostname = "slave-2.eu-east-1.example.com"
    c.vm.network "private_network", ip: "192.168.50.12"

client_script = <<SCRIPT
cat << EOF | sudo tee /etc/consul.d/client.json
{
  "node_name": "slave-2-eu-east-1",
  "data_dir": "/tmp/consul",
  "log_level": "DEBUG",
  "advertise_addr": "192.168.50.12",
  "bind_addr": "0.0.0.0",
  "client_addr": "0.0.0.0",
  "start_join": ["192.168.50.10"],
  "datacenter": "eu-east-1"
}
EOF

sudo service consul start

cat << EOF | sudo tee /etc/nomad.d/client.hcl
name = "slave-2-eu-east-1"
log_level = "DEBUG"
data_dir = "/tmp/nomad"
addresses {
  http = "0.0.0.0"
}
advertise {
  rpc = "192.168.50.12:4647"
}
datacenter = "eu-east-1"
region = "eu"
http_api_response_headers { Access-Control-Allow-Origin = "*" }
consul {}
client {
  enabled = true
  network_interface = "eth1"
}
EOF

sudo service nomad start

SCRIPT

    c.vm.provision "shell", inline: client_script, privileged: false
  end

  config.vm.define "slave-3-eu-east-1" do |c|
    c.vm.hostname = "slave-3.eu-east-1.example.com"
    c.vm.network "private_network", ip: "192.168.50.13"

client_script = <<SCRIPT
cat << EOF | sudo tee /etc/consul.d/client.json
{
  "node_name": "slave-3-eu-east-1",
  "data_dir": "/tmp/consul",
  "log_level": "DEBUG",
  "advertise_addr": "192.168.50.13",
  "bind_addr": "0.0.0.0",
  "client_addr": "0.0.0.0",
  "start_join": ["192.168.50.10"],
  "datacenter": "eu-east-1"
}
EOF

sudo service consul start

cat << EOF | sudo tee /etc/nomad.d/client.hcl
name = "slave-3-eu-east-1"
log_level = "DEBUG"
data_dir = "/tmp/nomad"
addresses {
  http = "0.0.0.0"
}
advertise {
  rpc = "192.168.50.13:4647"
}
datacenter = "eu-east-1"
region = "eu"
http_api_response_headers { Access-Control-Allow-Origin = "*" }
consul {}
client {
  enabled = true
  network_interface = "eth1"
}
EOF

sudo service nomad start

SCRIPT

    c.vm.provision "shell", inline: client_script, privileged: false
  end

  config.vm.define "master-eu-west-1" do |c|
    c.vm.hostname = "master.eu-west-1.example.com"
    c.vm.network "private_network", ip: "192.168.60.10"

server_script = <<SCRIPT
cat << EOF | sudo tee /etc/consul.d/server.json
{
  "node_name": "master-eu-west-1",
  "data_dir": "/tmp/consul",
  "log_level": "DEBUG",
  "server": true,
  "bootstrap_expect": 1,
  "advertise_addr": "192.168.60.10",
  "bind_addr": "0.0.0.0",
  "client_addr": "0.0.0.0",
  "start_join_wan": ["192.168.50.10"],
  "datacenter": "eu-west-1",
  "ui": true
}
EOF

sudo service consul start

cat << EOF | sudo tee /etc/nomad.d/server.hcl
name = "master-eu-west-1"
log_level = "DEBUG"
data_dir = "/tmp/nomad"
addresses {
  http = "0.0.0.0"
  rpc = "192.168.60.10"
  serf = "192.168.60.10"
}
advertise {
  rpc = "192.168.60.10:4647"
}
datacenter = "eu-west-1"
region = "eu"
http_api_response_headers { Access-Control-Allow-Origin = "*" }
consul {}
server {
  enabled = true
  bootstrap_expect = 1
}
EOF

sudo service nomad start

SCRIPT

    c.vm.provision "shell", inline: server_script, privileged: false
  end

  config.vm.define "slave-1-eu-west-1" do |c|
    c.vm.hostname = "slave-1.eu-west-1.example.com"
    c.vm.network "private_network", ip: "192.168.60.11"

client_script = <<SCRIPT
cat << EOF | sudo tee /etc/consul.d/client.json
{
  "node_name": "slave-1-eu-west-1",
  "data_dir": "/tmp/consul",
  "log_level": "DEBUG",
  "advertise_addr": "192.168.60.11",
  "bind_addr": "0.0.0.0",
  "client_addr": "0.0.0.0",
  "start_join": ["192.168.60.10"],
  "datacenter": "eu-west-1"
}
EOF

sudo service consul start

cat << EOF | sudo tee /etc/nomad.d/client.hcl
name = "slave-1-eu-west-1"
log_level = "DEBUG"
data_dir = "/tmp/nomad"
addresses {
  http = "0.0.0.0"
}
advertise {
  rpc = "192.168.60.11:4647"
}
datacenter = "eu-west-1"
region = "eu"
http_api_response_headers { Access-Control-Allow-Origin = "*" }
consul {}
client {
  enabled = true
  network_interface = "eth1"
}
EOF

sudo service nomad start

SCRIPT

    c.vm.provision "shell", inline: client_script, privileged: false
  end

  config.vm.define "slave-2-eu-west-1" do |c|
    c.vm.hostname = "slave-2.eu-west-1.example.com"
    c.vm.network "private_network", ip: "192.168.60.12"

client_script = <<SCRIPT
cat << EOF | sudo tee /etc/consul.d/client.json
{
  "node_name": "slave-2-eu-west-1",
  "data_dir": "/tmp/consul",
  "log_level": "DEBUG",
  "advertise_addr": "192.168.60.12",
  "bind_addr": "0.0.0.0",
  "client_addr": "0.0.0.0",
  "start_join": ["192.168.60.10"],
  "datacenter": "eu-west-1"
}
EOF

sudo service consul start

cat << EOF | sudo tee /etc/nomad.d/client.hcl
name = "slave-2-eu-west-1"
log_level = "DEBUG"
data_dir = "/tmp/nomad"
addresses {
  http = "0.0.0.0"
}
advertise {
  rpc = "192.168.60.12:4647"
}
datacenter = "eu-west-1"
region = "eu"
http_api_response_headers { Access-Control-Allow-Origin = "*" }
consul {}
client {
  enabled = true
  network_interface = "eth1"
}
EOF

sudo service nomad start

SCRIPT

    c.vm.provision "shell", inline: client_script, privileged: false
  end

  config.vm.define "slave-3-eu-west-1" do |c|
    c.vm.hostname = "slave-3.eu-west-1.example.com"
    c.vm.network "private_network", ip: "192.168.60.13"

client_script = <<SCRIPT
cat << EOF | sudo tee /etc/consul.d/client.json
{
  "node_name": "slave-3-eu-west-1",
  "data_dir": "/tmp/consul",
  "log_level": "DEBUG",
  "advertise_addr": "192.168.60.13",
  "bind_addr": "0.0.0.0",
  "client_addr": "0.0.0.0",
  "start_join": ["192.168.60.10"],
  "datacenter": "eu-west-1"
}
EOF

sudo service consul start

cat << EOF | sudo tee /etc/nomad.d/client.hcl
name = "slave-3-eu-west-1"
log_level = "DEBUG"
data_dir = "/tmp/nomad"
addresses {
  http = "0.0.0.0"
}
advertise {
  rpc = "192.168.60.13:4647"
}
datacenter = "eu-west-1"
region = "eu"
http_api_response_headers { Access-Control-Allow-Origin = "*" }
consul {}
client {
  enabled = true
  network_interface = "eth1"
}
EOF

sudo service nomad start

SCRIPT

    c.vm.provision "shell", inline: client_script, privileged: false
  end

  config.vm.define "master-us-east-1" do |c|
    c.vm.hostname = "master.us-east-1.example.com"
    c.vm.network "private_network", ip: "192.168.70.10"

server_script = <<SCRIPT
cat << EOF | sudo tee /etc/consul.d/server.json
{
  "node_name": "master-us-east-1",
  "data_dir": "/tmp/consul",
  "log_level": "DEBUG",
  "server": true,
  "bootstrap_expect": 1,
  "advertise_addr": "192.168.70.10",
  "bind_addr": "0.0.0.0",
  "client_addr": "0.0.0.0",
  "start_join_wan": ["192.168.50.10"],
  "datacenter": "us-east-1",
  "ui": true
}
EOF

sudo service consul start

cat << EOF | sudo tee /etc/nomad.d/server.hcl
name = "master-us-east-1"
log_level = "DEBUG"
data_dir = "/tmp/nomad"
addresses {
  http = "0.0.0.0"
  rpc = "192.168.70.10"
  serf = "192.168.70.10"
}
advertise {
  rpc = "192.168.70.10:4647"
}
datacenter = "us-east-1"
region = "us"
http_api_response_headers { Access-Control-Allow-Origin = "*" }
consul {}
server {
  enabled = true
  bootstrap_expect = 1
}
EOF

sudo service nomad start

SCRIPT

    c.vm.provision "shell", inline: server_script, privileged: false
  end

  config.vm.define "slave-1-us-east-1" do |c|
    c.vm.hostname = "slave-1.us-east-1.example.com"
    c.vm.network "private_network", ip: "192.168.70.11"

client_script = <<SCRIPT
cat << EOF | sudo tee /etc/consul.d/client.json
{
  "node_name": "slave-1-us-east-1",
  "data_dir": "/tmp/consul",
  "log_level": "DEBUG",
  "advertise_addr": "192.168.70.11",
  "bind_addr": "0.0.0.0",
  "client_addr": "0.0.0.0",
  "start_join": ["192.168.70.10"],
  "datacenter": "us-east-1"
}
EOF

sudo service consul start

cat << EOF | sudo tee /etc/nomad.d/client.hcl
name = "slave-1-us-east-1"
log_level = "DEBUG"
data_dir = "/tmp/nomad"
addresses {
  http = "0.0.0.0"
}
advertise {
  rpc = "192.168.70.11:4647"
}
datacenter = "us-east-1"
region = "us"
http_api_response_headers { Access-Control-Allow-Origin = "*" }
consul {}
client {
  enabled = true
  network_interface = "eth1"
}
EOF

sudo service nomad start

SCRIPT

    c.vm.provision "shell", inline: client_script, privileged: false
  end

  config.vm.define "slave-2-us-east-1" do |c|
    c.vm.hostname = "slave-2.us-east-1.example.com"
    c.vm.network "private_network", ip: "192.168.70.12"

client_script = <<SCRIPT
cat << EOF | sudo tee /etc/consul.d/client.json
{
  "node_name": "slave-2-us-east-1",
  "data_dir": "/tmp/consul",
  "log_level": "DEBUG",
  "advertise_addr": "192.168.70.12",
  "bind_addr": "0.0.0.0",
  "client_addr": "0.0.0.0",
  "start_join": ["192.168.70.10"],
  "datacenter": "us-east-1"
}
EOF

sudo service consul start

cat << EOF | sudo tee /etc/nomad.d/client.hcl
name = "slave-2-us-east-1"
log_level = "DEBUG"
data_dir = "/tmp/nomad"
addresses {
  http = "0.0.0.0"
}
advertise {
  rpc = "192.168.70.12:4647"
}
datacenter = "us-east-1"
region = "us"
http_api_response_headers { Access-Control-Allow-Origin = "*" }
consul {}
client {
  enabled = true
  network_interface = "eth1"
}
EOF

sudo service nomad start

SCRIPT

    c.vm.provision "shell", inline: client_script, privileged: false
  end

  config.vm.define "slave-3-us-east-1" do |c|
    c.vm.hostname = "slave-3.us-east-1.example.com"
    c.vm.network "private_network", ip: "192.168.70.13"

client_script = <<SCRIPT
cat << EOF | sudo tee /etc/consul.d/client.json
{
  "node_name": "slave-3-us-east-1",
  "data_dir": "/tmp/consul",
  "log_level": "DEBUG",
  "advertise_addr": "192.168.70.13",
  "bind_addr": "0.0.0.0",
  "client_addr": "0.0.0.0",
  "start_join": ["192.168.70.10"],
  "datacenter": "us-east-1"
}
EOF

sudo service consul start

cat << EOF | sudo tee /etc/nomad.d/client.hcl
name = "slave-3-us-east-1"
log_level = "DEBUG"
data_dir = "/tmp/nomad"
addresses {
  http = "0.0.0.0"
}
advertise {
  rpc = "192.168.70.13:4647"
}
datacenter = "us-east-1"
region = "us"
http_api_response_headers { Access-Control-Allow-Origin = "*" }
consul {}
client {
  enabled = true
  network_interface = "eth1"
}
EOF

sudo service nomad start

SCRIPT

    c.vm.provision "shell", inline: client_script, privileged: false
  end
end
