Prac1: 
Install httpd service :
yum install httpd
To start the httpd service :
service : 
service httpd start
systemctl enable httpd.start
Home dir of httpd service :
/var/www/html

Configuration file of httpd service :
/etc/httpd/conf/httpd.conf

Redirect : (in httpd.conf)
Redirect permanent /redirectFrom http://localhost/redirectTo
permanent means long term usage.

restart httpd service: 
service httpd restart

Alias (in httpd.conf) :
Alias /pathToShowOn /var/www/html/FolderToBeShown
Shows content of a different folder URI without redirecting 

Prac2 : 
Set ACL RULES for group consultants for /shares/content:
setfacl -Rm g:consultants:rwx /shares/content.
ACL Rules for user consultant1:
setfacl -Rm u:consultant1:- /shares/content (no permissions)
To set default ACL rules
setfacl -m d:u:consultant1:- /shares/content
To see all ACL entries :
getfacl /shares/content
1. Use the *chgrp* command to recursively update group ownership on the directory and its contents.
    
    
    [root@serverb ~]# **`chgrp -R managers /shares/cases`**
    
    
2. Use the *chmod* command to update the set-GID flag on the directory.
    
    
    [root@serverb ~]# **`chmod g+s /shares/cases`**
    
    
3. Use *chmod* to update all existing file permissions to rw for owner and group.
    
    
    [root@serverb ~]# **`chmod 660 /shares/cases/*`**
    
Use *setfacl* to recursively update the existing cases directory and its contents. Grant the contractors group read, write, and conditional execute permissions.


[root@serverb ~]# setfacl -Rm g:contractors:rwX /shares/cases

Prac3:
list block devices attached to the system: 
lsblk
Creating a new partition :
fdisk /dev/vdb
Assign file system to partition:
mkfs.ext4 /dev/vdb1
assigning mount point:
mount /dev/vdb1 /folderPath
display information about available block devices and
their associated attributes, such as UUIDs (Universally Unique
Identifiers) and file system types, on a Linux system.:
blkid
reboot the server:
reboot
fstab file path: 
/etc/fstab
unmount using :
umount /dev/vdb1 /folderPath
prac4: 
to use partition as swap memory
mkswap /dev/vdb1
to use swap memory : 
swapon
check if swap memory is created using :
lsblk or free
delete swap memory:
swapoff /dev/vdb1

prac5:
//creata pyhscial volume using 3 disks vdb1,2,3
pvcreate /dev/vdb1 /dev/vdb2 /dev/vdb3

//to display if physical volume is created
pvdisplay

//to create volume group (vggroup is the name of group anything can be given)
vgcreate vggroup /dev/vdb1 /dev/vdb1 /dev/vdb1

//now check if group is created check vgname in
pvdisplay

//PE smallest unit of storage

//create a logical volume (-L to specify the size, -n to give name)
lvcreate -L +1.5G -n lvgroup vggroup

// to display logical volume
lvdisplay

//create a directory where to mount
mkdir /mnt/lvdir

// to assign a file system
mkfs -t xfs /dev/vggroup/lvgroup

//go into /etc/fstab 
vim /etc/fstab
//add line
/dev/vggroup/lvgroup  /mnt/lvdir  xfs  defaults 0 0


//mount all
mount -a
//reboot
reboot

//login to server and check volume group and logical volumn
lsblk

//to delete
//1) umount 2) //lvremove vgremove pvremove

//lvextend to extend logical volume add space to logical volume
// -r resize file system -L size
lvextend -L +1G -r /dev/vggroup/lvgroup

// forcing the kernel to reread the files
partprobe

//to reboot
reboot

//lvs or lvdisplay to display

Prac6:
//to remove a physical volume from a volume group
vgreduce volGroupName /dev/vdd1

//to rename LVM:
lvrename volGroupName currLogVolName NewName
//create 2 partitions choose type Linux LVM
fdisk /dev/vdb

// add them to physcial volume
pvcreate /dev/vdb1 /dev/vdb2 
// add them to logical volume group
vgcreate  testvg /dev/vdb1 /dev/vdb2
// add lvg to logical volume
lvcreate -L +400M -n testlv testvg
//assign file system
mkfs -t xfs /dev/testvg/testlv
//create a dir and mount
mkdir /mnt/test
mount /dev/testvg/testlv /mnt/test
//enter the logical volume in fstab file
vim /etc/fstab
//then reboot the server

