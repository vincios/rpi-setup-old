# Common configurations for Raspberry Pi with Raspbian

> Indice
> + [VNC "cannot currently show the desktop" in headless mode](#vnc-cannot-currently-show-the-desktop-in-headless-mode)
> + [AutoMount Nas folders](#automount-nas-folders)
> + [Samba shares](#samba-shares)
> + [Duckdns cron configuration](#duckdns-cron-configuration)
> + [Plex Media Server](#plex-media-server)
> + [Build TOR](#build-tor)
> + [Run BridTools](#run-bridtools)
> + [Install jDownloader in headless mode](#install-jdownloader-in-headless-mode)
> + [JDownloader RAR5 support](#jdownloader-rar5-support)
> + [Update Node and npm](#update-node-and-npm)
> + [Install Deluge torrent client with web interface](#install-deluge-torrent-client-with-web-interface)
> + [Install Transmission](#install-transmission)
> + [Install LAMP software](#install-lamp-software)
>     + [Add xdebug](#add-xdebug)
>     + [Start, Stop and Restart apache2 service](#start-stop-and-restart-apache2-service)
> + [Install Ampache](#install-ampache)
> + [Deemix](#deemix)
>     + [Notes](#notes)
> + [Install Python 3.8](#install-python-38)
> + [Home Assistant](#home-assistant)
>     + [Service creation](#service-creation)
>     + [Switch to homeassistant user](#switch-to-homeassistant-user)
> 	  + [Updating](#updating)
>     + [Activate Advanced Mode](#activate-advanced-mode)
>     + [Create ssl certificates](#create-ssl-certificates)
>     + [Install HACS](#install-hacs)
> 	  + [Other useful commands](#other-useful-commands)
> + [Useful commands](#useful-commands)
>     + [List active processes](#list-active-processes)
> + [.bash_aliases](#bash_aliases)


## VNC "cannot currently show the desktop" in headless mode
Run `raspi-config` and change screen resolution to 1920x1080

## AutoMount Nas folders
- Follow points 1, 2, 3 [here](http://timlehr.com/auto-mount-samba-cifs-shares-via-fstab-on-linux/)
- Run `sudo nano /etc/fstab` and add these lines (changes paths as done in point 2)

``` 
//192.168.1.200/home          /media/nas/home/      cifs    credentials=/home/pi/.nascredentials,uid=pi,gid=pi,iocharset=utf8,file_mode=0777,dir_mode=0777,noperm,vers=1.0  0       0
//192.168.1.200/home_2        /media/nas/home_2/    cifs    credentials=/home/pi/.nascredentials,uid=pi,gid=pi,iocharset=utf8,file_mode=0777,dir_mode=0777,noperm,vers=1.0  0       0
//192.168.1.200/share         /media/nas/share/     cifs    credentials=/home/pi/.nascredentials,uid=pi,gid=pi,iocharset=utf8,file_mode=0777,dir_mode=0777,noperm,vers=1.0  0       0
//192.168.1.210/Multimedia    /media/qnas/Media/    cifs    credentials=/home/pi/.nascredentials,uid=pi,gid=pi,iocharset=utf8,file_mode=0777,dir_mode=0777,noperm           0       0
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
   create mask = 0740
   dierectory mask = 0750
   public = yes
```

- Choose a smb password for pi user

`sudo smbpasswd -a pi`


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
```bash
$ cd ~
$ git clone https://codeberg.org/RemixDev/deemix-pyweb.git .deemix
$ cd .deemix
$ git submodule update --init --recursive
$ python3 -m venv ./venv
$ source ./venv/bin/activate
$ python3 -m pip install -U -r requirements.txt
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
alias deemix="/home/pi/.deemix/launch.sh > /dev/null 2>&1 &"
```

Manually launch
```bash
$ ./launch.sh
```

Update programs and dependencies
```bash
$ ./update.sh
```

## Install Python 3.8
Python 3.8 is not on Debian repository so we have to build it from scratch.

- Start by installing the packages necessary to build Python source:

```bash
$ sudo apt update
$ sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libsqlite3-dev libreadline-dev libffi-dev wget libbz2-dev
```

- Download the latest release’s source code from the Python download page with `wget` or `curl` . The last one should be `3.8.6`:

```bash
wget https://www.python.org/ftp/python/3.8.6/Python-3.8.6.tar.xz
```

- When the download is complete, extract the tarball, navigate to the Python source directory and run the configure script:

```bash
$ tar -xf Python-3.8.6.tar.xz
$ cd Python-3.8.6
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

Now Python3.8 is installed. To use it instead of the system default 3.7 **you have to explicity run `python3.8`**, such as:
```bash
$ python3.8 --version
```

- Now you can clean up downloaded files
```bash
$ cd ..
$ sudo rm -rf Python-3.8.6.tar.xz
$ sudo rm -rf Python-3.8.6
```

## Home Assistant
### Install
- [Install Python 3.8](#install-python-38)
- Follow the official [guide](https://www.home-assistant.io/docs/installation/raspberry-pi/)

    For first launch use `hass -v` and wait until log scroll stops.
	
- Create the [service](#service-creation)

### Service creation
**NB:** Run this steps as `pi` user! 
- Create systemd service
```bash
$ sudo nano -w /etc/systemd/system/home-assistant@homeassistant.service
```
- Paste following text
```sh
[Unit]
Description=Home Assistant
After=network-online.target

[Service]
Type=simple
User=%i
ExecStart=/srv/homeassistant/bin/hass -c "/home/homeassistant/.homeassistant"

[Install]
WantedBy=multi-user.target
```

- Restart Systemd and load new service
```bash
$ sudo systemctl --system daemon-reload
$ sudo systemctl enable home-assistant@homeassistant
$ sudo systemctl start home-assistant@homeassistant
```

- Add these lines to `bash_aliases`
```sh
alias ha-start="sudo systemctl start home-assistant@homeassistant"
alias ha-stop="sudo systemctl stop home-assistant@homeassistant"
alias ha-restart="sudo systemctl restart home-assistant@homeassistant"
alias ha-restartlog="sudo systemctl restart home-assistant@homeassistant && sudo journalctl -f -u home-assistant@homeassistant"
```

### Switch to homeassistant user
To do some configuration operations (edit configuration files, updating application, etc.) you have to switch to the `homeassistant` user using the following command
```bash
$ sudo -u homeassistant -H -s
```

You can also automate this operation adding the command above in the `bash_aliases` file. Something such as:
```sh
alias ha-login="sudo -u homeassistant -H -s"
```

### Updating
To update to the latest version of Home Assistant Core follow these simple steps:

```bash
$ sudo -u homeassistant -H -s
$ source /srv/homeassistant/bin/activate
$ pip3 install --upgrade homeassistant
```

### Activate Advanced Mode
You can activate Advanced Mode under user profile page (click on the user's name at the bottom of the left sidebar).

### Edit configuration.yaml file
To edit the ***configuration.yaml*** file you have to [switch to homeassistant user](#switch-to-homeassistant-user)

### Create ssl certificate
From [this](https://indomus.it/guide/collegarsi-da-remoto-a-home-assistant-installato-su-raspberry-raspbian/) guide. <br>

**NOTE**: This guide assume a configured duckdns domain `cclouds.duckdns.org`. If you use a different domain, change scripts when indicated.

- Enable port forwarding to Home Assistant on the router (port 8123)

- Check that in `configuration.yaml` there isn't the field `base_url` under `http` block. Delete it if present.

- Open Home Assistant, go to `Settings` > `General` > `External URL` and insert your Home Assistant external URL.
    
	**NB**: You must [activate Advanced Mode](#activate-advanced-mode) to see `Eternal URL` field.
	
    ```
    http://cclouds.duckdns.org:8123
    ```
	
- [Switch to homeassistant user](#switch-to-homeassistant-user)

- Go to the user home folder (`$ cd /home/homeassistant`)

- Clone the Dehydrated repo

    ```bash
	$ git clone https://github.com/dehydrated-io/dehydrated.git
	```
	
- Enter into the Dehydrated folder (`cd dehydrated`), create a domains.txt file (`nano domains.txt`) and paste your domain
	
	```
	cclouds.duckdns.org
	```

- Create in the same directory a file config (`nano config`) and paste these lines
    
	```sh
	# Which challenge should be used? Currently http-01 and dns-01 are supported
    CHALLENGETYPE="dns-01"
    
    # Script to execute the DNS challenge and run after cert generation
    HOOK="${BASEDIR}/hook.sh"
	```

- Create in the same directory a file hook.sh (`nano hook.sh`) and paste these lines
    
	**NB:** change `domain` and `token` with your duckdns domain and token.
	
    ```sh
    #!/usr/bin/env bash
    set -e
    set -u
    set -o pipefail
    
    domain="cclouds"
    token="your-duckdns-token"
    
    case "$1" in
        "deploy_challenge")
            curl "https://www.duckdns.org/update?domains=$domain&token=$token&txt=$4"
            echo
            ;;
        "clean_challenge")
            curl "https://www.duckdns.org/update?domains=$domain&token=$token&txt=removed&clear=true"
            echo
            ;;
        "deploy_cert")
            sudo systemctl restart home-assistant@homeassistant.service
            ;;
        "unchanged_cert")
            ;;
        "startup_hook")
            ;;
        "exit_hook")
            ;;
        *)
            echo Unknown hook "${1}"
            exit 0
            ;;
    esac
	```

- Make the file executable
    
	```bash
	$ chmod 0777 hook.sh
	```

- Run dehydrated a first time to register the certificate
    
	```bash
	$ ./dehydrated --register --accept-terms
	````
	
	You should obtain an output like this
    ```bash
    # INFO: Using main config file /home/homeassistant/dehydrated/config
    + Generating account key...
    + Registering account key with ACME server...
    + Fetching account ID...
    + Done!
	````
- Run dehytrdrated a second time to sign the certificate
    
	```bash
	$ ./dehydrated -c
	````
	
	You should obtain an output like this
    ```bash
    # INFO: Using main config file /home/homeassistant/dehydrated/config
    Processing myhome.duckdns.org
    + Signing domains...
    + Generating private key...
    + Generating signing request...
    + Requesting challenge for myhome.duckdns.org... OK
    + Responding to challenge for myhome.duckdns.org... OK
    + Challenge is valid!
    + Requesting certificate...
    + Checking certificate...
    + Done!
    + Creating fullchain.pem...
    + Walking chain...
    + Done!
	````
	
	**NB:** if the prompt requests the user password, simply stop the execution (`Ctrl+C`). The execution is still valid.
	
Now we have a valid certificate signed with a private keys that expiry after 90 days. <br>

You can find the signed certificate and the private key in the folder `~/dehydrated/certs/cclouds.duckdns.org`. <br>
There should be these files (and others):

```
 -- ~/dehydrated/certs/cclouds.duckdns.org  
  |-- cert.csr  
  |-- cert.pem  
  |-- chain.pem  
  |-- fullchain.pem    <---- This is your signed certificate  
  |-- privkey.pem    <---- This is your private key  
```

Now we configure the system to check every day at 01:00 the certificate validity. **The certificate will be automatically renewd if the expiry date is less than 30 days**

- Run the `cron` editor
  
    ```bash
	$ export VISUAL=nano; crontab -e
	```
	
	You may see a warning that the cron file doesn't exists. If it ask you which editor use, choose `nano`.
	
- Paste this line the end of file

    ```
	0 1 * * * /home/homeassistant/dehydrated/dehydrated -c | tee /home/homeassistant/dehydrated/update.log
	```

Now we have to add the certificate and the key to Home Assistant configuration file.  

- Open the `configuration.yaml` file and add (or edit) these lines under the `http` block (if the block doesn't exists manually add it)

    ```
    http:
      ssl_certificate: /home/homeassistant/dehydrated/certs/cclouds.duckdns.org/fullchain.pem
      ssl_key: /home/homeassistant/dehydrated/certs/cclouds.duckdns.org/privkey.pem
    ```

- Open Home Assistant, go to `Settings` > `General` > `External URL` and edit the field

    ```
    https://cclouds.duckdns.org:8123
    ```
	
- Restart Home assistant

Home Assistant is now configured to use ssl connection. You can connect to it with the external URL `https://cclouds.duckdns.org:8123` or the internal URL `https://RASPI_IP:8123`.
In the latter case, you should see a security error, this is normal because the certificate is signed for the external URL.

### Install HACS
From [here](https://hacs.xyz/docs/installation/prerequisites) (With some modifications).

- Create `custom_components` folder into the `.homeassistant` folder.

    ```bash
	$ sudo install -g homeassistant -o homeassistant -d /home/homeassistant/.homeassistant/custom_components
	```
	`install` will create the custom_components folder and give to it homeassistant ownership.
	
- Download `hacs.zip` into tmp folder. Take the latest release link from [here](https://github.com/hacs/integration/releases/latest).

    ```bash
	$ wget https://github.com/hacs/integration/releases/download/1.6.2/hacs.zip -P /tmp/
	```
	
- Unzip it into a `hacs` folder (the folder name MUST BE exatly this), copy it in the homassistant folder, grant homeassistant ownership and delete temporary files.

    ```bash
	$ mkdir /tmp/hacs
	$ unzip -d /tmp/hacs /tmp/hacs.zip
	```
	
- Copy the folder into homeassistant custom_components folder and grant homeassistant ownership to it.

    ```bash
	$ sudo mv /tmp/hacs /home/homeassistant/.homeassistant/custom_components/
	$ sudo chown -R homeassistant:homeassistant /home/homeassistant/.homeassistant/custom_components/hacs/
	```

- Delete temporary files 

    ```bash
	$ rm /tmp/hacs.zip
	```

- Reboot the system

- Follow [this](https://hacs.xyz/docs/configuration/start) configuration steps.


### Other useful commands
- Verify Home Assistant service status
``` bash
$ sudo systemctl status home-assistant@homeassistant
```

- Start Home Assistant service
``` bash
$ sudo systemctl start home-assistant@homeassistant
````

- Stop Home Assistant service
``` bash
$ sudo systemctl stop home-assistant@homeassistant
```

- Restart Home Assistant service
``` bash
$ sudo systemctl restart home-assistant@homeassistant
```

- Disable Home Assistant service autostart
``` bash
$ sudo systemctl disable home-assistant@homeassistant
```

- Read real time Home Assistant `systemlog` rows
``` bash
$sudo tail -f /var/log/syslog | grep hass
```

- Read Home Assistant log output
``` bash
$ sudo journalctl -f -u home-assistant@homeassistant
```

Because the log can scroll quite quickly, you can select to view only the error lines
``` bash
$ sudo journalctl -f -u home-assistant@homeassistant | grep -i ‘error’
```

- Verify configuration by manual launch
``` bash
$ hass --script check_config –h
```

## Useful commands
### List active processes
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
alias bridtools='cd /home/pi/.BridTools/ && source ./venv/bin/activate'
alias bridhack='python brid_hack.py'
alias dlcextractor='python dlc_extractor.py'
alias linkscraping='python link_scraping.py'
alias deemix="/home/pi/.deemix/launch.sh > /dev/null 2>&1 &"
alias deluged-all="deluged && deluge-web -f"
alias ha-start="sudo systemctl start home-assistant@homeassistant"
alias ha-stop="sudo systemctl stop home-assistant@homeassistant"
alias ha-restart="sudo systemctl restart home-assistant@homeassistant"
alias ha-restartlog="sudo systemctl restart home-assistant@homeassistant && sudo journalctl -f -u home-assistant@homeassistant"
alias ha-login="sudo -u homeassistant -H -s"
```

After you add a new alias don't forget to run `. /home/pi/.bashrc`