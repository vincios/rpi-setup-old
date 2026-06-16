# General Setup

## Ngnix and PHP
1. Install ngnix

    ```bash
    $ sudo apt update
    $ sudo apt install nginx
    $ sudo systemctl stop nginx.service
    ```

    💡 Since we should have Traefik listening on the port `80` we stop the nginx service to prevent conflicts

2. Install PHP

    ```bash
    $ sudo apt install php-fpm
    ```

    💡 To enable php, nginx only needs the `php-fpm` module, but you may need to install other php modules according to each project requirements

3. Give to the user the permission to edit the `/var/www/html` folder

    ```bash
    $ sudo usermod -a -G www-data $USER
    $ sudo chown www-data:www-data /var/www/html/
    $ sudo chmod -R g+w /var/www/html/
    ```

    >💡 We have added the user to the `www-data` group, and the group-write permission to the `html` folder (owned by `www-data`)

4. Change the `default` site configuration

    - Edit the file `/etc/nginx/sites-available/default`
    - Replaces the following lines

      ```nginx
      # From
      listen 80 default_server;
      listen [::]:80 default_server;

      # To
      listen 8080 default_server;
      listen [::]:8080 default_server;
      ```
      ```nginx
      # From
      index index.html index.htm index.nginx-debian.html;

      # To
      index index.php index.html index.htm index.nginx-debian.html;
      ```
      ```nginx
      # From

      # pass PHP scripts to FastCGI server
      #location ~ \.php$ {
      #       include snippets/fastcgi-php.conf;
      #
      #       # With php-fpm (or other unix sockets):
      #       fastcgi_pass unix:/run/php/php7.4-fpm.sock;
      #       # With php-cgi (or other tcp sockets):
      #       fastcgi_pass 127.0.0.1:9000;
      #}

      # To
      location ~ \.php$ {
              include snippets/fastcgi-php.conf;
              fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
      }
      ```

5. Create the file `/var/www/html/index.php`

    ```php
    <?php phpinfo(); ?>
    ```

6. Start the nginx service

    ```
    $ sudo systemctl start nginx.service
    ```

7. Test it

    ```
    http://<YOUR PI's IP ADDRESS>:8080
    ```

> 💡 Now you can also remove the `default` site from the enabled sites
>
> ```bash
> $ sudo rm -f /etc/nginx/sites-enabled/default
> ```
>
> and just use the `/etc/nginx/sites-available/default` as template for your sites.


### SMB Access
Add the following configuration to the `/etc/samba/smb.conf` file

```ini
[html]
   comment = Web Directory
   path = /var/www/html
   writable = yes
   valid users = @www-data
   force group = www-data
   force user = www-data
   create mask = 0664
   directory mask = 0775
```

⚠️ The `/var/www/html` directory must be owned by `www-data:www-data`.

### Create a site
1. Your site's web files must be placed in the `/var/www/html/your.site` folder, giving the ownership to the `www-data` user

    ```bash
    $ chown -R www-data:www-data /var/www/html/your.site
    $ chmod -R 775 /var/www/html/your.site
    ```
2. Create the configuration file in the `/etc/nginx/sites-available/` 
  
    💡 You can use the `/etc/nginx/sites-available/default` as template

    ```bash
    $ sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/your.site
    ```

3. Edit the configuration file

    ⚠️ Don't forget to change the `root` directive to point to your site folder
  
4. Enable the site linking the configuration file to the `/etc/nginx/sites-enabled/` folder

    ```bash
    $ ln -s /etc/nginx/sites-available/your.site /etc/nginx/sites-enabled/your.site 
    ```

5. Test the configuration and restart the service

    ```bash
    $ sudo nginx -t # checks the configuration files syntax
    $ sudo systemctl reload nginx
    ```
    