//extend the logical volume by 200mb
lvextend -L +200M -r /dev/testvg/testlv

//we want to add 1.6g of vg
//create a partition 
fdisk /dev/vdc 
pvcreate /dev/vdc1
//to extend vg
vgextend testvg /dev/vdc1
//
lvextend -L +1G -r /dev/testvg/testlv

//to extend the size of vg and in that PE
vgcreate -s 8M vggroup /dev/vdd1 /dev/vdd2
//to create 50 such PE
lvcreate -l 50 -n lvgroup vggroup
//to extend to more 20 PE
lvextend -l +20 /dev/vggroup/lvgroup
//Guided Exercise not added

extra 
1. *lvresize*: With this command, you can resize (shrink or expand) an existing logical volume. It allows you to adjust the size of a logical volume dynamically, provided there is available space in the volume group.
    
2. *lvrename*: Used for renaming logical volumes. This command allows you to change the name of a logical volume.
3. *lvreduce*: Used to reduce the size of a logical volume. It can be used to reclaim unused space from a logical volume and return it to the volume group.
to give pe size : vgcreate --physicalextentsize 16M /dev/vdd3 -n yagna
to set amout of pe : lvcreate -l 20 yagna -n logvol
Prac7 : 
Do this in root workstation prac7

//check if httpd is running
serive httpd status

//install httpd service
yum install httpd

//start httpd service
service httpd start

//create a dir
mkdir /var/www/html/batch61
touch index.html
add  a line in index.html
//get ip address 
ifconfig

// get into configuration file
vim /etc/httpd/conf/httpd.conf
//at the end of the configuration file
NameVirtualHost ip_address:port //port is optional
<VirtualHost ip_address>
	DocumentRoot /var/www/html/batch61
	ServerName virtualhost.21162101020.com
</VirtualHost>

//to add hosts 
vim /etc/hosts
//add the host
ip_address batch61.com

restart httpd service

<Directory '/var/www/html/batch61'>
	AuthType Basic
	AuthName "Please enter the password"
	AuthBasicProvider file 
	AuthUserFile /etc/httpd/userpassword
	AuthGroupFile /etc/httpd/usergroup
	Require user user1 
	//Require group cba
</Directory>

restart httpd 

//add data in authentication file
vim /var/www/html/batch61/.htaccess
AuthType Basic
	AuthName "Please enter the password"
	AuthBasicProvider file 
	AuthUserFile /etc/httpd/userpassword
	Require user user1
	AuthGroupFile /etc/httpd/usergroup
	
restart httpd
//add user 
htpasswd -c /etc/httpd/userpassword user1
htpasswd /etc/httpd/userpassword user2
htpasswd  /etc/httpd/userpassword user3


//group
vim  /etc/httpd/usergroup
cba: user1 user2

Prac8
to install mailx, postfix and dovecot:
yum install mailx postfix dovecot
to create users:
useradd user1
To give password:
passwd user1

main.cf configuration file : 
/etc/postfix/main.cf
send mail using :
sendmail user@ac.in


Prac9
to get default zone:
firewall-cmd --get-default-zone
to get status of firewall 
firewall-cmd --state
intsall httpd and mod_ssl
yum install httpd mod_ssl
verify the status of nftables service is masked :
systemctl status nftables
verify the status of firewall service is enable:
systemctl status firewalld
allow a port,so services running on that port is allowed in network.
sudo firewall-cmd --zone=public --add-port=234/tcp --permanent
to check configuration :
firewall-cmd --list-all
to block a service :
sudo firewall-cmd --remove-service=cockpit --permanent
to reload :
firewall-cmd --reload
to disable firewall : 
systemctl disable firewalld
to enable firewall :
systemctl enable firewalld
to allow traffic from a specific IP address
firewall-cmd --zone=public --add-source=192.168.1.100 --permanent

prac10:
For certificate and key generation, we need to install two services:
mod_ssl and openssl (already did before).
For ssl certificate and key here is the command for that :
Openssl req -x509 -nodes -newkey rsa:2048 -keyout
/var/www/html/prac10_yagna/yagna.key -out
/var/www/html/prac10_yagna/yagna.crt
Here in this command
-x509 is is a digital certificate that uses the widely accepted
international X.
-newkey for generate newkey
Rsa:2048 here rsa is an type of cryptosystem an 2048 is key size
-keyout path is for generate key in given path

