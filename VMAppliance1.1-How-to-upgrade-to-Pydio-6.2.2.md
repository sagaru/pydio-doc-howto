The repository of Pyio 6.2.2 has been available in our server. After execution to install or upgradeYou will find in this how to the steps to update your Pydio to latest version by using package management of system such as yum or apt


# Cent OS 6 / Redhat 6

On centos, it requires to use some extra repositories for php54 and its dependencies. Execute following command to add such repositories

`rpm -ivh https://www.softwarecollections.org/en/scls/rhscl/php54/epel-6-x86_64/download/rhscl-php54-epel-6-x86_64.noarch.rpm`

`rpm -ivh https://www.softwarecollections.org/en/scls/remi/php54more/epel-6-x86_64/download/remi-php54more-epel-6-x86_64.noarch.rpm`


### Install pydio repositories:

**Community version**

`wget https://download.pydio.com/pub/linux/centos/6/pydio-release-1-2.el6.noarch.rpm`

`rpm -i pydio-release-1-2.el6.noarch.rpm`

**Enterprise version**

`wget https://API_KEY:API_SECRET@download.pydio.com/auth/linux/centos/6/x86_64/pydio-enterprise-release-1-2.el6.noarch.rpm`

> Note: Enterprise version requires two repositories.

where you would replace API_KEY / API_SECRET by the values retrieved from your pydio.com account.

`rpm -i pydio-enterprise-release-1-2.el6.noarch.rpm`

`yum update`

The following commands are applied for installation of new pydio and for doing upgrade from Pydio 6.0 to latest version

`yum install pydio-all`

`yum install pydio-enterprise`

Activate php54

`source /opt/rh/php54/enable`

Disable old php (5.3)

`mv /etc/httpd/conf.d/php.conf /etc/httpd/conf.d/php.conf.old`

> Note: the php.ini file is located in: /opt/rh/php54/root/etc/php.ini

Restart apache service

`service httpd restart`

**Manually upgrade Database**
Use this command to connect to MySQL database:

`mysql -u pydio -p`

> Tip: You can get the current password for *pydio* account in mysql by executing following command in your server:
`sudo cat /var/lib/pydio/plugins/boot.conf/bootstrap.json | grep mysql_password`

Then execute the script in this link to upgrade your database:

https://raw.githubusercontent.com/pydio/pydio-core/develop/dist/php/6.2.0.mysql


# Virtual Appliance 1.1.0 or Debian 7

Virtual Appliance 1.1.0 is built base on Debian 7.4 (Wheezy) 32 bits. There are several step to upgrade Pydio to latest version in using official repo

## Step 1: Preparation before upgrade
1.1. Add repo source.

Firstly,  install some package
`sudo apt-get install apt-transport-https`

`sudo apt-get install debian-keyring debian-archive-keyring`

Secondly, please disable old pydio source if it existes in /etc/apt/source.list of your server. Then add the source according to the version you are going to install:

*repo source pydio community*:

`deb https://download.pydio.com/pub/linux/debian/ wheezy-backports main`

*repo source pydio enterprise*:

`deb https://API_KEY:API_SECRET@download.pydio.com/auth/linux/debian/ wheezy-backports main non-free`

> Note: Enterprise version requires two repositories.

where you would replace API_KEY / API_SECRET by the values retrieved from your pydio.com account.

- install debian-keyring debian-archive-keyring and https-transport

If you are connecting pydio with samba share in using meta.mount plugins, you may add the repo source for *udevil* to replace *sudo*, which was disabled in PYdio since version 6.2

*repo source for udevil*

`deb http://http.debian.net/debian wheezy-backports main`

**Manually update DB**

Use this command to connect to MySQL database:

`mysql -u pydio -p`

> Tip: You can get the current password for *pydio* account in mysql by executing following command in your VM:
`sudo cat /var/lib/pydio/data/plugins/boot.conf/bootstrap.json | grep mysql_password`

Then execute the script in this link to upgrade your database:

https://raw.githubusercontent.com/pydio/pydio-core/develop/dist/php/6.2.0.mysql

and this script if you want to upgrade to Enterprise version
<pre>
CREATE TABLE IF NOT EXISTS `ajxp_session` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `session_name` varchar(50) NOT NULL,
  `session_id` varchar(60) NOT NULL,
  `user` varchar(255) NOT NULL,
  `crt_repository` varchar(32) NOT NULL,
  `ip` varchar(45) NOT NULL,
  `ua` mediumtext NOT NULL,
  `status` int(11) NOT NULL DEFAULT '1',
  `updated` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `session_name_2` (`session_name`,`session_id`),
  KEY `session_user` (`user`),
  KEY `session_status` (`status`),
  KEY `updated` (`updated`)
) DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
</pre>

## Step 2: Upgrade

Commands:
- `apt-get update`
- `apt-get install pydio-all`
- `apt-get install pydio-enterprise`
- `apt-get install udevil`

During the installation, you may be asked for replacement some old files:

PHP version end-of-line alert

![php_alert](images/upgrades/VMAppliance1.1-How-to-upgrade-to-Pydio-6.2.2/php_alert.png)

