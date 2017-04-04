# Installation of HDP in AWS
This guide describes to do manually setup Ambari on EC2 for the purpose of setting up a custom Hadoop environment using HDP (Hortonworks build packages)

## Reference Links
### [Hortonhorks Installation Guide](http://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/ch_Getting_Ready.html)
### [Hortonhorks in EC2 Post](https://hortonworks.com/blog/deploying-hadoop-cluster-amazon-ec2-hortonworks/)

## AWS Provisioning

### Create a Security Group

Name: bts-hdp

HTTP         TCP  80         Anywhere
HTTP         TCP  8080       Anywhere
SSH          TCP  22         Anywhere
CustomTCP    TCP  7180       Anywhere
AllTCP       TCP  0 - 65535  (bts-hdp)
AllUDP       UDP  0 - 65535  (bts-hdp)

### Setting up Apache Ambari on EC2

#### Create 1 Intance (m4.large) for Ambari Head
* Redhat enterprise 7
* Protect against accidental termination
* Add 100GB magnetic storage (delete on termination)
* Use security group created
* Create or use appdev pemfile (maintain in a secure place)

### Setup base image
* SSH as ec2-user to public ip address
```bash
ssh -i ec2-user@[master_public_ip]
```
* Check location
```bash
lsblk
```
* Format 100GB drive
```bash
sudo mkfs -t ext4 /dev/xvdb
```
* Mount the drive
```bash
sudo mkdir /grid && sudo mount /dev/xvdb /grid
```
* Add info to /etc/fstab to be pernament mount
```bash
/dev/xvdb /grid ext4 defaults,nofail 0 2
```
* Reboot and reconnect
* Disable SELinux
  * in /etc/sysconfig/selinux
```bash
SELINUX=disabled
```
* Install and disable firewall
```bash
sudo yum install firewalld
sudo systemctl disable firewalld
sudo service firewalld stop
```
* Install ntp
```bash
sudo yum install ntp
```
* Start ntpd
```bash
sudo systemctl enable ntpd
sudo systemctl start ntpd
```
* Check umask
```bash
umask
```
  * If umask is 0002 edit (/etc/profile) to have umask be just 0022 (remove if statement clause)

* Reboot 
```bash
sudo reboot
ssh -i key.pem ec2-user@[master_public_ip]
```
* Ensure that umask is 0022
```bash
umask
```
* Add pem key in the nodes
  * From the local computer we will copy the .pem key in the servers
  ```bash
  scp -i key.pem .aws/marc-bts.pem ec2-user@[public_ip]:
  ```
  * Move the key in each server
  ```bash
  ssh -i key.pem ec2-user@[public_ip]

  mv key.pem .ssh/id_rsa
  ```

* Go to AWS dashboard and create image based on this instance
Right button > Image > Create Image

### Spinning up other machines
* Go into AWS Console and create new instances
  * 2 Instances m4.large using the image created
* Under configure instance Add the following lines "as text":
```bash
sudo mkdir /grid
sudo mkfs -t ext4 /dev/xdvb
sudo mount /dev/xdvb /grid
```
* Once created, copy in a file the list of Private DNS


## Install Ambari
* Connect to the Ambari Machine
* Install wget
```bash
sudo yum install wget
```
* Add the hdfp ambari repo 
```bash
sudo wget -nv http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.4.0.1/ambari.repo -O /etc/yum.repos.d/ambari.repo
```
* Update yum
```bash
sudo yum repolist
```
* Install ambari server 
```bash
sudo yum install ambari-server
```
* Setup ambari-server
```bash
sudo ambari-server setup
```
  * Use defaults (No custom user account, java8, accept agreement, no advanced db config)
* Start ambari-server
```bash
sudo ambari-server start
```
* SOLVING KNOWN ISSUES:
  * [known issues](https://docs.hortonworks.com/HDPDocuments/Ambari-2.1.0.0/bk_releasenotes_ambari_2.1.0.0/content/ambari_relnotes-2.1.0.0-known-issues.html)
  * For each host:
  ```bash
  sudo yum install ntp
  sudo systemctl enable ntpd
  sudo systemctl start ntpd
  sudo yum remove snappy
  sudo yum install snappy-devel



## Ambari Loadup on EC2
This is a guide to load up the hadoop ecosystem to begin writing big data applications that interact with HDP
* Start all nodes: HDPNode1,HDPNode2,ambariHead on ec2
* Start ambari-server on ambariHead
```bash
ssh -i key.pem ec2-user@[master_public_ip]
sudo ambari-server start
```
* Go to [master_public_ip]:8080
* Login into Ambari using our login information

### Create a Cluster
* add the list of Private DNS
* upload the key .pem

### Install this services
  * HDFS (Hadoop file system)
  * YARN (resource manage)
  * Hive (HiveSQL for Hadoop)
  * ZooKeeper (Leader election)
  * HBase (noSQL key-value store)
  * Spark (in-memory cluster computing)


### Create Cluster
* Connect through browser to ambari at public_url:8080
* Username: admin Password: admin
* Name the cluster
* Select HDP2.5.3 (HDP version 2.5.3)
  * Under Advanced Options unselect all repos that arent rhel7
* For target hosts, copy the privateDNS addresses for all machines
* For SHH key, paste in the id_rsa file that was generated
* User is ec2-user


