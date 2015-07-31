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
'''
# for centos
yum install parallel 
# for mac
brew install parallel
# for debian/ubuntu
apt-get install parallel
'''


2) Install [vagrant-hostmanager plugin](https://github.com/smdahlen/vagrant-hostmanager) to better manage `/etc/hosts`

```
vagrant plugin install vagrant-hostmanager
```

2) Install [vagrant-managed-servers](https://github.com/tknerr/vagrant-managed-servers) 

```
vagrant plugin install vagrant-managed-servers
```

3) Set up configuration 

There are other options in vagrantconfig.yaml that you can specify such as set number of vms in the cluster, and automatically run smoke tests

```
num_instance: 3
rur_smoke_test: true
```

You can also determine what components are being installed and tested

```
components: [hadoop, yarn]
smoke_test_components: [mapredcue, pig]
```

## GO
For sequencial Big-Data run:

```
vagrant up --provider=managed
vagrant provision
```

For parallel provisioning run:

```
./para-deploy.sh
```

#### Parallel provisioning

**This script is based on Joe Miller's para-vagrant.sh script please see NOTICE for more information**

Script reads parameter `num_instance`, `run_smoke_tests` and `smoke_test_components` from `vagrantconfig.yaml` to determine how many machines to spin up and whether or not to run smoke tests and what components will be tested

This script will spin up vms on openstack sequentially first, and then do the provisioning in parallel. Each guest machine will have it's own log file. And will generate a log file for smoke tests if `run_smoke_tests` set to true 

There are some sketchy places in the code...(cuz I suck in bash) such as: 
* handle unprintable ^M without destroying the format of log file. I haven't figure out how yet
* use of `sed`: so OS X hates me, and it won't let me use `sed -r` so again I create another temporary file for the smoke tests log


#### TODO

* test installing all available components
  * spark2 is not working, looks like a puppet problem
* test all the provided smoke tests
  * mahout smoke tests didn't run all the way throught
* enable_local_repo
* modify the code to make it more generic, I only tried this on Centos 6
