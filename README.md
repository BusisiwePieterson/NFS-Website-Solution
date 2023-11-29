# NFS 3-TIER ARCHITECTURE

![image](image/Screenshot_1.png)

## Prepare NFS Server

1. Spin up 4 EC2 Instances 
   - Storage : RHEL linux 8 (NFS Server)
   - 2 Webservers : RHEL linux 8
   - Database Server : Ubuntu 20.04 (MySQL Database)

![image](image/Screenshot_2.png)

2. Create a 10 Gib Volume then attach it to the NFS Server

![image](image/Screenshot_3.png)

3. Create a partition on each of the 3 disks
   - sudo gdisk /dev/xvdf
   - sudo gdisk /dev/xvdg
   - sudo gdisk /dev/xvdh

![image](image/Screenshot_4.png)

![image](image/Screenshot_5.png)

4. Run `sudo yum install lvm2` to install the LMV2 package. 

![image](image/Screenshot_6.png)

5. Use `pvcreate` to mark each of the 3 disks as physical volumes to be used by LVM


![image](image/Screenshot_7.png)

6. Add all 3 PVs to a group called **web-data VG**
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


7. Install the NFS server and configure it to strat on reboot and check that it is running.

  ```
  sudo yum -y update
  sudo yum install nfs-utils -y
  sudo systemctl strat nfs-server.service
  sudo systemctl enable nfs-server.service
  sudo systemctl status nfs-server.service
  
  ```

![image](image/Screenshot_12.png)




![image](image/Screenshot_9.png)

![image](image/Screenshot_10.png)

![image](image/Screenshot_11.png)



![image](image/Screenshot_13.png)

![image](image/Screenshot_14.png)

![image](image/Screenshot_15.png)

![image](image/Screenshot_16.png)

![image](image/Screenshot_17.png)

![image](image/Screenshot_18.png)

![image](image/Screenshot_19.png)

![image](image/Screenshot_20.png)

![image](image/Screenshot_21.png)

![image](image/Screenshot_22.png)

![image](image/Screenshot_23.png)

![image](image/Screenshot_24.png)

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