Added port 443 with ip
SSLEngine on //set to on
SSLCertificateFile /path/certificate //adding certificate
SSLCertificateKeyFile /path/key //adding key

prac11
Task 1: Create a user of your name and schedule a job to create a file which
have the current date and time stored in it. The job should be executed on a
specific date and time (you can specify the date and time as per your
convenience)
Now we’ll create a user named yagna with useradd yagna
Now for a set date and time, we’ll enter into the crontab job schedule file with
Command: crontab -u yagna -e
Here -u yagna This means that you're editing the crontab file for the
specified user.
And in the crontab file, we’ll just put the bellowed line
7 11 3 3 * date >> /home/student/task1.txt
In this command
7: Minute 11: Hour 3: Day 3: Month * : Day of the week (any day of the week)
Date is a function that return current date and time
/home/student/task1.txt will save to this path

If we have scheduled a job via crontab, we can remove it directly through the
crontab file.
But if our job was created via at command than we’ve to remove via atrm
command
Like let suppose
I’ve some job in queue i can see it with command atq

For not allowing user3 and user4 we edit cron.deny file in system and add
user3 and user4 in that file in one at line and save it so than user3 and user4
will not be able to schedule any jobs.

Task 5: You need to create a script that will store the details about the kernel
messages related to drivers in a specific file. And this task
#!/bin/bash
dmesg | grep -i "driver" >> /home/student/kernel_logs.txt
This code will store logs into kernal_logs.txt files
Now to make the script executable we’ll give permission to that script with
command: chmod +x task1.sh

prac12:
### Practical 12 Tasks with Commands

#### Task 1: Configure MariaDB server on server.example.com

**a) Set the root password as "access" and block root access from remote hosts.**

```sh
sudo yum install mariadb mariadb-server
sudo systemctl start mariadb
sudo mysql_secure_installation
```

**b) Create a user "yourname" by password "password".**

```sh
mysql -u root -p
CREATE USER 'yourname'@'localhost' IDENTIFIED BY 'password';
```

**c) Create another user with password as "@yourname".**

```sh
CREATE USER 'user1'@'localhost' IDENTIFIED BY '@yourname';
```

**d) Only local hosts should have access to MariaDB server.**

```sh
sudo mysql_secure_installation
```

#### Task 2: Security settings for MariaDB

```sh
sudo mysql_secure_installation
```

#### Task 3: Create databases and tables with entries

```sh
mysql -u root -p
CREATE DATABASE Batch61;
CREATE DATABASE yourname_2116;
USE Batch61;
CREATE TABLE table1 (id INT PRIMARY KEY, name VARCHAR(255), age INT);
CREATE TABLE table2 (id INT PRIMARY KEY, subject VARCHAR(255), score INT);
CREATE TABLE table3 (id INT PRIMARY KEY, department VARCHAR(255), location VARCHAR(255));
INSERT INTO table1 VALUES (1, 'Alice', 22), (2, 'Bob', 23), (3, 'Charlie', 24), (4, 'David', 25), (5, 'Eve', 26);
```

#### Task 4: Assign user privileges

**Full access to both databases:**

```sh
GRANT ALL PRIVILEGES ON *.* TO 'yourname'@'localhost' IDENTIFIED BY 'password';
```

**Read access to one database, read and write access to another:**

```sh
GRANT SELECT ON Batch61.* TO 'user1'@'localhost' IDENTIFIED BY '@yourname';
GRANT SELECT, INSERT ON yourname_2116.* TO 'user1'@'localhost' IDENTIFIED BY '@yourname';
```

#### Task 5: Access for third user

```sh
CREATE USER 'user2'@'localhost' IDENTIFIED BY 'user2';
GRANT ALL PRIVILEGES ON Batch61.table1 TO 'user2'@'localhost';
GRANT ALL PRIVILEGES ON Batch61.table2 TO 'user2'@'localhost';
```

#### Task 6: Check user privileges

```sh
SHOW GRANTS FOR 'user'@'localhost';
SELECT USER, Select_priv, Insert_priv, Update_priv, Delete_priv FROM mysql.user;
```

#### Task 7: Revoke a permission

```sh
REVOKE SELECT ON Batch61.* FROM 'user1'@'localhost';
```

#### Task 8: Secure MariaDB database and create table

