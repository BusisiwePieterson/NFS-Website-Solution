# NFS 3-TIER ARCHITECTURE

![image](image/Screenshot_1.png)

## Prepare NFS Server

Spin up 4 EC2 Instances 

   - Storage : RHEL linux 8 (NFS Server)
   - 2 Webservers : RHEL linux 8
   - Database Server : Ubuntu 20.04 (MySQL Database)

![image](image/Screenshot_2.png)

Create a 10 Gib Volume then attach it to the NFS Server

![image](image/Screenshot_3.png)

Create a partition on each of the 3 disks
   - sudo gdisk /dev/xvdf
   - sudo gdisk /dev/xvdg
   - sudo gdisk /dev/xvdh

![image](image/Screenshot_4.png)

![image](image/Screenshot_5.png)

Run `sudo yum install lvm2` to install the LMV2 package. 

![image](image/Screenshot_6.png)

Use `pvcreate` to mark each of the 3 disks as physical volumes to be used by LVM


![image](image/Screenshot_7.png)

Add all 3 PVs to a group called **web-data VG**
   - Run `sudo vgcreate webdata-vg /dev/xvh1 dev/xvdg1 /dev/xvdf1`
   
   **Create 3 Logical Volumes**
   - `sudo lvcreate -n lv-apps -L 9G webdata-vg`

   - `sudo lvcreate -n lv-logs -L 9G webdata-vg`
   
   - `sudo lvcreate -n lv-opt -L 9G webdata-vg`

   **Then Create mount points on `/mnt` directory for the logical volumes as follow:**

   - `sudo mkfs -t xfs /dev/webdata-vg/lv-apps`
   - `sudo mkfs -t xfs /dev/webdata-vg/lv-logs`
   - `sudo mkfs -t xfs /dev/webdata-vg/lv-opt`

![image](image/Screenshot_8.png)


Install the NFS server and configure it to start on reboot and check that it is running.

  ```
  sudo yum -y update
  sudo yum install nfs-utils -y
  sudo systemctl strat nfs-server.service
  sudo systemctl enable nfs-server.service
  sudo systemctl status nfs-server.service
  
  ```

![image](image/Screenshot_12.png)



Set up pemissions that will allow the Web servers to read, write and execute files on NFS.

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service

```

![image](image/Screenshot_14.png)

Configure access to NFS within the same subnet

```
sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

sudo exportfs -arv

```

![image](image/Screenshot_15.png)

Check which port is used by NFS and open it using SG Inbound Rules.

`rpcinfo -p | grep nfs`


![image](image/Screenshot_16.png)

Open the following ports: **TCP 111, UDP 111, UDP 2049, TCP 2049** for NFS server to be accessible from your client.

![image](image/Screenshot_17.png)

## Configure backend database as part of 3 tier architecture

Launch a database server, ssh into it and install MySQL server. 
  
  ```
  sudo apt upgrade
  sudo apt install mysl-server -y
  
  ```


![image](image/Screenshot_9.png)

Add Inbound Rule TCP 3306 and the source as the DB CIDR `172.31.0.0/20`

![image](image/Screenshot_10.png)

Create a database and name it `tooling`

 - `CREATE DATABASE tooling;`

Create a database user and name it `webaccess`

 - `CREATE USER 'webaccess'@'172.31.0.0/20' IDENTIFIED BY 'password123';`

![image](image/Screenshot_11.png)



![image](image/Screenshot_13.png)

Install NFS client -  `sudo yum install nfs-utils nfs4-acl-tools -y`

![image](image/Screenshot_18.png)

Mount `/var/www/` and traget the NFS server's export for apps

```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

```

![image](image/Screenshot_19.png)

Make sure that the changes will persist on Web Server after reboot.

Open `sudo vi /etc/fstab` and add the line `<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
`

![image](image/Screenshot_20.png)

Install Remi's repo, Apached and PHP

```
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

setsebool -P httpd_execmem 1


```

![image](image/Screenshot_21.png)

![image](image/Screenshot_22.png)

Next, we verify that Apache files and directories are available on the Web Server in `/var/www` nd also on the NFS server in `/mnt/apps`

Then we create a new file `touch test.txt` to check if the same file is accessible from other Web Servers

![image](image/Screenshot_25.png)

![image](image/Screenshot_26.png)

![image](image/Screenshot_27.png)

![image](image/Screenshot_28.png)

![image](image/Screenshot_29.png)

![image](image/Screenshot_30.png)

![image](image/Screenshot_31.png)

![image](image/Screenshot_32.png)

![image](image/Screenshot_33.png)

![image](image/Screenshot_34.png)

![image](image/Screenshot_35.png)

![image](image/Screenshot_36.png)

![image](image/Screenshot_37.png)

![image](image/Screenshot_38.png)

![image](image/Screenshot_39.png)

![image](image/Screenshot_40.png)

![image](image/Screenshot_41.png)

