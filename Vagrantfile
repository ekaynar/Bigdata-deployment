# -*- mode: ruby -*-
# vi: set ft=ruby :

require "yaml"
_config = YAML.load(File.open(File.join(File.dirname(__FILE__), "vagrantconfig.yaml"), File::RDONLY).read)
CONF = _config

# Read configurations from config file
# number of instances
num_instances = CONF['num_instances']

# Whether to run smoke tests
run_smoke_tests = CONF['run_smoke_tests'] 

# Smoke test Components to run
smoke_test_components = CONF['smoke_test_components'].join(',')

# Components to install on machines
components = CONF['components']

# Which Linux Distribution to use. Right now only centos is tested
distro = CONF['distro']

# JDK package name
jdk = CONF['jdk']

# Repository
repo = CONF['repo']

#Define the master node hostname
hadoop_master = “server1.bu.edu”


#Dine the server ip adreesses and hostanmes in an array.
machines_ips = Array["dummy","129.10.3.32","128.197.41.33"]
machines_hostnames = Array["dummy","server1.bu.edu","server2.bu.edu"]
machines_aliases = Array["dummy","server1","server2”]



$script = <<SCRIPT
service iptables stop
chkconfig iptables off
# Prepare puppet configuration file
mkdir -p /etc/puppet/hieradata
cp /bigtop-home/bigtop-deploy/puppet/hiera.yaml /etc/puppet
cp -r /bigtop-home/bigtop-deploy/puppet/hieradata/bigtop/ /etc/puppet/hieradata/
cat > /etc/puppet/hieradata/site.yaml << EOF
bigtop::hadoop_head_node: #{hadoop_master}
hadoop::hadoop_storage_dirs: [/data/1, /data/2]
bigtop::bigtop_repo_uri: #{repo}
hadoop_cluster_node::cluster_components: #{components}
bigtop::jdk_package_name: #{jdk}
EOF
SCRIPT

#echo "managed.server ="+ machines_ips[i]
Vagrant.configure("2") do |config|
    config.hostmanager.enabled = true
    (1..num_instances).each do |i|
      config.vm.define "baremetal#{i}" do |baremetal|
          baremetal.vm.box = "tknerr/managed-server-dummy"
          #baremetal.hostmanager.enabled = true
          baremetal.ssh.pty = true
          baremetal.vm.provider :managed do |managed, override|
            managed.server = machines_ips[i]
            override.ssh.username = "ukaynar"
            override.ssh.private_key_path = "~/.ssh/id_rsa"
          end
          
          baremetal.vm.hostname = machines_hostnames[i]
          baremetal.hostmanager.aliases = machines_aliases[i]
          

          # sync folder from local to vm using rsync
          baremetal.vm.synced_folder “../../bigtop-home", "/bigtop-home"
          baremetal.vm.synced_folder "~/Desktop/benchmark", "/benchmark" 
          baremetal.vm.provision :hostmanager

          # st up environment, java to be exact to run puppet
          baremetal.vm.provision "shell", inline: <<-SHELL
            sudo yum install -y vim java-1.7.0-openjdk
            sudo yum install -y words
            sudo chown -R hdfs:hdfs /benchmark
          SHELL

          baremetal.vm.provision :shell do |shell|
            shell.path = "/Users/ugurkaynar/vagrant-managed-servers/bigtop-home/bigtop-deploy/vm/utils/setup-env-" + distro + ".sh"
          end
          baremetal.vm.provision "shell", inline: $script

          # run puppet to deploy hadoop
          baremetal.vm.provision :puppet do |puppet|
            puppet.module_path = "~/MOC/hadoop/bigtop-home/bigtop-deploy/puppet//modules"
            puppet.manifests_path = "~/MOC/hadoop/bigtop-home/bigtop-deploy/puppet/manifests/"
            puppet.manifest_file = "site.pp"
            puppet.options = '--debug'
          end
        end
    end
end