```sh
CREATE DATABASE result;
USE result;
CREATE TABLE students (name VARCHAR(20), marks INT(10));
INSERT INTO students VALUES ('John', 85), ('Jane', 90);
```

#### Task 9: Update records

```sh
UPDATE students SET marks=95 WHERE name='John';
```

#### Task 10: Delete user

```sh
DROP USER 'user'@'localhost';
```

#### Task 11: Backup database

```sh
mysqldump -u root -p Batch61 > /root/result.dump
```

#### Task 12: Grant update privilege

```sh
CREATE USER 'user3'@'localhost' IDENTIFIED BY 'user3';
GRANT UPDATE ON *.* TO 'user3'@'localhost';
```

#### Task 13: Delete a table

```sh
DROP TABLE table1;
```

#### Task 14: Delete a database

```sh
DROP DATABASE yourname_2116;
```

These commands follow the instructions from the practical assignment and perform the necessary tasks on the MariaDB server.

Prac13:
Here are the detailed questions, commands, and explanations based on the provided practical tasks:

### Task 1: Working with the Apache HTTP Server Container using Podman

**Question**: How do you pull an `httpd` image from `registry.redhat.io`, run it with port forwarding, perform persistent mounting, and manage it using systemctl?

1. **Install Podman**:
   ```sh
   sudo yum install podman
   ```
   **Explanation**: Installs Podman, a container management tool, which is an alternative to Docker.

2. **Login into Podman**:
   ```sh
   podman login registry.redhat.io
   ```
   **Explanation**: Logs into the Red Hat container registry using your Red Hat credentials.

3. **Search for the `httpd` image**:
   ```sh
   podman search httpd
   ```
   **Explanation**: Searches the Red Hat container registry for the `httpd` image.

4. **Pull the `httpd` image**:
   ```sh
   podman pull registry.redhat.io/rhel8/httpd-24
   ```
   **Explanation**: Pulls the specified `httpd` image from the Red Hat container registry.

5. **Check installed images**:
   ```sh
   podman images
   ```
   **Explanation**: Lists all images that have been pulled and are available locally.

6. **Run the image with port forwarding**:
   ```sh
   podman run -d -p 8000:8080 registry.redhat.io/rhel8/httpd-24
   ```
   **Explanation**: Runs the `httpd` container in detached mode, forwarding host port 8000 to container port 8080.

7. **Allow inbound traffic on port 8000**:
   ```sh
   sudo firewall-cmd --add-port=8000/tcp --permanent
   sudo firewall-cmd --reload
   ```
   **Explanation**: Configures the firewall to allow TCP traffic on port 8000 and reloads the firewall rules.

8. **Create a directory for persistent mounting**:
   ```sh
   sudo mkdir -p /var/www/html
   ```
   **Explanation**: Creates a directory on the host to be used for persistent storage by the container.

9. **Create a systemd service file for managing the container**:
   ```sh
   sudo vim /etc/systemd/system/httpd.service
   ```
   Add the following content:
   ```ini
   [Unit]
   Description=Apache HTTP Server
   After=network.target

   [Service]
   Type=simple
   ExecStart=/usr/bin/podman run -d --publish=8080:80 --volume=/var/www/html:/usr/local/apache2/htdocs registry.redhat.io/rhel8/httpd-24
   ExecStop=/usr/bin/podman stop %n
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```
   **Explanation**: Creates a systemd service file to manage the Apache HTTP server container. The service starts the container with port forwarding and mounts the `/var/www/html` directory for persistent storage.

10. **Enable and start the service**:
    ```sh
    sudo systemctl enable httpd.service
    sudo systemctl start httpd.service
    ```
    **Explanation**: Enables the service to start at boot and immediately starts the service.

### Task 2: Working with MySQL Server Container

**Question**: How do you create a MySQL container, perform port forwarding, create a database and a table, and perform CRUD operations?

1. **Search for MySQL images**:
   ```sh
   podman search mysql
   ```
   **Explanation**: Searches for MySQL images in the container registry.

2. **Pull the MySQL image**:
   ```sh
   podman pull docker.io/library/mysql:latest
   ```
   **Explanation**: Pulls the latest MySQL image from Docker Hub.

3. **Create a directory for persistent storage and set permissions**:
   ```sh
   sudo mkdir -p /var/lib/mysql
   sudo chmod 777 /var/lib/mysql
   ```
   **Explanation**: Creates a directory for MySQL data with full permissions to ensure the container can write to it.

