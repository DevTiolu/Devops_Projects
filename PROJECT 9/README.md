# Implementing Wordpress Website With LVM Storage Management

## Introduction

Certainly! WordPress is a popular open-source content management system (CMS) and website creation tool written in PHP. It is widely used for building websites, blogs, and online stores due to its flexibility, ease of use, and extensive ecosystem of themes and plugins.

What Is LVM Storage?
LVM stands for Logical Volume Manager, which is a technology used on Linux systems to manage storage devices and provide advanced features for managing disk space, such as logical volumes, snapshots, and volume resizing. LVM allows for more flexible and dynamic management of disk partitions compared to traditional partitioning schemes.
LVM provides a flexible and efficient way to manage storage resources on Linux systems, offering advanced features that enhance data management and system flexibility. It is particularly useful in environments where storage requirements are dynamic and may change over time.
Key Components of LVM:

Physical Volumes (PVs):These are physical storage devices (e.g., hard drives, SSDs) or partitions that are initialized for use with LVM. Each physical volume is assigned a unique identifier.

Volume Groups (VGs):These are created by combining one or more physical volumes. A volume group represents a pool of storage that can be divided into logical volumes.

Logical Volumes (LVs):These are virtual partitions created from free space within volume groups. They can be thought of as equivalent to partitions in traditional disk partitioning schemes (e.g., ext4 partitions). Logical volumes can be resized dynamically without requiring downtime.

Advantages of LVM:

Flexibility: LVM allows administrators to resize logical volumes dynamically, enabling easy allocation and reallocation of disk space without the need to repartition disks.

Volume Management: LVM simplifies storage management by abstracting physical storage devices into logical volumes, providing a unified view of storage across multiple disks.

Snapshots: LVM supports creating snapshots, which are read-only copies of a logical volume at a specific point in time. Snapshots are useful for backups and testing without affecting the original data.

Striping and Mirroring: LVM supports RAID-like functionality such as striping (to improve performance) and mirroring (to provide data redundancy) at the volume group level.

Common LVM Commands:

pvcreate: Initializes a physical volume for use with LVM. vgcreate: Creates a volume group by combining one or more physical volumes. lvcreate: Creates a logical volume within a volume group. lvresize: Resizes an existing logical volume. lvextend / lvreduce: Extends or reduces the size of a logical volume. lvdisplay: Displays information about logical volumes.

Use Cases for LVM:

Server Virtualization: LVM is commonly used in virtualization environments to manage disk resources for virtual machines.

Database Servers: LVM allows for easy resizing of partitions used by databases to accommodate changing storage requirements.

Backup Systems: LVM snapshots can be used for creating consistent backups of data without interrupting ongoing operations.

Implementing Wordpress Web Solution
Three-tier Architecture Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

Your 3-Tier Setup

A Laptop or PC to serve as a client
An EC2 Linux Server as a web server (This is where you will install WordPress)
An EC2 Linux server as a database (DB) serve
Create an AWS instance using RedHat Distribution

The EC2 instance will serve as a Web Server, create 3 volumes in the same AZ as the Web Srver EC2, each of 10GB.

Attach the three volumes one by one to your Webserver EC2 instance

alt text alt text

Open up the Linux terminal to begin configuration

Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdp.

alt text alt text

Use df -h command to see all mounts and free space on your server
alt text

Use gdisk utility to create a single partition on each of the 3 disks
sudo gdisk /dev/xvdf

alt text alt text alt text

Use lsblk utility to view the newly configured partition on each of the 3 disks.
alt text

Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.
alt text alt text

Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.
alt text

Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1

alt text

verify that VG has been created
alt text

Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

sudo lvcreate -n apps-lv -L 14G webdata-vg sudo lvcreate -n logs-lv -L 14G webdata-vg

alt text

Verify that your Logical Volume has been created successfully by running sudo lvs
alt text

Verify the entire setup
alt text alt text

Use mkfs.ext4 to format the logical volumes with ext4 filesystem
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
alt text

Create /var/www/html directory to store website files
sudo mkdir -p /var/www/html

Create /home/recovery/logs to store backup of log data
sudo mkdir -p /home/recovery/logs

Mount /var/www/html on apps-lv logical volume
sudo mount /dev/webdata-vg/apps-lv /var/www/html/

Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
sudo rsync -av /var/log/. /home/recovery/logs/

alt text

Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important)
sudo mount /dev/webdata-vg/logs-lv /var/log

Restore log files back into /var/log directory
sudo rsync -av /home/recovery/logs/. /var/log

alt text

UPDATE THE /ETC/FSTAB FILE
The UUID of the device will be used to update the /etc/fstab file;

sudo blkid
alt text

sudo nano /etc/fstab
Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.

alt text

Test the configuration and reload the daemon

 sudo mount -a
 sudo systemctl daemon-reload
alt text

Verify your setup by running df -h, output must look like this:

alt text

Prepare the Database Server
Launch a second RedHat EC2 instance that will have a role – ‘DB Server’ Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to var/www/db directory instead of /var/www/html/.

Install WordPress On EC2 WebServer
Update the repository

sudo yum -y update

alt text

Install wget, Apache and it’s dependencies

sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

alt text

Start Apache

sudo systemctl enable httpd
sudo systemctl start httpd
alt text

To install PHP and it’s depemdencies

sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
alt text

Restart Apache

sudo systemctl restart httpd

Download Wrdpress and Copy Wordpress to var/www/html

mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
sudocp -R wordpress /var/www/html/
Configure SELinux Policies

sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
Install MySQL On EC2 DB Server
sudo yum update sudo yum install mysql-server -y

alt text

Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:

sudo systemctl restart mysqld
sudo systemctl enable mysqld
Configure DB To Work With WordPress
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
alt text

Configure WordPress To Connect To Remote Database. Edit inbound rule and open port 3306 on database server and allow connection from only our database server.

alt text

Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client

sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
alt text

Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

Change permissions and configuration so Apache could use WordPress:

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

Verify Database Credentials
Check the wp-config.php file in your WordPress webserver to ensure that the database credentials are correct.

sudo nano /var/www/html/wordpress/wp-config.php

Verify the values below

define('DB_NAME', 'wordpress');
define('DB_USER', 'myuser');
define('DB_PASSWORD', 'mypass');
define('DB_HOST', 'database_private_ipaddress');
Try to access from your browser the link to your WordPress http:///wordpress/

alt text alt text

alt text alt text

Conclusion In conclusion, this project has provided a comprehensive guide to implementing a WordPress website with LVM storage management. Key takeaways include:

Understanding the fundamentals of LVM, such as physical volumes, volume groups, and logical volumes. Recognizing the benefits of LVM, including dynamic resizing, striping, mirroring, and snapshots.