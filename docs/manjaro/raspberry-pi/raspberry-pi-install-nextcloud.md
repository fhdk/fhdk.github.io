---
title: 'Install NextCloud on Raspberry Pi'
date: '07:32 14-10-2022'
taxonomy:
    category:
        - docs
publish_date: '16-10-2022 12:11'
---

**Difficulty: ★★☆☆☆**

NextCloud is an open-source collaboration tool which can be selfhosted.

## Prerequisites
For this you need a server device - e.g. Raspberry Pi 4B using WiFi

 - https://forum.manjaro.org/search?q=raspberry%20pi%20wifi

This document was written using a Raspberry PI 4B 4GB using a 16GB microSD.

To install NextCloud you need to prepare the following components.

1. Apache or Nginx
2. PHP
3. Sqlite or MariaDB

The most widely used setup is called **LAMP** and is an acronym for **L**inux, **A**packe, **M**ariaDB/MYSQL, **P**HP/Perl/Python.

Refer to the following document to setup your **LAMP** stack and return here when ready as this document assumes you have the phpMyAdmin webapp available on the system.

* https://forum.manjaro.org/t/howto-install-apache-mariadb-mysql-php-lamp/13000

## Important
Always ensure your system is up-to-date before adding new packages
```
sudo pacman -Syu
```

NextCloud requires any one of these locales to be enabled and generated

* en_US.UTF-8 
* fr_FR.UTF-8
* es_ES.UTF-8 
* de_DE.UTF-8 
* ru_RU.UTF-8 
* pt_BR.UTF-8 
* it_IT.UTF-8 
* ja_JP.UTF-8 
* zh_CN.UTF-8

Ensure you have at least **en_US.UTF-8** locale enabled and generated.
```
echo en_US.UTF-8 | sudo tee -a /etc/locale.gen
```

Generate locale
```
sudo locale-gen
```

## Installing NextCloud

This document will focus on the Web installer provided as a NextCloud community project nonetheless you have the option to use the **nextcloud** package from the repo along with the extensive documentation at https://wiki.archlinux.org/title/Nextcloud

## NextCloud Web installer

The Web installer [https://download.nextcloud.com/server/installer/setup-nextcloud.php][1] is a php script you download and place in the root of the web service.

If you followed the **LAMP** stack your http root lives at `/srv/http`.

Open a terminal and navigate to the http root folder
```text
cd /srv/http
```

Use **curl** to fetch the script
```text
sudo curl -O https://download.nextcloud.com/server/installer/setup-nextcloud.php
```

Best practice is to go over the content and ensure it is legit
```text
less setup-nextcloud.php
```

## Setup Wiard
Before you continue the setup process - install required php packages.
```
sudo pacman -S php-gd  php-imagick php-intl
```
Depending on your actual requirements and the NextCloud features you want to make use of you will need to install other packages. A reference list can be viewed by visiting the Arch Community Repo package [https://archlinux.org/packages/community/any/nextcloud/][2]

The NextCloud developers recommend the use of MariaDB/MYSQL - but you can use sqlite for test and development. If you choose to use sqlite - you will need the php-sqlite package as well
```
sudo pacman -Syu php-sqlite
```

When you have installed the packages edit your system's php.ini
```
sudo micro /etc/php/php.ini
```
Search and locate **extension=bcmath** or scroll down to the extensions list and enable
```
[...]
extension=bcmath
extension=bz2
extension=exif
extension=gd
extension=iconv
extension=imagick
extension=intl
extension=pdo_mysql
extension=pdo_sqlite
```

Search and locate **memory_limit**
```
[...]
memory_limit = 512M
[...]
```

Search and locate **date.timezone**, uncomment and set it to your timezone e.g.
```
date.timezone = Europe/Copenhagen
```

Save the file and close it then restart your http service
```
sudo systemctl restart httpd
```

Ensure the http user is set as owner of the `/srv/http` folder and content.
```
sudo chown http:http /srv/http -R
```

Load the installer in your browser by navigating to your server's ip or name
```text
http://172.30.30.128/setup-nextcloud.php
```

When you click the **Next** button dependencies are verified and if they are found click **Next** to install in a subfolder named **nextcloud**. Replace nextcloud with a **.** if you want NextCloud in the root of the webserver.

Be patient - after a while you will be greeted with a **Success** message - click **Next**.

## Setting up database

This document is only intended to wet your feet wet so SQLite is just fine but for production you should use MariaDB. For productdion use you should refer to the [NextCloud documentation][3]

To create a database for your nextcloud app we will use phpMyAdmin. Navigate your browser to the web server's phpmyadmin url
```
http://ip.x.y.z/phpmyadmin
```

Login using your mariadb root user and the password created earlier.

* Click the **User accounts** tab, then **Add user account**
* Click the button next **Generate password** labelled **Generate**
* Copy the password from the textbox and paste it somewhere convenient
* Tick the box next to **Create database with same name and grant all privileges**
* Scroll to the bottom and click **Go**

Go back to the NextCloud database setup page

* Fill in your desired administrative username
* Create a strong password
* Click the **Storage & Database** dropdown and select **MySQL/MariaDB**
* Fill in database user **nextcloud**
* Fill in database passwd - copy/paste the passwd created above
* Fill in the database name **nextcloud**
* Don't change the host
* Click **Install** - have patience
* The script may redirect you to a non-existing page on nextcloud.com - ignore that

Navigate to your own nextcloud address
```
http://ip.x.y.z/nextcloud
```

## You are almost done
The rest is up to you - please remember to ensure your MariaDB engine is configured as recommended at [NextCloud documetnation][3]


Crossposted at [Manjaro Forum][4]

[1]: https://download.nextcloud.com/server/installer/setup-nextcloud.php
[2]: https://archlinux.org/packages/community/any/nextcloud/
[3]: https://docs.nextcloud.com/server/latest/admin_manual/configuration_database/linux_database_configuration.html
[4]: https://forum.manjaro.org/t/root-tip-how-to-install-nextcloud-using-nextcloud-web-installer/126683