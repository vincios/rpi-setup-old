# General Setup

**NOTE:** Please read before [First operations](#first-operations).

Index
- [General Setup](#general-setup)
  - [First operations](#first-operations)
  - [VNC "cannot currently show the desktop" in headless mode](#vnc-cannot-currently-show-the-desktop-in-headless-mode)
  - [AutoMount Nas folders](#automount-nas-folders)
  - [Samba shares](#samba-shares)
  - [Duckdns cron configuration](#duckdns-cron-configuration)
  - [Plex Media Server](#plex-media-server)
  - [Build TOR](#build-tor)
  - [Run BridTools](#run-bridtools)
  - [Install jDownloader in headless mode](#install-jdownloader-in-headless-mode)
  - [JDownloader RAR5 support](#jdownloader-rar5-support)
  - [Update Node and npm](#update-node-and-npm)
  - [Install Deluge torrent client with web interface](#install-deluge-torrent-client-with-web-interface)
  - [Install Transmission](#install-transmission)
  - [Install LAMP software](#install-lamp-software)
    - [Add xdebug](#add-xdebug)
    - [Start, Stop and Restart apache2 service](#start-stop-and-restart-apache2-service)
  - [Install Ampache](#install-ampache)
  - [Deemix](#deemix)
    - [Notes](#notes)
  - [Install Python 3](#install-python-3)
  - [Install Pi-hole](#install-pi-hole)
- [Useful commands](#useful-commands)
  - [List active processes](#list-active-processes)
- [.bash\_aliases](#bash_aliases)

## First operations
Tutorials in this document assumes that you have first followed this paragraph. <br>
**Please make sure to follow this steps before all other tutorials!**

1. Create an `~/Apps` folder

    ```sh
	$ mkdir ~/Apps
	```

2. Create a `~/.logs` folder

    ```sh
	$ mkdir ~/.logs
	```
	
3. Create a `~/.bash_aliases` file
    - `$ nano ~/.bash_aliases`
	- Paste these lines
	
	    ```bash
		alias ll='ls -l'
        alias la='ls -la'
		```
		
	- Exit and save
	
	- `$ source ~/.bashrc`
	
## VNC "cannot currently show the desktop" in headless mode
Run `raspi-config` and change screen resolution to 1920x1080

## AutoMount Nas folders
From [here](http://timlehr.com/auto-mount-samba-cifs-shares-via-fstab-on-linux/).

- Install dependencies

    ```bash
    $ sudo apt update
    $ sudo apt install cifs-utils
    ```

- Create mountpoints into the `/media` folder

    ```bash
    $ sudo mkdir /media/dnas
    $ sudo mkdir /media/qnas
    $ sudo mkdir /media/qnas/Media
    $ sudo mkdir /media/qnas/Download
    ```

- Create credentials files, in a `~/.credentials` folder (create if not exists)
    One for each network share (if it have different credentials)

    - `nano ~/.credentials/.qnascredentials`

        ```bash
        user=<YOUR-USER>
        password=<YOUR-PASSWORD>
        domain=WORKGROUP
        ```

    - Give access only to the user

        ```bash
        $ chmod 600 ~/.credentials/.qnascredentials
        ```

    - Repeat for each network share you want to login

- Run `sudo nano /etc/fstab` and add these lines (changes paths as done in point 2)

    ``` 
    //192.168.1.200/Volume_1    /media/dnas/            cifs    credentials=/home/raspi/.credentials/.dnascredentials,uid=raspi,gid=raspi,iocharset=utf8,file_mode=0755,dir_mode=0777,noperm,vers=1.0	0	0
    //192.168.1.210/Multimedia  /media/qnas/Media/      cifs    credentials=/home/raspi/.credentials/.qnascredentials,uid=raspi,gid=raspi,iocharset=utf8,file_mode=0755,dir_mode=0777,noperm			0	0
    //192.168.1.210/Download    /media/qnas/Download/   cifs    credentials=/home/raspi/.credentials/.qnascredentials,uid=raspi,gid=raspi,iocharset=utf8,file_mode=077,dir_mode=0777,noperm			0	0
    ```

- Run `raspi-config` and enable "Wait for Network at Boot" under "Boot options"
- Check if OK with `sudo mount -a`


## Samba shares
- `sudo apt-get install samba`

- `sudo nano /etc/samba/smb.conf`

Samba share user folder with some security restrictions (See "[homes]" section in *smb.conf*). So, instead of change this configuration we add a new one. 

- Add to bottom:

```
[PiShare]
   comment = Pi Share
   path = /home/pi
   browseable = yes
   writeable = yes
   only guest = no
   read only = no
   create mask = 0740
   dierectory mask = 0750
   public = yes
```

- Choose a smb password for pi user

`sudo smbpasswd -a pi`

NB: if the network folder is non writeable, try this command `sudo pdbedit -a -u pi`

## Duckdns cron configuration
See [here](https://www.duckdns.org/install.jsp?tab=pi&domain=vncs10). BUT:
- Change `duck.sh` as follows: (Change domains and token in update URL if necessary)

``` sh
timestamp() {
  date +"%Y-%m-%d %H:%M:%S"
}

echo url="https://www.duckdns.org/update?domains=vncs10&token=b6e9eba3-1b42-4c66-898a-0d5204833f36&ip=" | curl -k -o /home/pi/duckdns/log.log -K -
echo " | Last run: $(timestamp)" >> /home/pi/duckdns/log.log
```

- Change crontab string as follows:

``` sh 
*/5 * * * * /home/pi/duckdns/duck.sh >/dev/null 2>&1
```


## Plex Media Server
See [here](https://pimylifeup.com/raspberry-pi-plex-server/).


## Build TOR
**NEW** (but not tested yet):

Build from git. See [here](https://tor.stackexchange.com/questions/75/how-can-i-install-tor-from-the-source-code-in-the-git-repository).

**OLD** (tested):

- `sudo apt install libevent-dev`

- Download tor source tar form [here](https://www.torproject.org/it/download/tor/)

- `tar xzf tor-0.4.0.5.tar.gz; cd tor-0.4.0.5`

- ` ./configure && make`

- `sudo make install`


Now install obfs4proxy (**for both OLD and NEW**):

- `sudo apt install obfs4proxy`


## Run BridTools
First install *TOR* and *obfs4proxy* (See [Build Tor](#build-tor)).

**NEW**:
- Simply copy scripts folder to Pi and run `./install.sh` inside it
- In `config.ini` file add these lines in *BASE* section:

```
TOR_DATA_DIR = /usr/local/share/tor
TOR_DIR = /usr/local/bin
TOR_PLUG_DIR = /usr/bin
```

**OLD**:

- Copy scripts folder to Pi and browse this folder

- `python3 -m venv ./venv`

- `source venv/bin/activate`

- `pip install -r requirements.txt`

- In `config.ini` file add these lines in *BASE* section:

```
TOR_DATA_DIR = /usr/local/share/tor
TOR_DIR = /usr/local/bin
TOR_PLUG_DIR = /usr/bin
```


## Install jDownloader in headless mode
See [here](https://support.jdownloader.org/Knowledgebase/Article/View/52/0/install-jdownloader-on-nas-and-embedded-devices). Then
- Run in headless mode with `java -jar JDownloader.jar &` 

OR, to hide any output from the terminal

- Run in headless mode with `java -Djava.awt.headless=true -jar JDownloader.jar >/dev/null 2>/dev/null &`

To open the GUI:
- ` wget -O /home/pi/[JD_Install_dir]/jDownloader.png http://jdownloader.org/_media/knowledge/wiki/jdownloader.png`

- Go to desktop

- `nano jDownloader.desktop`

- Paste this:

``` bash
[Desktop Entry]
Encoding=UTF-8
Name=jDownloader2
Comment=jDownloader2
Exec=bash -c "java -jar /home/pi/[JD_install_dir]/JDownloader.jar"
Icon=/home/pi/[JD_install_dir]/jDownloader.png
Type=Application
Categories=GTK;Utility;
```

- `chmod +x jDownloader.desktop`


## JDownloader RAR5 support
**NB**: You colud download the precompiled SevenZipBindings jar files from [here](https://board.jdownloader.org/showpost.php?p=455292&postcount=583) and copy them in *[JD_Install_dir]/libs*
 **OR** compile it manually (better):

From [here](https://www.ixsystems.com/community/threads/guide-jdownloader2-in-11-1-release-iocage-with-rar5-working.74073/) (or [here](https://board.jdownloader.org/showpost.php?p=446708&postcount=465)):

- Install cmake and openjdk8-jdk

``` bash
$ sudo apt-get install cmake
$ sudo apt-get install openjdk8-jdk
```

- Go to JD folder and make folders:

``` bash
$ mkdir rar5
$ cd rar5
```

- Clone repository

``` bash
$ git clone https://github.com/borisbrodski/sevenzipjbinding.git sevenzipbinding
```

**NB**: would be better to choose one branch between `bind_16.02` and `migrate-to-15.09-try2` but it should work also with `master` branch.
So first try with `master` then, in case of failure, try `bind_16.02` and, as last chance, `migrate-to-15.09-try2`.

You can change branch with the `git checkout [branch_name]` command.

- Run CMake

``` bash
$ cmake . -DJAVA_JDK=/usr/lib/jvm/java-8-openjdk-armhf`
$ make
$ make package
```

- Move jar library to JD lib folder

``` bash
$ unzip sevenzipjbinding-16.02-2.01beta-Linux-arm.zip
$ cd sevenzipjbinding-16.02-2.01beta-Linux-arm/lib/
$ mv sevenzipjbinding.jar [JD_install_dir]/libs/sevenzipjbinding1509.jar
$ mv sevenzipjbinding-Linux-arm.jar [JD_Install_dir]/libs/sevenzipjbinding1509LinuxArmVersion.jar
```

**NB**: If you change branch, rename commands above with correct version numbers. But **DON'T** change destination filenames (*sevenzipjbinding1509.jar* and *sevenzipjbinding1509LinuxArmVersion.jar*)


## Update Node and npm
- Remove old node

``` bash
$ sudo npm uninstall -g npm
$ sudo apt remove npm
$ sudo apt-get autoremove
```

- Install new version (From [here](https://github.com/nodesource/distributions/blob/master/README.md))

``` bash
$ curl -sL https://deb.nodesource.com/setup_13.x | sudo -E bash -
$ sudo apt-get install -y nodejs
```

## Install Deluge torrent client with web interface
`NB: Instead of Deluge, install Transmission. This section will remain as refrence`. See [here](#install-transmission).<br>
See [here](https://pimylifeup.com/raspberry-pi-deluge).<br>
Do not follow the last part ("Setting up Deluge as a service").<br>
Don't forget to add the alias in the `.bash_aliases` file.


## Install Transmission
```bash
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install transmission-cli transmission-common transmission-daemon
$ sudo /etc/init.d/transmission-daemon stop
```
- Edit configuation file
```bash 
$ sudo nano /etc/transmission-daemon/settings.json
```
- **Edit** this entries as follow:
```json
{
    "download-dir": "/media/nas/home_2/DOWNLOADS/torrent",
    "incomplete-dir": "/media/nas/home_2/DOWNLOADS/torrent/parts",
    "incomplete-dir-enabled": true,
    "rpc-enabled": true,
    "rpc-username": "transmission",
    "rpc-password": "transmission",
    "rpc-whitelist-enabled": false,
    "umask": 2,
}
```
- Add this entries at bottom
```json
    "watch-dir": "/media/nas/home/Pi/torrents",
    "watch-dir-enabled": true
```

- Add user `pi` to the group `debian-transmission`
```bash
$ sudo usermod -a -G debian-transmission pi
```

- Add this lines in `.bash_aliases`
```
alias t-start='sudo service transmission-daemon start'
alias t-stop='sudo service transmission-daemon stop'
alias t-list='transmission-remote -n 'transmission:transmission' -l'
alias t-basicstats='transmission-remote -n 'transmission:transmission' -st'
alias t-fullstats='transmission-remote -n 'transmission:transmission' -si'
```
- Reboot


## Install LAMP software
See [here](https://howtoraspberrypi.com/how-to-install-web-server-raspberry-pi-lamp).<br>
**NB: To install phpMyAdmin**, download latest version from the [phpMyAdmin homepage](https://www.phpmyadmin.net/), unzip it, move to `/var/www/html/`, rename the folder to `phpmyadmin` and change permissions to 755. Then

- In phpmyadmin folder, create (if not exists) a `config.inc.php` file and edit it
``` bash
$ cp config.sample.inc.php config.inc.php #only if not exists
$ nano config.inc.php
```
- Find the line
``` php
$cfg['blowfish_secret'] = ''; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
```
- Insert a 32 random characters string between quotes

### Add xdebug
Paste `php_info()` (or `php -i` in command line) result into the official [wizard](https://xdebug.org/wizard), update also the `php.ini` file mentioned into `php_info()` page, at **Loaded Configuration File** line.

### Start, Stop and Restart apache2 service
```
## Start command ##
$ systemctl start apache2.service
## Stop command ##
$ systemctl stop apache2.service
## Restart command ##
$ systemctl restart apache2.service
```

## Install Ampache
**NB**: This steps are taken and detailed form the Ampache's [official installation wiki](https://github.com/ampache/ampache/wiki/Installation). Refer to that page for updated installation steps.<br>

**NB2**: this guide assume that Apache uses the default `/var/www/html/` public folder. Change the code according to your web server public html folder location.<br>

- First install *LAMP* (See [Install LAMP software](#install-lamp-software)).

- Download the Ampache application from git to an `ampache` folder on the web server

``` bash
$ cd /var/www/html/
$ mkdir ampache
$ git clone -b master https://github.com/ampache/ampache.git ampache
```

- Install Composer. Refer to the [official page](https://getcomposer.org/download/) **BUT** on the third command, run instead of `php composer-setup.php`

``` bash
$ sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```
**NB**: You can run above commands from any folder but maybe is better to avoid the Ampache folder.<br>

- Run composer dependencies installation from Ampache folder

``` bash
$ cd /var/www/html/ampache
$ composer install --prefer-source --no-interaction
```

Now we have to enable the url rewriting functionality. Next steps are taken from [here](https://www.digitalocean.com/community/tutorials/how-to-rewrite-urls-with-mod_rewrite-for-apache-on-ubuntu-16-04/). <br>
1. Enable mod_rewrite on Apache

``` bash
$ sudo a2enmod rewrite
$ sudo systemctl restart apache2
```
To check if module is enabled launch a `phpinfo()` page and search `mod_rewrite` into the **Loaded Modules** section. <br>

2. By default, Apache prohibits using an .htaccess file to apply rewrite rules, so first you need to allow changes to the file `000-default.conf`
``` bash
$ sudo nano /etc/apache2/sites-available/000-default.conf
```
3. Inside the `<VirtualHost *:80>` block insert
```
# Enable .htaccess files for /var/www/html and subfolders
<Directory /var/www/html>
       Options Indexes FollowSymLinks MultiViews
       AllowOverride All
       Require all granted
</Directory>
```

This configuration will enable .haccess files for the default vhost (VirtualHost) located in /var/www/html folder and all subfolders.<br>
**TODO**: Investigate whether it would be better to create a new vhost.

4. Restart server 

``` bash
$ sudo systemctl restart apache2
```


**NB**: if you want test if url rewriting works correctly follow the "Step 3" paragraph in the [above](https://www.digitalocean.com/community/tutorials/how-to-rewrite-urls-with-mod_rewrite-for-apache-on-ubuntu-16-04) guide.

- In your Ampache folder install .htaccess files
``` bash
$ cd /var/www/html/ampache
$ cp ./rest/.htaccess.dist ./rest/.htaccess
$ cp ./play/.htaccess.dist ./play/.htaccess
$ cp ./channel/.htaccess.dist ./channel/.htaccess
```

- For each one of the three file above, edit them to match your Ampache public address (eg., `RewriteRule ^(/[^/]+|[^/]+/|/?)$ /play/index.php` become `RewriteRule ^(/[^/]+|[^/]+/|/?)$ /ampache/play/index.php` if your Ampache public url is `http://localhost/ampache/`). Do this for each `RewriteRule` statement. <br>
For example, `./rest/.htacces` should become

``` 
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-s
    RewriteRule ^(.+)\.view$ /ampache/rest/index.php?ssaction=$1 [PT,L,QSA]
    RewriteRule ^fake/(.+)$ /ampache/play/$1 [PT,L,QSA]
</IfModule>
```

- Make `ampache.cfg.php` writeable
``` bash
$ cd /var/www/html/ampache
$ sudo chown -R www-data:www-data ./config
```

- Increase PHP max upload size. Run `sudo nano /etc/php/7.3/apache2/php.ini` and change the following values
```
; Maximum allowed size for uploaded files.
upload_max_filesize = 40M

; Must be greater than or equal to upload_max_filesize
post_max_size = 40M
```
Change the following values


- Navigate to your ampache address (eg., `http://localhost/ampache`) to start the Ampache installation.
   - **NB**: During the installation on the "Step 1" check the **Create Database User** option.

## Deemix
NOTE: This guide assume you have a ~/Apps folder. If not, change the script as well.

```bash
$ cd ~/Apps
$ git clone https://git.rip/RemixDev/deemix-pyweb.git deemix
$ cd deemix
$ git submodule update --init --recursive
$ python3 -m venv ./venv
$ source ./venv/bin/activate
$ python3 -m pip install -U -r server-requirements.txt
# Launch and close (Ctrl-C) deemix (to create app folders)
$ python3 server.py --host 0.0.0.0
$ deactivate
```

- Make a new file `update.sh` (`nano update.sh`) and paste following code

```sh
echo "DEEMIX UPDATE"
echo
echo " Updating git repositories..."
git pull
git submodule update --init --recursive

echo
echo " Updating python packages..."
source ./venv/bin/activate
python3 -m pip install -U --upgrade-strategy eager -r requirements.txt
deactivate

echo
echo "UPDATE COMPLETED!"
```

- Make a new file `launch.sh` (`nano launch.sh`) and paste following code

```sh
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )" # get source file diretctory location
cd $DIR

echo "Launching deemix..."
source ./venv/bin/activate
python3 server.py --host 0.0.0.0 --serverwide-arl

# Script will be stuck here until killed

echo "Exiting ..."
deactivate
```

- Make files executable

```bash
$ chmod u+x update.sh
$ chmod u+x launch.sh
```

- Login into Deezer and exctract `arl` string

- Paste the string into `~/.config/deemix/.arl`

### Notes
**NB**: don't forget to add into `.bash_aliases`
```sh
alias deemix="/home/pi/Apps/deemix/launch.sh > /home/pi/.logs/deemix.log 2>&1 &"
```

Manually launch
```bash
$ ./launch.sh
```

Update programs and dependencies
```bash
$ ./update.sh
```

## Install Python 3
Python 3.10 is not on Debian repository so we have to build it from scratch.

- Start by installing the packages necessary to build Python source:

```bash
$ sudo apt update
$ sudo apt install build-essential tk-dev libncurses5-dev libncursesw5-dev libreadline6-dev libdb5.3-dev libgdbm-dev libsqlite3-dev libssl-dev libbz2-dev libexpat1-dev liblzma-dev zlib1g-dev libffi-dev
```

- Download the latest release’s source code from the Python download page with `wget` or `curl`. For example, for the version `3.10.0`:

```bash
wget https://www.python.org/ftp/python/3.10.0/Python-3.10.0.tar.xz
```

- When the download is complete, extract the tarball, navigate to the Python source directory and run the configure script:

```bash
$ tar -xf Python-3.10.0.tar.xz
$ cd Python-3.10.0
$ ./configure --enable-optimizations
```

The script performs a number of checks to make sure all of the dependencies on your system are present. The `--enable-optimizations` option will optimize the Python binary by running multiple tests, which will make the build process slower.

- Run `make` to start the build process:
```bash
$ make -j 4
```
The `-j` to correspond to the number of cores in your processor.

- Once the build is done, install the Python binaries by running the following command as a user with sudo access:
```bash
$ sudo make altinstall
```
**NB:** Do not use the standard make install as it will overwrite the default system python3 binary. <br> 

Now Python3.10 is installed. To use it instead of the system default 3.7 **you have to explicity run `python3.10`**, such as:
```bash
$ python3.10 --version
```

- Now you can clean up downloaded files
```bash
$ cd ..
$ sudo rm -rf Python-3.10.0.tar.xz
$ sudo rm -rf Python-3.10.0
```

## Install Pi-hole
- Install Pi-hole (from official [guide](https://docs.pi-hole.net/main/basic-install/))

    ```bash
	$ curl -sSL https://install.pi-hole.net | bash
	```

   **NB**: Take note of the admin password (shown at installation end).
   
- Change the lighttpd port
    - Edit the `lighttpd.conf` file
	
	    ```bash
		$ sudo nano /etc/lighttpd/lighttpd.conf
		```
	
	- Restart the service
	
	    ```bash
		$ sudo service lighttpd restart
		```

- Install [whitelist](https://github.com/anudeepND/whitelist)

    ```bash
	$ git clone https://github.com/anudeepND/whitelist.git
    $ sudo python3 whitelist/scripts/whitelist.py
	```
Notes: 

- If you want use Pi-hole over a VPN, read this [guide](https://gabriele.tips/virtual-private-pi-holed-network/) (**untested**). 

- Current block lists are taken from [here](https://www.andreadraghetti.it/block-list-e-white-list-per-pi-hole-e-ad-blocker/).



# Useful commands
## List active processes
Simple
```sh
$ ps aux
```

With pagination
```sh
$ ps aux | less
```

Search for a specific process
```sh
$ ps aux | grep process
```

# .bash_aliases
``` sh
alias hello='echo ciao'
alias ll='ls -l'
alias jd='java -Djava.awt.headless=true -jar /home/pi/JDownloader/JDownloader.jar >/dev/null 2>/dev/null &'
alias bridtools='cd /home/pi/Apps/BridTools/ && source ./venv/bin/activate'
alias bridhack='python brid_hack.py'
alias dlcextractor='python dlc_extractor.py'
alias linkscraping='python link_scraping.py'
alias deemix="/home/pi/Apps/deemix/launch.sh > /home/pi/.logs/deemix.log 2>&1 &"
alias deluged-all="deluged && deluge-web -f"
alias ha-start="sudo systemctl start home-assistant@homeassistant"
alias ha-stop="sudo systemctl stop home-assistant@homeassistant"
alias ha-restart="sudo systemctl restart home-assistant@homeassistant"
alias ha-restartlog="sudo systemctl restart home-assistant@homeassistant && sudo journalctl -f -u home-assistant@homeassistant"
alias ha-login="cd /home/homeassistant && sudo -u homeassistant -H -s"
```

After you add a new alias don't forget to run `. /home/pi/.bashrc`