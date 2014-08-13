# -*- mode: ruby -*-
# vi: set ft=ruby :
VAGRANTFILE_API_VERSION = "2"

## Cassandra cluster settings
server_count = 1
network = '192.168.2.'
first_ip = 10

servers = []
seeds = []
cassandra_tokens = []
(0..server_count-1).each do |i|
  name = 'node' + (i + 1).to_s
  ip = network + (first_ip + i).to_s
  seeds << ip
  servers << {'name' => name,
              'ip' => ip,
              'initial_token' => 2**127 / server_count * i}
end

# Vagrant::Config.run do |config|
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  servers.each do |server|
    config.vm.box = "precise"
    config.vm.box_url = "http://files.vagrantup.com/precise64.box"
    config.vm.host_name = server['name']
    config.vm.network "private_network", ip: server['ip']
    config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
      vb.customize ["modifyvm", :id, "--memory", 2048 ] # lower memory if you don't have that much.
    end

    config.librarian_chef.cheffile_dir = "vagrant"
    config.vm.provision :shell, :inline => "sudo aptitude -y install build-essential"
    config.vm.provision :shell, :inline => "gem install chef --version 11.4.2 --no-rdoc --no-ri --conservative"
    config.vm.provision :chef_solo do |chef|
      chef.log_level = :debug
      chef.cookbooks_path = ["vagrant/cookbooks", "vagrant/site-cookbooks"]
      chef.add_recipe "updater"
      chef.add_recipe "java"
      chef.add_recipe "cassandra::tarball"
      chef.json = {
        :java => {
            :jdk_version => "7"
        },
        :cassandra => {'cluster_name' => 'My Cluster',
                       'initial_token' => server['initial_token'],
                       'seeds' => seeds.join(","),
                       'listen_address' => server['ip'],
                       'broadcast_address' => server['ip'],
                       'rpc_address' => server['ip']}
      }
    end
    config.vm.define server['name'] 
  end
  
end
