# Installation of HDP in AWS
This guide describes to do manually setup Ambari on EC2 for the purpose of setting up a custom Hadoop environment using HDP (Hortonworks build packages)

## Reference Links
### [Hortonhorks Installation Guide](http://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/ch_Getting_Ready.html)
### [Hortonhorks in EC2 Post](https://hortonworks.com/blog/deploying-hadoop-cluster-amazon-ec2-hortonworks/)

## 1st Step: AMI Creation

### Create a Security Group

* Create new security group with the name: bts-hdp
```bash
HTTP         TCP  80         Anywhere

HTTP         TCP  8080       Anywhere

SSH          TCP  22         Anywhere

CustomTCP    TCP  7180       Anywhere
```
* Save the group and edit again to add:

```bash
AllTCP       TCP  0 - 65535  (bts-hdp)

AllUDP       UDP  0 - 65535  (bts-hdp)
```
### Launch a Instance

#### Create 1 Intance (m4.large) with the options
* AMI: RedHat enterprise 7.3
* Network: vpc by default + No precense subnet + Public IP Use subnet  setting (enable)
* Storage: Root disk 40 GB + Add Volume 100GB Magnetic (delete on termination)
* Use security group created (bts-hdp)
* Create or use new .pem key (copy this file in a secure place)

### Setup Base image
* Connect SSH as ec2-user to public ip address
```bash
ssh -i bts-hdp.pem ec2-user@[public_ip_address]
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

* Install text editor:
```bash
sudo yum install nano -y
```

* Enable the pernament mount:
  * edit this file: /etc/fstab
  ```bash
  sudo nano /etc/fstab
  ```
  * add this line:
  ```bash
  /dev/xvdb /grid ext4 defaults,nofail 0 2
  ```

* Reboot and Reconnect
```bash
sudo reboot

ssh -i bts-hdp.pem ec2-user@[public_ip_address]
```

* Disable SELinux
  * edit this file: /etc/sysconfig/selinux
  ```bash
  sudo nano /etc/fstab
  ```
  * change this value:
  ```bash
  SELINUX=disabled
  ```

* Install and disable firewall:
```bash
sudo yum install firewalld
sudo systemctl disable firewalld
sudo service firewalld stop
```

* Install and enable ntp
```bash
sudo yum install ntp
sudo systemctl enable ntpd
sudo systemctl start ntpd
```
* Check umask
```bash
umask
```
  * If umask is 0002 edit this file: /etc/profile 
  ```bash
  sudo nano /etc/profile 
  ```
  * Remove the IF statement clause and let only: umask 022

* Reboot and Reconnect
```bash
sudo reboot

ssh -i key.pem ec2-user@[master_public_ip]
```

* Ensure that umask is 0022
```bash
umask
```

* Go to AWS dashboard and create an AMI based on this instance:
Right button > Image > Create Image



## 2nd Step: Spinning up other machines

* Go into AWS Console and create new instances:
  * 2 Instances m4.large using the AMI created

* Under "configure instance" Add the following lines "as text":
```bash
sudo mkdir /grid
sudo mkfs -t ext4 /dev/xdvb
sudo mount /dev/xdvb /grid
```
* Once created, copy in a file the list of Private DNS.

* Add pthe em key in all the nodes
  * From the local computer we will copy the .pem key in the servers
  ```bash
  scp -i key.pem .aws/marc-bts.pem ec2-user@[public_ip]:
  ```
  * Connect and Move the key in each server
  ```bash
  ssh -i key.pem ec2-user@[public_ip]

  mv key.pem .ssh/id_rsa
  ```


## 3rd Step: Install Ambari

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
  ```


## 4th Step: Install  HDP using Ambari

This is a guide to load up the hadoop ecosystem to begin writing big data applications that interact with HDP

* Start all nodes: HDPNode1, HDPNode2, HDPMaster
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
