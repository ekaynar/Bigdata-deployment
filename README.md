	Licensed to the Apache Software Foundation (ASF) under one or more
	contributor license agreements. See the NOTICE file distributed with
	this work for additional information regarding copyright ownership.
	The ASF licenses this file to You under the Apache License, Version 2.0
	(the "License"); you may not use this file except in compliance with
	the License. You may obtain a copy of the License at

	http://www.apache.org/licenses/LICENSE-2.0

	Unless required by applicable law or agreed to in writing, software
	distributed under the License is distributed on an "AS IS" BASIS,
	WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	See the License for the specific language governing permissions and
	limitations under the License.

----------------------------------------------------------------------------

# BigTop Bare-metal Provisioner

## Overview 

This vagrant recipe is based on the vagrant recipe from `vagrant-puppet-vm` with added feature of vagrant-openstack-provider plugin. The plugin allows us to deploy a Hadoop cluster on an actual baremetal. 

The Vagrantfile creates a BigTop Bare-metal Big-Data cluster by using BigTop puppet recipes and pulling from existing bigtop repositories. Big-Data enviroment may include Hadoop, Hive, Mahout..etc.

When the configuration is correctly set up in vagrantconfig.yaml, we should be able to deploy a cluster with on single command `vagrant up`


## Set up the enviroment

1) Install vagrant from [official website](https://www.vagrantup.com/)

2) If you want to provision machines in parallel, install gnu-parallel

```
# for centos
yum install parallel 
# for mac
brew install parallel
# for debian/ubuntu
apt-get install parallel
```



3) Install [vagrant-hostmanager plugin](https://github.com/smdahlen/vagrant-hostmanager) to better manage `/etc/hosts`

```
vagrant plugin install vagrant-hostmanager
```

4) Install [vagrant-managed-servers](https://github.com/tknerr/vagrant-managed-servers) 

```
vagrant plugin install vagrant-managed-servers
```

5) Set up configuration 

There are options in vagrantconfig.yaml that you can specify such as set number of bare-metal machines in the cluster

```
num_instance: 3
```

You can also determine what components are being installed and tested

```
components: [hadoop, yarn, hive]

```

6) Define the bare-metal servers ip addresses and hostnames.
 In Vagrantfile you should specify the bare-metal servers ip addresses and hostnames such that:

```
machines_ips = Array["111.111.11.11","222.222.22.22"]
machines_hostnames = Array["server1.bu.edu","server1.bu.edu"]
machines_aliases = Array[server1","server2"]
```

Also you should define the master node such that:
```
hadoop_master = “server1.bu.edu”
```
## GO
For sequencial Big-Data deployment run:

```
vagrant up --provider=managed
vagrant provision
```

For parallel Big-Data deployment run:

```
./para-deploy.sh
```