4. **Run the MySQL container**:
   ```sh
   podman run -d --name=mysqlcontainer -p 3306:3306 -e MYSQL_ROOT_PASSWORD=yourpassword -v /var/lib/mysql:/var/lib/mysql mysql:latest
   ```
   **Explanation**: Runs the MySQL container in detached mode, forwarding port 3306 and setting the root password. The host directory `/var/lib/mysql` is mounted for persistent storage.

5. **Open a new container to access MySQL CLI**:
   ```sh
   podman exec -it mysqlcontainer mysql -u root -p
   ```
   **Explanation**: Opens a new interactive terminal session inside the running MySQL container and accesses the MySQL CLI as the root user.

6. **Create a database and a table**:
   ```sql
   CREATE DATABASE yagna;
   USE yagna;
   CREATE TABLE students (
       id INT AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(255) NOT NULL,
       age INT,
       major VARCHAR(255)
   );
   ```
   **Explanation**: Creates a new database named `yagna` and a table `students` with columns for student details.

7. **Perform CRUD operations inside MySQL**:
   ```sql
   INSERT INTO students (name, age, major) VALUES ('John Doe', 20, 'Computer Science');
   SELECT * FROM students;
   UPDATE students SET age = 21 WHERE name = 'John Doe';
   DELETE FROM students WHERE name = 'John Doe';
   ```
   **Explanation**: Inserts a new record into the `students` table, retrieves all records, updates a record, and deletes a record.

These steps provide detailed instructions on how to accomplish each task described in the practical assignment, including the necessary commands and their explanations.

prac14:
### Commands and Explanations from the PDF:

#### Question 1: Ensure that whenever a new user is created, they will have a folder named `welcome` created by default in the home directory.

1. **Create the `welcome` folder inside `/etc/skel`**:
   ```sh
   sudo mkdir /etc/skel/welcome
   ```
   **Explanation**: The `/etc/skel` directory contains files and directories that are automatically copied to a new user's home directory when the user is created. By creating a `welcome` folder here, it ensures that all new users will have this folder in their home directory.

2. **Create a new user and set a password**:
   ```sh
   sudo useradd newuser
   sudo passwd newuser
   ```
   **Explanation**: The `useradd` command creates a new user, and `passwd` sets the password for this user.

3. **Check if the `welcome` folder is created**:
   ```sh
   ls /home/newuser/
   ```
   **Explanation**: This command lists the contents of the new user's home directory to verify the presence of the `welcome` folder.

#### Question 2: Create an Archive

1. **Create an archive file for `/var/log`**:
   ```sh
   sudo tar -cvzf /parent/log_backup.tar.gz /var/log
   ```
   **Explanation**: This `tar` command creates (`c`) a new archive file named `log_backup.tar.gz` in the `/parent` directory, includes verbose output (`v`), compresses it using gzip (`z`), and specifies the archive file name (`f`).

2. **List the contents of the archive**:
   ```sh
   tar -tvzf /parent/log_backup.tar.gz
   ```
   **Explanation**: This `tar` command lists (`t`) the contents of the archive file with verbose output (`v`), decompresses it using gzip (`z`), and specifies the archive file name (`f`).

3. **Extract the archived file**:
   ```sh
   sudo tar -xvzf /parent/log_backup.tar.gz -C /parent/
   ```
   **Explanation**: This `tar` command extracts (`x`) files from the archive with verbose output (`v`), decompresses it using gzip (`z`), specifies the archive file name (`f`), and extracts the contents to the `/parent` directory (`-C /parent/`).

4. **Create 3 files, zip and unzip them**:
   ```sh
   touch file1 file2 file3
   zip files.zip file1 file2 file3
   unzip files.zip -d /destination_directory
   ```
   **Explanation**:
   - `touch file1 file2 file3`: Creates three empty files.
   - `zip files.zip file1 file2 file3`: Compresses the three files into a zip archive named `files.zip`.
   - `unzip files.zip -d /destination_directory`: Extracts the contents of the zip file into the specified destination directory.

#### Question 3: Find all files & directories owned by group `tutor` and copy them to `/var/Document`

1. **Create a group**:
   ```sh
   sudo groupadd tutor
   ```
   **Explanation**: Creates a new group named `tutor`.

