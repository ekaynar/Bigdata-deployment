## Usage
1) Set up environment

Install vagrant from official website

If you want to provision machines in parallel, install gnu-parallel

# for centos
'''
yum install parallel
'''
# for mac
'''
brew install parallel
'''
# for debian/ubuntu
'''
apt-get install parallel
'''

2) Install [vagrant-hostmanager](https://github.com/smdahlen/vagrant-hostmanager) plugin to better manage /etc/hosts
```
$ vagrant plugin install vagrant-hostmanager
```
2) Install [vagrant-managed-servers](https://github.com/tknerr/vagrant-managed-servers)
```
vagrant plugin install vagrant-managed-servers
```