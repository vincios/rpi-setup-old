## Plex Media Server
See [here](https://pimylifeup.com/raspberry-pi-plex-server/).

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
    "watch-dir": "/media/nas/home/raspi/torrents",
    "watch-dir-enabled": true
```

- Add user `raspi` to the group `debian-transmission`
```bash
$ sudo usermod -a -G debian-transmission raspi
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
alias deemix="/home/raspi/Apps/deemix/launch.sh > /home/raspi/.logs/deemix.log 2>&1 &"
```

Manually launch
```bash
$ ./launch.sh
```

Update programs and dependencies
```bash
$ ./update.sh
```