2. **Create files**:
   ```sh
   touch file1 file2 file3
   ```
   **Explanation**: Creates three empty files.

3. **Give ownership to the group**:
   ```sh
   sudo chown :tutor file1 file2 file3
   ```
   **Explanation**: Changes the group ownership of the files to `tutor`.

4. **Find files owned by `tutor` group and copy them**:
   ```sh
   sudo find / -group tutor -exec cp -r {} /var/Document \;
   ```
   **Explanation**: Uses the `find` command to search for all files and directories owned by the `tutor` group (`-group tutor`) and then executes (`-exec`) the `cp -r` command to copy them to `/var/Document`. The `{}` is replaced by the current file path found by `find`, and `\;` indicates the end of the command to be executed.

#### Question 4: Search a string from a file and store output

1. **Find files containing the string "ZZZ" and store the output**:
   ```sh
   grep -r "ZZZ" /usr/share/ > /tmp/ingroupfile
   ```
   **Explanation**: Searches recursively (`-r`) in the `/usr/share/` directory for the string "ZZZ" and redirects the output to a file `/tmp/ingroupfile`.

2. **Check the output using `cat`**:
   ```sh
   cat /tmp/ingroupfile
   ```
   **Explanation**: Displays the contents of the file `/tmp/ingroupfile` to verify the search results.

These commands cover the tasks outlined in the practical assignment, ensuring correct implementation and verification of each step.

prac15:
Start the advstorage-stratis Lab
2) open an SSH session to servera as student.
3) Install the stratisd and stratis-cli packages using the yum command.
1
ITIM(2CSE605) Yagna(21162101020)
4) Activate the stratisd service using the systemctl command.
Create a Stratis pool named stratispool1 using the stratis pool create command.
Verify the availability of stratispool1 using the stratis pool list command.
2
ITIM(2CSE605) Yagna(21162101020)
5)Add the block device /dev/vdc to stratispool1 using the stratis pool add-data
command.
Verify the size of stratispool1 using the stratis pool list command.
Verify the block devices that are currently members of stratispool1 using the stratis
blockdev list command.
6)
Create the thin-provisioned file system stratis-filesystem1 on stratispool1 using
the stratis filesystem create command. It may take up to a minute for the
command to complete.
Verify the availability of stratis-filesystem1 using the stratis filesystem list
command
Create a directory named /stratisvol using the mkdir command.
Mount stratis-filesystem1 on /stratisvol using the mount command.
3
ITIM(2CSE605) Yagna(21162101020)
Verify that the stratis-filesystem1 volume is mounted on /stratisvol using the
mount command.
Create the text file /stratisvol/file1 using the echo command.
4
ITIM(2CSE605) Yagna(21162101020)
7)Verify that the thin-provisioned file system stratis-filesystem1 dynamically grows
as the data on the file system grows.
Create a 2 GiB file on stratis-filesystem1 using the dd command. It may take up to a
minute for the command to complete.
View the current usage of stratis-filesystem1 using the stratis filesystem list
command.
8) Create a snapshot of stratis-filesystem1 named stratis-filesystem1-snap.
The snapshot will provide you with access to any file that is deleted from
stratis-filesystem1.Create a snapshot of stratis-filesystem1 using the stratis
filesystem snapshot command.
5
ITIM(2CSE605) Yagna(21162101020)
9)Unmount /stratisvol and /stratisvol-snap using the umount command.
10)Remove the thin-provisioned file system stratis-filesystem1 and its snapshot
stratis-filesystem1-snap from the system.
Destroy stratis-filesystem1-snap using the stratis filesystem destroy
command.
Destroy stratis-filesystem1 using the stratis filesystem destroy command.


prac16:
//install samba samba-client samba-common
yum install samba samba-client samba-common
//make dir /samba and add hello.txt inside it
//Enter into /etc/samba/smb.conf
[global]
	workgroup = WORKGROUP
[sambashare]
	read only = no
	path = /samba
	browsable = yes
	valid users = smbl
	write list = smbl
//make sure smbl
useradd smbl
//assign smb password
smbpasswd -a smbl
//give ownership and permission
chown -R smbl:smbl /samba
chmod -R 777 /samba
//disable firewall protection
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload
//start the service
systemctl enable smb.service
systemctl start smb.service
//access using smbclient
smbclient //172.25.250.9/sambashare -U smbl