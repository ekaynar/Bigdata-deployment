# -*- mode: ruby -*-
# vi: set ft=ruby :

require "yaml"
_config = YAML.load(File.open(File.join(File.dirname(__FILE__), "vagrantconfig.yaml"), File::RDONLY).read)
CONF = _config

# number of instances
num_instances = CONF['num_instances']
# Whether to run smoke tests
run_smoke_tests = CONF['run_smoke_tests'] 
# Smoke test Components to run
smoke_test_components = CONF['smoke_test_components'].join(',')
components = CONF['components']
distro = CONF['distro']
jdk = CONF['jdk']
repo = CONF['repo']
hadoop_master = "moc01.bu.edu"

machines_ips = Array["128.197.41.26","128.197.41.33"]

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

Vagrant.configure("2") do |config|

  (1..num_instances).each do |i|
      
      config.vm.box = "tknerr/managed-server-dummy"
      #config.hostmanager.enabled = true
      config.ssh.pty = true
      config.vm.provider :managed do |managed, override|
        managed.server = machines_ips[i]
        override.ssh.username = "ukaynar"
        override.ssh.private_key_path = "~/.ssh/id_rsa"
      end
      config.vm.synced_folder "/Users/ugurkaynar/vagrant-managed-servers/bigtop-home", "/bigtop-home"
      config.vm.synced_folder "~/Desktop/benchmark", "/benchmark" 

      config.vm.provision "shell" do |s|
        s.inline = "echo hello >> test.txt"
      end

      config.vm.provision "shell", inline: <<-SHELL
        sudo yum install -y vim java-1.7.0-openjdk
        sudo yum install -y words

      SHELL
      config.vm.provision :shell do |shell|
        shell.path = "/Users/ugurkaynar/vagrant-managed-servers/bigtop-home/bigtop-deploy/vm/utils/setup-env-" + distro + ".sh"
      end
      config.vm.provision "shell", inline: $script

    # run puppet to deploy hadoop
      config.vm.provision :puppet do |puppet|
        puppet.module_path = "~/MOC/hadoop/bigtop-home/bigtop-deploy/puppet//modules"
        puppet.manifests_path = "~/MOC/hadoop/bigtop-home/bigtop-deploy/puppet/manifests/"
        puppet.manifest_file = "site.pp"
        puppet.options = '--debug'
      end
  end
end