## Install Pi-hole
1. Make sure that your system is fully updated (with `apt`)
2. [Install Pi-hole](https://docs.pi-hole.net/main/basic-install/)

    ⚠️ If you want to use [nginx](#ngnix-and-php) as web server for the web inteface, during the installation wizard, install the web interface **without** the default `lighttpd` web server:

      - Do you want to install the web admin interface? `Yes`
      - Do you want to install the `lighttpd` web server? `No`

    ```bash
    $ curl -sSL https://install.pi-hole.net | bash
    ```

    ⚠️ Make note of the admin password

3. Install [whitelist](https://github.com/anudeepND/whitelist)

    ```bash
	$ git clone https://github.com/anudeepND/whitelist.git
    $ sudo python3 whitelist/scripts/whitelist.py
	```

4.  If you don't have installed the default `lighttpd` web server, you must follow [Nginx as web server](#nginx-as-web-server)

Notes: 
- Current block lists are taken from [here](https://www.andreadraghetti.it/block-list-e-white-list-per-pi-hole-e-ad-blocker/).


### Nginx as web server [🦆]
0. The Pi-hole web interface needs of the following php modules to work. Make sure you have installed them on your system

    ```bash
    $ sudo apt install php-cgi php-xml php-sqlite3 php-intl
    ```

1. Stop and disable the default `lighttpd` (if necessary)

    ```bash
    $ service lighttpd stop
    $ systemctl disable lighttpd
    ```

2. Create the configuration file `/etc/nginx/sites-available/pihole` with the following code

    ```nginx
    server {
        listen 8088;
        listen [::]:8088;

        root /var/www/html;
        server_name pihole.<YOUR_DUCKDNS_DOMAIN>.duckdns.org;
        autoindex on;
        port_in_redirect off;

        index index.php index.html index.htm;

        location / {
            expires max;
            try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
            fastcgi_pass unix:/run/php/php7.4-fpm.sock;
            fastcgi_param FQDN true;
        }

        location /*.js {
            index admin/index.js;
        }

        location /admin {
            root /var/www/html;
            index index.php index.html index.htm;
        }

        location ~ /\.ht {
            deny all;
        }
    }
    ```

    ⚠️ Edit the line `fastcgi_pass unix:/run/php/php7.4-fpm.sock;` with your php version

    ⚠️ Edit the line `server_name pihole.<YOUR_DUCKDNS_DOMAIN>.duckdns.org;` with your duckdns domain

3. Give the right permission to the Pi-hole web folder

    ```bash
    $ sudo chown -R www-data:www-data /var/www/html/admin/
    $ sudo chmod 755 /var/www/html/admin/
    ```

4. Add the group `pihole` to the `www-data` user

    We need to add the group `pihole` to the `www-data` since it will be needed by the interface to work with the database

    ```bash
    $ sudo usermod -aG pihole www-data
    ```

5. Restart the `nginx` service

### (lighttpd only) Change the lighttpd port
The Pi-hole admin dashboard is served by a `lighttpd` web server on the port `80`, that we have to change to `8088`

- Edit the `lighttpd.conf` file

 ```bash
$ sudo nano /etc/lighttpd/lighttpd.conf
```

- Restart the service

 ```bash
$ sudo service lighttpd restart
```

### Traefik configuration
- Follow [Annex: Add custom dynamic configuration](#annex-add-custom-dynamic-configuration), use the following configuration:

    <details>
    <summary>✨ Click to see the code</summary>

    ```yml
    http:
      routers:
        pi-hole:
          rule: Host(`pihole.{{ env "DUCKDNS_DOMAIN"}}.duckdns.org`)
          service: "pi-hole"

          # Middlewares to which the request will be forwarded when the route is activated
          # Optional
          middlewares:
            - redirect-to-admin
            - authentication

          # Enable the TLS encryption
          # Normally, you should not need to edit this section
          tls:
            certResolver: "duckdnsResolver"
            domains:
              - main: "{{ env "DUCKDNS_DOMAIN"}}.duckdns.org"
                sans:
                  - "*.{{ env "DUCKDNS_DOMAIN"}}.duckdns.org"

      middlewares:
        redirect-to-admin:
          redirectRegex:
            regex: "^https://([^/]+)/?$"
            replacement: "https://${1}/admin/"

      # Service's urls where the request will be forwarded
      services:
        pi-hole:
          loadBalancer:
            servers:
            - url: "http://127.0.0.1:8088/admin/"
    ```

    </details>


### Update
```sh
$ sudo pihole -up
```

#### Web Admin repo is missing from system 
If you use [nginx as web server](#nginx-as-web-server-🦆) you may encounter the error `Web Admin repo is missing from system` during the update. 

In this case, try this fix

```sh
$ sudo git config --global --add safe.directory /var/www/html/admin
$ sudo pihole -up
```

If doesn't work, try to rebuild the web inteface folder

```sh
$ sudo rm /var/www/html/admin 
$ sudo git clone https://github.com/pi-hole/AdminLTE.git /var/www/html/admin
$ sudo chown -R www-data:www-data /var/www/html/admin/
$ sudo chmod 755 /var/www/html/admin/
$ pihole -r
```

Then, select `Repair`.

## Jellyfin
Follow the [official guide](https://jellyfin.org/docs/general/installation/linux#debian)

### Traefik configuration
- Follow [Annex: Add custom dynamic configuration](#annex-add-custom-dynamic-configuration), use the following configuration:

    <details>
    <summary>✨ Click to see the code</summary>

    ```yml
    http:
      routers:
        jellyfin:
          rule: Host(`jellyfin.{{ env "DUCKDNS_DOMAIN"}}.duckdns.org`)
          service: jellyfin

          # Enable the TLS encryption
          # Normally, you should not need to edit this section
          tls:
            certResolver: "duckdnsResolver"
            domains:
              - main: "{{ env "DUCKDNS_DOMAIN"}}.duckdns.org"
                sans:
                  - "*.{{ env "DUCKDNS_DOMAIN"}}.duckdns.org"

      # Service's urls where the request will be forwarded
      services:
        jellyfin:
          loadBalancer:
            servers:
            - url: "http://127.0.0.1:8096"
    ```

    </details>



## Install Docker
Just follow the [official guide](https://docs.docker.com/engine/install/raspbian/).

💡 According the official page, the recommended method to install docker in production should be [using the repository](https://docs.docker.com/engine/install/raspbian/#install-using-the-repository). If it doesn't work (packages not found error), just use the [convenience script](https://docs.docker.com/engine/install/raspbian/#install-using-the-convenience-script).

💡 In case you got a `permission denied while trying to connect to the Docker daemon socket` error, make sure that your user is in the `docker` group

```sh
$ sudo usermod -aG docker ${USER}
```


## Build TOR
Adapted from [1](https://tor.stackexchange.com/questions/75/how-can-i-install-tor-from-the-source-code-in-the-git-repository) and [2](https://www.torbox.ch/?page_id=205), we will build the latest offical release. Instead, if you want to build from the repository (instable, but with the lastest features), see [3](https://tor.stackexchange.com/questions/75/how-can-i-install-tor-from-the-source-code-in-the-git-repository) and [4](https://tor.stackexchange.com/questions/22510/how-to-build-and-install-tor-from-the-source-code-from-git-repository).

1. Install the build prerequisites

    ```bash
    $ sudo apt-get install git build-essential automake libevent-dev libssl-dev zlib1g-dev
    ```

2. Download the latest release from the official [source](https://www.torproject.org/download/tor/)


    ```bash
    $ wget https://dist.torproject.org/tor-${torversion}.tar.gz
    $ tar -zxvf tor-${torversion}.tar.gz
    ```

2. Build

    ```bash
    $ ./configure
    $ make
    ```
3. Install

    ```bash
    $ sudo make install
    ```

💡 The `torrc` file should be located into the `/usr/local/etc/tor/` folder. You can check it running `tor`: the first few lines of the output should tell the exact location where the configuration file need to be placed. If there is no `torrc` file in the target folder, create it copying the `torrc.sample` file you can find in the same folder.

### obfs4proxy
0. Download Go complier

    To build obfs4proxy we need a Go compiler. We don't need to install it permanentely. Just download the latest `linux-arm64` version from the [official](https://go.dev/dl/) page, extract and add it to the path

    ```bash
    $ wget https://go.dev/dl/go${goversion}.linux-arm64.tar.gz
    $ tar -xzvf go${goversion}.linux-arm64.tar.gz
    $ GOPATH=$HOME/go
    $ export PATH=<GO-LOCATION-PATH>/go/bin:$PATH
    $ go version # test it
    ```

    💡 At the end we can then delete the extracted `go` and the `$HOME/go` folders

1. Clone the repository

    ```bash
    $ git clone https://salsa.debian.org/pkg-privacy-team/obfs4proxy.git
    $ cd obfs4proxy
    ```

2. Build obfs4proxy

    ```bash
    $ export GO111MODULE="on"
    $ go build -o obfs4proxy/obfs4proxy ./obfs4proxy
    ```

3. Install

    ```bash
    $ sudo cp ./obfs4proxy/obfs4proxy /usr/local/bin
    ```

### snowflake
0. Download Go complier

    To build snowflake we need a Go compiler. We don't need to install it permanentely. Just download the latest `linux-arm64` version from the [official](https://go.dev/dl/) page, extract and add it to the path

    ```bash
    $ wget https://go.dev/dl/go${goversion}.linux-arm64.tar.gz
    $ tar -xzvf go${goversion}.linux-arm64.tar.gz
    $ GOPATH=$HOME/go
    $ export PATH=<GO-LOCATION-PATH>/go/bin:$PATH
    $ go version # test it
    ```

    💡 At the end we can then delete the extracted `go` and the `$HOME/go` folders

1. Clone the repository

    ```bash
    $ git clone https://github.com/tgragnato/snowflake
    ```

2. Open the `snowflake/proxy/` folder, build and install

    ```bash
    $ go get
    $ go build
    $ sudo cp proxy /usr/local/bin/snowflake-proxy
    ```

3. Open to the `snowflake/client/` folder, build and install

    ```bash
    $ go get
    $ go build
    $ sudo cp client /usr/local/bin/snowflake-client
    ```

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


## JDownloader
Follow the [official](https://support.jdownloader.org/Knowledgebase/Article/View/52/0/install-jdownloader-on-nas-and-embedded-devices) guide.

### Run headless
The headless command listed in the guide

```bash
$ java -jar JDownloader.jar &
```

Shows a lot of outputs to the terminal. To prevent this, use instead

```bash
$ java -Djava.awt.headless=true -jar JDownloader.jar >/dev/null 2>/dev/null &
```

💡 Add it to the `.bash_aliases` file

### Run the GUI
To open the GUI, we simple make a executable shortuct on the desktop.

1. Download a JDownloader icon 

    ```bash
    $ wget -O [JD_Install_dir]/jDownloader.png http://jdownloader.org/_media/knowledge/wiki/jdownloader.png
    ```

2. Create a `jDownloader.desktop` file on the raspi desktop with the following content

    ```ini
    [Desktop Entry]
    Encoding=UTF-8
    Name=jDownloader2
    Comment=jDownloader2
    Exec=bash -c "java -jar [JD_install_dir]/JDownloader.jar"
    Icon=[JD_install_dir]/jDownloader.png
    Type=Application
    Categories=GTK;Utility;
    ```

    ⚠️ Don't forget to edit the `Exec` and the `Icon` parameter wiht yuour JD install path

3. Make it executable

    ```bash
    $ chmod +x jDownloader.desktop
    ```


### RAR5 support
We have to add the SevenZipBinding library to the JDownloader installation.

You can either:

- Download a precompiled jar files from [here](https://board.jdownloader.org/showpost.php?p=455292&postcount=583) and copy it in `[JD_Install_dir]/libs`
 
- Compile it manually (preferred)

To compile from source (adapted from [here](https://www.ixsystems.com/community/threads/guide-jdownloader2-in-11-1-release-iocage-with-rar5-working.74073/), or [here](https://board.jdownloader.org/showpost.php?p=446708&postcount=465)):

1. Install `cmake` and `openjdk8-jdk`

    ``` bash
    $ sudo apt-get install cmake
    $ sudo apt-get install openjdk8-jdk
    ```

2. Go to JD folder and make folders:

    ``` bash
    $ mkdir rar5
    $ cd rar5
    ```

3. Clone repository

    ``` bash
    $ git clone https://github.com/borisbrodski/sevenzipjbinding.git sevenzipbinding
    ```

    ⚠️ It should be better to choose a branch between `bind_16.02` and `migrate-to-15.09-try2` but it should work also with `master` branch. So first try with `master` then, in case of failure, try `bind_16.02` and, as last chance, `migrate-to-15.09-try2`.

    💡 You can change branch with the `git checkout [branch_name]` command.

4. Run CMake

    ``` bash
    $ cmake . -DJAVA_JDK=/usr/lib/jvm/java-8-openjdk-armhf
    $ make
    $ make package
    ```

5. Move jar library to JD lib folder

    ``` bash
    $ unzip sevenzipjbinding-16.02-2.01beta-Linux-arm.zip
    $ cd sevenzipjbinding-16.02-2.01beta-Linux-arm/lib/
    $ mv sevenzipjbinding.jar [JD_install_dir]/libs/sevenzipjbinding1509.jar
    $ mv sevenzipjbinding-Linux-arm.jar [JD_Install_dir]/libs/sevenzipjbinding1509LinuxArmVersion.jar
    ```

    ⚠️ If you change branch, rename the commands above with correct version numbers. But **DON'T** change destination filenames (`sevenzipjbinding1509.jar` and `sevenzipjbinding1509LinuxArmVersion.jar`)


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
See [here](https://pimylifeup.com/raspberry-pi-deluge).<br>
Do not follow the last part ("Setting up Deluge as a service").<br>
Don't forget to add the alias in the `.bash_aliases` file.

## VaultWarden
[VaultWarden]() is an unofficial Bitwarden compatible server written in Rust.

### Install VaultWarden [🦆]
> [!WARNING]
> Immich requires [docker](#install-docker).

1. Create a `vaultwarden` folder somewhere

2. Create a `docker-compose.yml` file inside it, paste the following code

    ```yml

    ```

3. Download the default `.env` file

    ```bash
    $ wget -O .env https://raw.githubusercontent.com/dani-garcia/vaultwarden/main/.env.template
    ```

4. Edit the `.env` file
    1. Uncomment a `DOMAIN` variable, edit it according to your domain

5. [Configure fail2ban](https://github.com/dani-garcia/vaultwarden/wiki/Fail2Ban-Setup)

  > [!TIP]
  > Unless changed with the `LOGS_LOCATION` env variable, the default logs location is `./vw-logs`


### First run
If you want to [enable the Admin page](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page) and generate the `ADMIN_TOKEN` via a temporary container, follow the steps 1-3.

Otherwise, skip to the step 4.

In the `docker-compose.yml` folder
1. Pull the container

    ```bash
    $ docker compose pull
    ```

2. Generate the `ADMIN_TOKEN`

    ```bash
    $ docker run --rm -it vaultwarden/server /vaultwarden hash --preset owasp
    ```

3. Paste the result into the `.env` file

4. Start the container

    ```bash
    $ docker compose up -d
    ```
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
alias jd='java -Djava.awt.headless=true -jar /home/raspi/JDownloader/JDownloader.jar >/dev/null 2>/dev/null &'
alias bridtools='cd /home/raspi/Apps/BridTools/ && source ./venv/bin/activate'
alias bridhack='python brid_hack.py'
alias dlcextractor='python dlc_extractor.py'
alias linkscraping='python link_scraping.py'
alias deemix="/home/raspi/Apps/deemix/launch.sh > /home/raspi/.logs/deemix.log 2>&1 &"
alias deluged-all="deluged && deluge-web -f"
alias ha-start="sudo systemctl start home-assistant@homeassistant"
alias ha-stop="sudo systemctl stop home-assistant@homeassistant"
alias ha-restart="sudo systemctl restart home-assistant@homeassistant"
alias ha-restartlog="sudo systemctl restart home-assistant@homeassistant && sudo journalctl -f -u home-assistant@homeassistant"
alias ha-login="cd /home/homeassistant && sudo -u homeassistant -H -s"
```

After you add a new alias don't forget to run `. /home/raspi/.bashrc`*