Select (press) `q`key to pass

![phpini_replacement](images/upgrades/VMAppliance1.1-How-to-upgrade-to-Pydio-6.2.2/replacement_phpini.png)

select **keep the local version currently installed** to keep old version of /etc/php5/cli/php.ini and /etc/php5/apache2/php.ini

![replacement_pydio_files](images/upgrades/VMAppliance1.1-How-to-upgrade-to-Pydio-6.2.2/replacement_pydio_etc.png)

![replacement_pydio_files](images/upgrades/VMAppliance1.1-How-to-upgrade-to-Pydio-6.2.2/replacement_pydio_etc2.png)

Select `Y` to use new version files of Pydio


## Step 3: After upgrade
1. Verify public links
2. Verify /api
3. Verify php.ini:In new repo, ioncube is installed by separated package then it could be duplicated if we declare two times in php.ini and apache won't be able start. In this case, please go to /etc/php5/apache2(cli)/php.ini and remove this extension
4. Reedit bootstrap_repositories.php: In Pydio Appliance 1.1.0, two default workspaces "My Files" and "common files" is converted and saved in database to be editable from GUI. Therefore, after installation of Pydio, you will see they are duplicated. Open /etc/pydio/bootstrap_repositories.php and comment (using /* */) or delete two workspaces.
5. Verify pydio.conf of apache:Execute following command to disable pydio.conf (created after installation)

 `sudo a2disconf pydio.conf`

 `sudo service apache2 restart`

6. Enable some inactive plugins: There are some plugins in Pydio are set to disable by default such as access.smb. Please go to the settings and enable it.
7. Configure udevil:
Due to security reason, we've decided to remove `sudo` from meta.mount plugins since Pydio 6.2. There are many user are using Pydio 6.1 or older version with meta.mount plugins activated.

For more information about udevil: https://ignorantguru.github.io/udevil/

– Install udevil package
`sudo apt-get install udevil`

– udevil configure file: /etc/udevil/udevil.conf

There are some important parameters:

allowed_types = $KNOWN_FILESYSTEMS, file, smbfs, nfs
>Note: be careful space after ,

allowed_users = *

allowed_groups = *

allowed_media_dirs = /media, /media/$USER, /run/media/$USER, /var/lib/pydio/data/, /media/test

allowed_devices = /dev/*

allowed_networks = *

forbidden_networks =


– Testing by using command to mount in terminal
udevil mount smb://pydio@192.168.0.3/share /media/test/abc
then:
udevil umount smb://pydio@192.168.0.3/share /media/test/abc

Reconfigure in meta.mount

![Configure meta.mount with udevil](images/upgrades/VMAppliance1.1-How-to-upgrade-to-Pydio-6.2.2/metamountwithudevil.png)


Attention: udevil does not support passing password via variable environment



















# Ubuntu 14.04 (trusty)

## Step 1: Preparation before upgrade
1.1. Add repo source.

Firstly,  install some package
`sudo apt-get install apt-transport-https`

Secondly, please disable old pydio source if it existes in /etc/apt/source.list of your server. Then add the source according to the version you are going to install:

*repo source pydio community*:

`deb https://download.pydio.com/pub/linux/debian/ trusty main`

*repo source pydio enterprise*:

`deb https://API_KEY:API_SECRET@download.pydio.com/auth/linux/debian/ trusty non-free`

> Note: Enterprise version requires two repositories.

where you would replace API_KEY / API_SECRET by the values retrieved from your pydio.com account.

**Manually update DB**

Use this command to connect to MySQL database:

`mysql -u pydio -p`

> Tip: You can get the current password for *pydio* account in mysql by executing following command in your VM:
`sudo cat /var/lib/pydio/data/plugins/boot.conf/bootstrap.json | grep mysql_password`

Then execute the script in this link to upgrade your database:

https://raw.githubusercontent.com/pydio/pydio-core/develop/dist/php/6.2.0.mysql

and this script if you want to upgrade to Enterprise version
<pre>
CREATE TABLE IF NOT EXISTS `ajxp_session` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `session_name` varchar(50) NOT NULL,
  `session_id` varchar(60) NOT NULL,
  `user` varchar(255) NOT NULL,
  `crt_repository` varchar(32) NOT NULL,
  `ip` varchar(45) NOT NULL,
  `ua` mediumtext NOT NULL,
  `status` int(11) NOT NULL DEFAULT '1',
  `updated` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `session_name_2` (`session_name`,`session_id`),
  KEY `session_user` (`user`),
  KEY `session_status` (`status`),
  KEY `updated` (`updated`)
) DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
</pre>

## Step 2: Upgrade

Commands:
- `apt-get update`
- `apt-get install pydio-all`
- `apt-get install pydio-enterprise`

## Step 3: After upgrade
1. Execute following command to disable pydio.conf (created after installation)

 `sudo a2disconf pydio.conf`

 `sudo service apache2 restart`

2. Go to Pydio >> Settings >> Application Core:  DOWNLOAD URL: /pydio/data/public (default value)
