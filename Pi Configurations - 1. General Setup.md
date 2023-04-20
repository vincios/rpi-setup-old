# General Setup
⚠️ Please read before [First operations](#first-operations).

💡 Services with the duck [🦆] symbol have your duckdns domain hardcoded in some configuration files. You may need to reconfigure them if you change your duckdns domain.

Index
- [General Setup](#general-setup)
  - [First operations](#first-operations)
  - [VNC "cannot currently show the desktop" in headless mode](#vnc-cannot-currently-show-the-desktop-in-headless-mode)
  - [AutoMount Nas folders](#automount-nas-folders)
  - [Samba shares](#samba-shares)
    - [Note for Windows users](#note-for-windows-users)
  - [Duckdns cron configuration \[🦆\]](#duckdns-cron-configuration-)
  - [Traefik \[🦆\]](#traefik-)
    - [Install Traefik](#install-traefik)
    - [Configure Traefik (static configuration)](#configure-traefik-static-configuration)
    - [Configure Services (dynamic configuration)](#configure-services-dynamic-configuration)
    - [Setup systemd service](#setup-systemd-service)
    - [Configure log rotation](#configure-log-rotation)
    - [Update Traefik](#update-traefik)
    - [Annex: Add custom dynamic configuration](#annex-add-custom-dynamic-configuration)
  - [Wireguard VPN \[🦆\]](#wireguard-vpn-)
    - [Install Wireguard](#install-wireguard)
    - [Add clients](#add-clients)
      - [Setup the configuration environment](#setup-the-configuration-environment)
      - [Create the client's `.conf` file](#create-the-clients-conf-file)
      - [Add the new client to the server's `.conf` file](#add-the-new-client-to-the-servers-conf-file)
      - [Connect your client and test the connection](#connect-your-client-and-test-the-connection)
    - [(Optional) Route all internet traffic through the VPN Tunnel](#optional-route-all-internet-traffic-through-the-vpn-tunnel)
      - [Split Tunnel vs Full Tunnel](#split-tunnel-vs-full-tunnel)
      - [Enable IP forwarding on the server](#enable-ip-forwarding-on-the-server)
      - [Enable NAT on the server](#enable-nat-on-the-server)
      - [Accessing your home LAN](#accessing-your-home-lan)
      - [Route the entire Internet traffic through the WireGuard tunnel](#route-the-entire-internet-traffic-through-the-wireguard-tunnel)
  - [Install Pi-hole](#install-pi-hole)
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
- [Useful commands](#useful-commands)
  - [List active processes](#list-active-processes)
- [.bash\_aliases](#bash_aliases)

## First operations
Tutorials in this document assumes that you have first followed this paragraph. <br>
**Please make sure to follow this steps before all other tutorials!**

0. The OS default user name is `raspi`
   
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
    //192.168.1.200/Volume_1    /media/dnas/            cifs    credentials=/home/raspi/.credentials/.dnascredentials,uid=raspi,gid=raspi,iocharset=utf8,file_mode=0755,dir_mode=0755,noperm,vers=1.0   0   0
    //192.168.1.210/Multimedia  /media/qnas/Media/      cifs    credentials=/home/raspi/.credentials/.qnascredentials,uid=raspi,gid=raspi,iocharset=utf8,file_mode=0755,dir_mode=0755,noperm            0   0
    //192.168.1.210/Download    /media/qnas/Download/   cifs    credentials=/home/raspi/.credentials/.qnascredentials,uid=raspi,gid=raspi,iocharset=utf8,file_mode=0755,dir_mode=0755,noperm            0   0
    ```

- Run `raspi-config` and enable "Wait for Network at Boot" under "Boot options"
- Check if OK with `sudo mount -a`


## Samba shares
- `sudo apt-get install samba`

- `sudo nano /etc/samba/smb.conf`

- Default Samba share of the user folder have some security restrictions. So, choose one of the following options
    
    - **OPTION 1**: edit the default configuration
      
      -  In the `[homes]` section of *smb.conf* find the line `read only = yes` and change to `read only = no`
  
    - **OPTION 2**: crate a new one configuration

        - Add to the bottom

            ```
            [PiShare]
            comment = Pi Share
            path = /home/raspi
            browseable = yes
            writeable = yes
            only guest = no
            read only = no
            create mask = 0740
            dierectory mask = 0750
            public = yes
            valid users = %S
            ```

- Choose a smb password for raspi user

    ```bash
    $ sudo smbpasswd -a raspi
    ```

- Restart smb

    ```bash
    $ sudo systemctl restart smbd.service
    ```

### Note for Windows users
If the network folder is not visible or is not writeable, try this solutions (one at time):

- If windows explorer doesn't ask for password when you open the network share: open the Windows Credential Manager and, under the `Windows Credentias` tab, manually add the credentias for the network address `\\RASPBERRYPI`. Then, restart Explorer or the computer.

- On the raspberry, try this command `sudo pdbedit -a -u raspi`

## Duckdns cron configuration [🦆]
Adapted from [here](https://www.duckdns.org/install.jsp?tab=pi).

1. Create a `.duckdns` folder in your home directory

2. Create a `duck.sh` file with following content

    ```bash
    timestamp() {
      date +"%Y-%m-%d %H:%M:%S"
    }

    echo url="https://www.duckdns.org/update?domains=$DUCKDNS_DOMAINS&token=$DUCKDNS_TOKEN&ip=" | curl -k -o ~/.duckdns/log.log -K -
    echo " | Last run: $(timestamp)" >> ~/.duckdns/log.log
    ```
  
3. Create a `duck.conf.sh` file with following content

    ```bash
    export DUCKDNS_DOMAINS=<YOUR_DUCKDNS_DOMAINS>
    export DUCKDNS_TOKEN=<YOUR_DUCKDNS_TOKEN>
    ```

    ⚠️ Don't forget to edit the `<YOUR_DUCKDNS_DOMAINS>` and `<YOUR_DUCKDNS_TOKEN>` fields. No quotes needed. 
    
    💡 `<YOUR_DUCKDNS_DOMAINS>` can be a comma separated (**NO spaces**) list of domains

4. Make the script executable

    ```bash
    $ chmod 700 duck.sh duck.conf.sh
    ```

5. Test the script

    ```bash
    $ . $HOME/.duckdns/duck.conf.sh; $HOME/.duckdns/duck.sh
    ```

    ⚠️ Note the leading dot `.`

    💡 Check the result in the `log.log` file

6. Edit the cron configuration

    ```bash
    $ crontab -e
    ```

7. Add this line to the bottom

    ```bash
    */5 * * * * . $HOME/.duckdns/duck.conf.sh; $HOME/.duckdns/duck.sh >/dev/null 2>&1
    ```

8. Start the cron service

    ```bash
    $ sudo service cron start
    ```

## Traefik [🦆]
Traefik is designed to run in docker and auto discover the services by its [providers](https://doc.traefik.io/traefik/providers/overview/) (like the [docker](https://doc.traefik.io/traefik/providers/docker/) one). But in this setup, we will install it locally and use the [file provider](https://doc.traefik.io/traefik/providers/file/) to define the dynamic configuration (i.e. Routers, Services and Middlewares).

Starting from [this](https://adapttive.com/blog/deploying-node-js-app-with-pm-2-and-traefik/) guide, we will skip the node part and change the acme configuration to work with Duckdns.

### Install Traefik
1. Go to the Traefik [releases](https://github.com/containous/traefik/releases) page, find the latest `linux_arm64` release. Download and extract

    ```bash
    $ wget https://github.com/traefik/traefik/releases/download/${traefik_version}/traefik_${traefik_version}_linux_arm64.tar.gz
    $ tar -zxvf traefik_${traefik_version}_linux_arm64.tar.gz
    ```

2. Use help to check it works

    ```bash
    $ ./traefik --help
    ```

3. Make executable and move to `/usr/local/bin`

    ```bash
    $ sudo chmod +x traefik
    $ sudo cp traefik /usr/local/bin/
    $ sudo chown root:root /usr/local/bin/traefik
    $ sudo chmod 755 /usr/local/bin/traefik
    ```

4. Give the traefik binary the ability to bind to privileged ports (80, 443) as non-root
   
    ```bash
    $ sudo setcap 'cap_net_bind_service=+ep' /usr/local/bin/traefik
    ```

5. Setup traefik user and group and permissions

    ```bash
    $ sudo groupadd -g 321 traefik
    $ sudo useradd -g traefik --no-user-group --home-dir /var/www --no-create-home --shell /usr/sbin/nologin --system --uid 321 traefik
    $ sudo usermod -a -G traefik $USER
    ```

6. Create the configuration and logs folders, give them the right permissions

    ```bash
    # configuration folders
    $ sudo mkdir /etc/traefik
    $ sudo mkdir /etc/traefik/dynamic
    $ sudo mkdir /etc/traefik/acme
    $ sudo touch /etc/traefik/acme/acme.json
    $ sudo chown -R root:root /etc/traefik
    $ sudo chown -R traefik:traefik /etc/traefik/dynamic /etc/traefik/acme
    $ sudo chmod 600 /etc/traefik/acme/acme.json

    # logs files
    $ sudo mkdir /var/log/traefik
    $ sudo touch /var/log/traefik/traefik.log
    $ sudo touch /var/log/traefik/access.log
    $ sudo chown traefik:traefik  /var/log/traefik/ /var/log/traefik/access.log  /var/log/traefik/traefik.log
    ```

### Configure Traefik (static configuration)
We configure Traefik to [automatic renew](https://doc.traefik.io/traefik/https/acme/#automatic-renewals) Let's Encrypt ACME certificates, with a `duckdns` domain and a `DNS-01` challenge.

1. Create the file `/etc/traefik/traefik.yml` with the following content:
    
    <details>
    <summary>✨ Click to see the code</summary>

    ```yml
    entryPoints:
      web:
        address: ":80"
        http:
          redirections:
            entryPoint:
              to: "websecure"
              scheme: "https"
      websecure:
        address: ":443"
        http:
    certificatesResolvers:
      duckdnsResolver:
        # Enable ACME (Let's Encrypt): automatic SSL.
        acme:
          # Email address used for Let's Encrypt registration.
          email: "<YOUR_EMAIL>"

          # File or key used for certificates storage.
          # Recommended: give to the file permissions 600
          storage: "/etc/traefik/acme/acme.json"

          # The certificates' duration in hours.
          # It defaults to 2160 (90 days) to follow Let's Encrypt certificates' duration.
          # certificatesDuration: 2160

          # With duckdns, we use a DNS-01 ACME challenge
          # NB: don't forget to set your duckdns token in a DUCKDNS_TOKEN environment variable
          dnsChallenge:
            provider: "duckdns"
            # Wait x seconds before traefik checks the TXT record
            delayBeforeCheck: 20
    log:
      level: "INFO"
      filePath: "/var/log/traefik/traefik.log"
    accessLog:
      filePath: "/var/log/traefik/access.log"
      bufferingSize: 100
    api:
      # Set to true allows to access the api without HTTPS
      insecure: false
      # Enable the dashboard service
      dashboard: true
    providers:
      # The traefik dynamic configuration (Routers/Services/Middlewares) is configured with a file provider
      file:
        # all the yml files found into this folder will be used as dynamic configuration
        # so you can split your services configuration in multiple files, one for each service
        # for an example, see /etc/traefik/dynamic/dashboard.yml
        directory: "/etc/traefik/dynamic"
        watch: true
    ```

    ⚠️ Don't forget to edit the `<YOUR_EMAIL>` field
    
    💡 For the first times, you may want to set the log level to `DEBUG` to troubleshoot (eventual) errors
    
    </details>

2. Set file permissions

    ```bash
    $ sudo chown traefik:traefik /etc/traefik/traefik.yml
    $ sudo chmod 644 /etc/traefik/traefik.yml
    ```

### Configure Services (dynamic configuration)
We use the [file provider](https://doc.traefik.io/traefik/providers/file/) to manually set the dynamic configuration (Routers, Services and Middlewares) for each service we want to be served by the Traefik proxy.

The dynamic configuration will be stored in the `/etc/traefik/dynamic` folder, split in multiple files, one for each service we want to configure.

1. Create the file `/etc/traefik/dynamic/middlewares.yml` with the following content:

    ```yaml
    http:
      middlewares:
        authentication:
          basicAuth:
            users:
              - "<YOUR_USER>:<YOUR_HASHED_PASSWORD>"
    ```

    The `users` field is an array of authorized users. Each user must be declared using the `name:hashed-password` format. See the [BasicAuth](https://doc.traefik.io/traefik/middlewares/http/basicauth/#configuration-examples) documentation. 
    
    To generate the `name:hashed-password` string you can use an online HTPasswd Generator, like [this](https://www.web2generators.com/apache-tools/htpasswd-generator).

2. Create the file `/etc/traefik/dynamic/dashboard.yml`, with the following content:

    ```yaml
    http:
      routers:
        # Overrides the default 'api' router
        api:
          # The rule matches http://example.com/api/ or http://example.com/dashboard/
          # but does not match http://example.com/hello
          rule: Host(`traefik.{{ env "DUCKDNS_DOMAIN"}}.duckdns.org`) && (PathPrefix(`/api`) || PathPrefix(`/dashboard`))
          entrypoints:
            - web
            - websecure
            # - traefik  # uncomment to use the :8080 port
          service: api@internal
          middlewares:
            - authentication
          tls:
            certResolver: "duckdnsResolver"
            domains:
              - main: "{{ env "DUCKDNS_DOMAIN"}}.duckdns.org"
                sans:
                  - "*.{{ env "DUCKDNS_DOMAIN"}}.duckdns.org"
    ```

    > 💡 Using this configuration the dashboard will be available at the address
    >
    >> https://traefik.DUCKDNS_DOMAIN.duckdns.org/dashboard/
    >
    > ⚠️ Note the trailing `/`: without it you should receive a _404 Not found_ error

  1. Set file permissions
  
      ```bash
      $ sudo chown -R traefik:traefik /etc/traefik/dynamic
      $ sudo chmod -R 644 /etc/traefik/dynamic
      $ sudo chmod 755 /etc/traefik/dynamic
      ```

### Setup systemd service
1. Create the file `/etc/systemd/system/traefik.service` with the following content

    <details>
    <summary>✨ Click to see the code</summary>

    ```ini
    [Unit]
    Description=traefik proxy
    After=network-online.target
    Wants=network-online.target systemd-networkd-wait-online.service

    [Service]
    Restart=on-abnormal
    ; User and group the process will run as.
    User=traefik
    Group=traefik
    ; Set environment variables
    EnvironmentFile=/etc/traefik/traefik.conf
    PassEnvironment=DUCKDNS_DOMAIN DUCKDNS_TOKEN
    ; Always set "-root" to something safe in case it gets forgotten in the traefik file.
    ExecStart=/usr/local/bin/traefik --configfile=/etc/traefik/traefik.yml
    ; Limit the number of file descriptors; see `man systemd.exec` for more limit settings.
    LimitNOFILE=1048576
    ; Use private /tmp and /var/tmp, which are discarded after traefik stops.
    PrivateTmp=true
    ; Use a minimal /dev (May bring additional security if switched to 'true', but it may not work on Raspberry Pi's or other devices, so it has been disabled >PrivateDevices=false
    ; Hide /home, /root, and /run/user. Nobody will steal your SSH-keys.
    ProtectHome=true
    ; Make /usr, /boot, /etc and possibly some more folders read-only.
    ProtectSystem=full
    ; … except /etc/traefik/acme, because we want Letsencrypt-certificates there.
    ;   This merely retains r/w access rights, it does not add any new. Must still be writable on the host!
    ReadWriteDirectories=/etc/traefik/acme
    ; The following additional security directives only work with systemd v229 or later.
    ; They further restrict privileges that can be gained by traefik. Uncomment if you like.
    ; Note that you may have to add capabilities required by any plugins in use.
    CapabilityBoundingSet=CAP_NET_BIND_SERVICE
    AmbientCapabilities=CAP_NET_BIND_SERVICE
    NoNewPrivileges=true

    [Install]
    WantedBy=multi-user.target
    ```

    </details>
2. Create the file `/etc/traefik/traefik.conf` with the following content

    ```env
    DUCKDNS_DOMAIN=<YOUR_DUCKDNS_DOMAIN>
    DUCKDNS_TOKEN=<YOUR_DUCKDNS_TOKEN>
    ```

    ⚠️ Don't forget to edit the `<YOUR_DUCKDNS_DOMAIN>` and `<YOUR_DUCKDNS_TOKEN>` fields. No quotes needed!

3. Set file permissions

    ```bash
    $ sudo chown root:root /etc/systemd/system/traefik.service
    $ sudo chown traefik:traefik /etc/traefik/traefik.conf
    $ sudo chmod 644 /etc/systemd/system/traefik.service /etc/traefik/traefik.conf
    ```

4. Reload systemd and enable the service

    ```bash
    $ sudo systemctl daemon-reload
    $ sudo systemctl enable traefik.service
    $ sudo systemctl start traefik.service
    ```

5. Now Traefik should be up and you should reach the dashboard page at the address

    > https://traefik.DUCKDNS_DOMAIN.duckdns.org/dashboard/

    ⚠️ Note the trailing `/`: without it you should receive a _404 Not found_ error

    If not, check logs for any errors with `journalctl --boot -u traefik.service` or log files at `/var/log/traefik/`

### Configure log rotation
Traefik doesn't rotate log files by default, so we use `logrotate` to rotate the log files.

1. `logrotate` should already be installed on Raspberry OS, you can check this with the command

    ```bash
    $ logrotate -v
    ```

    If not, install it

    ```bash
    $ sudo apt install logrotate
    ```

2. Create the file `/etc/logrotate.d/traefik` (with `sudo`) and paste this content
   
    ```config
    /var/log/traefik/*.log {
      compress
      rotate 7
      daily
      missingok
      notifempty
      dateext
      postrotate
        systemctl kill --signal=USR1 traefik
      endscript
    }
    ```

    💡 This configuration will daily compress the logs files, keeping the last 7 days logs and deleting the oldest ones. See this [logrotate](https://guide.debianizzati.org/index.php/Logrotate:_configurare_la_rotazione_automatica_dei_log) tutorial, and the `man logrotate` command.


### Update Traefik
Update Traefik is simple as replace the binary file with the updated one

1. Stop the service

    ```bash
    $ sudo systemctl stop traefik.service
    ```

2. Follow the first 4 steps of the [install](#install-traefik) guide

### Annex: Add custom dynamic configuration
- Add new [Middlewares](https://doc.traefik.io/traefik/middlewares/overview/) 
  - To the `/etc/traefik/dynamic/middlewares.yml` file if they are global Middlewares (i.e. can be shared between multiple routers)
  - To the Service's dynamic configuration file (see next point) if they are local Middlewares (i.e. related only to a Service's router)

 - To add a new Service
    
    1. Create a new file `service_name.yml` in the `/etc/traefik/dynamic/` folder 
    2. Configure into this file the Service's [Router](https://doc.traefik.io/traefik/routing/routers/) and [Service](https://doc.traefik.io/traefik/routing/services/) (and, eventually, local Middlewares)
    
        Use the following template as reference

        <details>
        <summary>✨ Click to see the code</summary>

        ```yaml
        http:
          routers:
            my-router:
              rule: Host(`new-service.{{ env "DUCKDNS_DOMAIN"}}.duckdns.org`)
              # OR rule: (Host(`{{ env "DUCKDNS_DOMAIN"}}.duckdns.org`) && PathPrefix(`/new-service`))

              # If no entryPoints specified, the router will accept requests from all defined entry points. 
              # If you want to limit the router scope only to some entry points, uncomment these lines
              # entryPoints:
              #  - "websecure"

              service: "my-service"

              # Middlewares to which the request will be forwarded when the route is activated
              # Optional
              middlewares:
                - "authentication"
                - "my-local-middleware"
              
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
            my-service:
              loadBalancer:
                servers:
                - url: "http://<private-ip-server-1>:<private-port-server-1>/"

          # Local middlewares (i.e. middlewares that will be used only by my-router)
          # Optional
          middlewares:
            my-local-middleware:
              addPrefix:
                prefix: "/foo"
        ```

        💡 For an example, see the file `/etc/traefik/dynamic/dashboard.yml` 

        </details>

    3. Set file permissions

        ```bash
        $ sudo chown traefik:traefik /etc/traefik/dynamic/service_name.yml
        $ sudo chmod 644 /etc/traefik/dynamic/service_name.yml
        ```

## Wireguard VPN [🦆]
At its core, all WireGuard does is create an interface from one computer to another. It doesn’t really let you access other computers on either end of the network, or forward all your traffic through the VPN server, or anything like that. It just connects two computers, directly, quickly and securely.

Basically, for each computer (peer) we have to configure an interface, with an IP address, a private key and a listening port. Then we have to list the public keys of all the peers we wants to connect.

The configuration will be written in a `.conf` file. For details about this configuration file, see this [tutorial](https://www.stavros.io/posts/how-to-configure-wireguard/). 

In our setup, we have a server peer (the pi) to which all client peers (the other devices) connect.  

We will follow the official Pi-hole guide for setup Wireguard. All the necessary steps are briefly summarized below, however for a more detailed description, you can follow the [guide](https://docs.pi-hole.net/guides/vpn/wireguard/server/) as well.

### Install Wireguard
1. Install Wireguard

    ```bash
    $ sudo apt update
    $ sudo apt install wireguard wireguard-tools
    ```

2. Use a sudo shell to generate the server public and private keys in the `/etc/wiregurard/` folder

    ```bash
    $ sudo -i
    $ cd /etc/wireguard/
    $ umask 077  # This makes sure credentials don't leak in a race condition
    $ wg genkey | tee server.key | wg pubkey > server.pub
    ```

    💡 This will generate two files, the private key `server.key` file and public key `server.pub` file. The public key is for telling the world, the privatekey file is secret and should stay on the computer it was generated on. You'll need to paste the contents of these files in the config file, since WireGuard doesn’t support referencing them by path yet.

3. Create the `/etc/wireguard/wg0.conf` file and paste the following content

    ```ini
    [Interface]
    Address = 10.100.0.1/24, fd08:4711::1/64
    ListenPort = 47111
    ```  

    💡 In the `[Interface]` section of the config file we configure the peer's wireguard network interface. We assign a static IP address to the interface, so `10.100.0.1` will be our server IP address (where `10.100.0` is the network part). To the other peers we will assign different IPs.

    ⚠️ Don't forget to open the `47111/udp` port on your router

4. Add the server's private key (written in the `server.key` file) to the config file, then exit the sudo session

    ```bash
    $ echo "PrivateKey = $(cat /etc/wireguard/server.key)" >> /etc/wireguard/wg0.conf
    $ exit
    ```

To check everything is running, you can run

```bash
$ sudo wg
```

the output should be something like 

```bash
interface: wg0
  public key: XYZ123456ABC=   ⬅ Your public key will be different
  private key: (hidden)
  listening port: 47111
```

Also, running `ifconfig`, you should see a `wg0` interface listed.

### Add clients
Wireguard creates peer-to-peer connections (called VPN tunnels) between devices. Each peer must have a `.conf` file containing its Wireguard interface configuration (the `[Interface]` section) and list of `[Peer]` sections, one for each peer it will be connected to.

So, to create a connection between our server and client we have make two configuration steps:

1. Create a `.conf` file for the client (also known as `VPN Profile`): this file will contain the client's interface configuration and a `[Peer]` section containing the parameters for the connection with the server
2. Create, in the server's `.conf` file, a `[Peer]` section containing the parameters for the connection with the client

> 💡 For the sake of simplicity, we will create the client's `.conf` file on the server itself. This, however, means that you need to store this config file securely, as it contains the private key of your client. An alternative way of doing this is to generate the `.conf` file locally on your client and add the necessary lines to your server's configuration.


#### Setup the configuration environment
1. Enter in a sudo session an set variables

    ```bash
    $ sudo -i
    $ cd /etc/wireguard/
    $ umask 077
    $ name="client-name"
    ```

    ⚠️ Change the `name` variable value with the name you want give to the new client, for example `vincenzo-laptop`

2. Generate the client's private and public keys

    ```bash
    $ wg genkey | tee "/etc/wireguard/clients/${name}.key" | wg pubkey > "/etc/wireguard/clients/${name}.pub"
    ```

    ⚠️ Create the `/etc/wireguard/clients/` folder if doesn't exists

3. Generate a pre-shared key (PSK) 

    We furthermore generate a pre-shared key (PSK) in addition to the keys above. This adds an additional layer of symmetric-key cryptography to be mixed into the already existing public-key cryptography and is mainly for post-quantum resistance.

    ```bash
    $ wg genpsk > "/etc/wireguard/clients/${name}.psk"
    ```

#### Create the client's `.conf` file
4. Create the client's `.conf` file
   
    ```bash
    $ echo "[Interface]" > "/etc/wireguard/clients/${name}.conf"
    ```

5. Add the IP address to the client's `.conf` file

    The `Address` setting of the `[Interface]` section sets the peer's static IP, so make sure to increment it for each client you add to Wireguard. 

    ```bash
    $ echo "Address = 10.100.0.[X]/32, fd08:4711::[X]/128" >> "/etc/wireguard/clients/${name}.conf"
    ```
    ⚠️ Don't forget to edit the `[X]` part of the address. Increment it for each client you add

6. Add the DNS setting and client's private key to the client's `.conf` file

    ```bash
    $ echo "DNS = 10.100.0.1" >> "/etc/wireguard/clients/${name}.conf"
    $ echo "PrivateKey = $(cat "/etc/wireguard/clients/${name}.key")" >> "/etc/wireguard/clients/${name}.conf"
    ```

    💡 We set the server's IP `10.100.0.1` as `DNS` setting since we will install Pi-hole on the server

7. Add the server's connection parameters as `[Peer]` section of the client's `.conf` file

    Edit the `/etc/wireguard/clients/${name}.conf` file and paste the following content at the end

    ```ini
    [Peer]
    AllowedIPs = 10.100.0.1/32, fd08:4711::1/128  # ⬅ Allows only comunication with the server
    # OR AllowedIPs = 10.100.0.0/24, fd08:4711::1/64 ⬅ Allows comunication between all peers in the wireguard's network
    Endpoint = [your public IP or domain]:47111
    PersistentKeepalive = 25
    ```

    💡 About these parameters:
    - `AllowedIPs`: see the the section [Split Tunnel vs Full Tunnel](#split-tunnel-vs-full-tunnel)
    - `Endpoint`: set your server public IP or domain. Tipically, your duckdns domain
    - `PersistentKeepalive`: see the note on the [pihole](https://docs.pi-hole.net/guides/vpn/wireguard/client/#create-client-configuration) documentation. Briefly, we send a keepalive packet every 25 second to the server to prevent the (router/server's) NAT deletes the client's mapping rule

8. Add the server's public key and the pre-shared key to the client's `.conf` file

    ```bash
    $ echo "PublicKey = $(cat /etc/wireguard/server.pub)" >> "/etc/wireguard/clients/${name}.conf"
    $ echo "PresharedKey = $(cat "/etc/wireguard/clients/${name}.psk")" >> "/etc/wireguard/clients/${name}.conf"
    ```

Now, your client's `.conf` file (i.e. `/etc/wireguard/clients/${name}.conf`) should look like this

```ini
[Interface]
Address = 10.100.0.[X]/32, fd08:4711::[X]/128
DNS = 10.100.0.1
PrivateKey = XYZ123456ABC=

[Peer]
AllowedIPs = 10.100.0.1/32, fd08:4711::1/128
Endpoint = [your public IP or domain]:47111
PersistentKeepalive = 25
PublicKey = XYZ123456ABC=
PresharedKey = XYZ123456ABC=
```

#### Add the new client to the server's `.conf` file
9. Now we have to add a new `[Peer]` section in the server's `.conf` file with the client's connection parameters

    ```bash
    $ echo "# ${name}" >> /etc/wireguard/wg0.conf
    $ echo "[Peer]" >> /etc/wireguard/wg0.conf
    $ echo "PublicKey = $(cat "/etc/wireguard/clients/${name}.pub")" >> /etc/wireguard/wg0.conf
    $ echo "PresharedKey = $(cat "/etc/wireguard/clients/${name}.psk")" >> /etc/wireguard/wg0.conf
    $ echo "AllowedIPs = 10.100.0.[X]/32, fd08:4711::[X]/128" >> /etc/wireguard/wg0.conf
    $ echo "" >> /etc/wireguard/wg0.conf
    ```

    ⚠️ Don't forget to replace the `[X]` part of the `AllowedIPs` setting with the client's IP address set in step 5

The server's `.conf` file (i.e. `/etc/wireguard/wg0.conf`) should look like this

```ini
[Interface]
Address = 10.100.0.1/24, fd08:4711::1/64
ListenPort = 47111
PrivateKey = XYZ123456ABC=

# ... other [Peer] sections

# client-name
[Peer]
PublicKey = XYZ123456ABC=
PresharedKey = XYZ123456ABC=
AllowedIPs = 10.100.0.[X]/32, fd08:4711::[X]/128
```

10. Reload your server config to add the new client, then exit the sudo session

    ```bash
    $ wg syncconf wg0 <(wg-quick strip wg0)
    $ exit
    ```

#### Connect your client and test the connection
You can now copy the client's `.conf` file to your client. If the client is a mobile device such as a phone, `qrencode` can be used to generate a scanable QR code:

```bash
$ sudo -i
$ sudo qrencode -t ansiutf8 < "/etc/wireguard/clients/${name}.conf"
$ exit
```

>💡 You can directly scan this QR code with the official WireGuard app after clicking on the blue plus symbol in the lower right corner


After creating/copying the `.conf` file to your client, you may use the client you prefer to connect to your sever.

You can check if your client successfully connected by running

```bash
$ sudo wg
```

on the server. If everything works, it should show some traffic for your client:

```bash
interface: wg0
  public key: XYZ123456ABC=          ⬅ Your server's public key will be different
  private key: (hidden)
  listening port: 47111

peer: XYZ123456ABC=   ⬅ Your peer's public key will be different
  preshared key: (hidden)
  allowed ips: 10.100.0.2/32
  latest handshake: 32 seconds ago
  transfer: 3.43 KiB received, 188 B sent
```

### (Optional) Route all internet traffic through the VPN Tunnel
>💡 This section is optional: if you want to route only the DNS queries through the VPN, you can skip this part

#### Split Tunnel vs Full Tunnel
In the steps above we have configured a *split tunnel*. In this configuration, only DNS packets are routed through the tunnel, while the internet trafic still remains free. Instead, in a *full tunnel* all the internet traffic is routed through the tunnel.

That's mainly configured by the `AllowedIPs` setting, on the client `.conf` file. 

The `AllowedIPs` setting acts as [a routing table when sending packets, and an ACL when receiving packets](https://www.wireguard.com/#cryptokey-routing). When a peer tries to send a packet to an IP, it will check `AllowedIPs`, and if that IP appears in the list, the packet will be routed through the WireGuard interface. When the peer receives a packet over the wiregurad interface, it will check `AllowedIPs` again, and if the source address is not in the list, it will be dropped.

We need to change some roules on the server's firewall to allow the packets forwarding, but basically to route the packets trough the tunnel we need simply to edit the client's `AllowedIPs` setting.

#### Enable IP forwarding on the server
1. Edit the file `/etc/sysctl.d/99-sysctl.conf` and uncomment the lines

    ```ini
    net.ipv4.ip_forward = 1
    net.ipv6.conf.all.forwarding = 1
    ```

2. Save the file and apply the new options

    ```bash
    $ sudo sysctl -p
    ```

    If you see the options repeated like
    
    ```bash
    net.ipv4.ip_forward = 1
    net.ipv6.conf.all.forwarding = 1
    ```

    they were enabled successfully.

Now your Wireguard server is able to forward the packets coming from the wireguard interfce `wg0` to other interfaces (like the `eth0`).

#### Enable NAT on the server
Now we have to edit the Wireguard server’s firewall to add rules that will ensure traffic to and from the server and clients is routed correctly.

We will use the Wireguard `PostUp` and `PreDown` configuration settings. The `PostUp` lines will run when the Wireguard server starts, the `PreDown` lines run when the Wireguard Server stops the VPN inteface. In this way we can add the NAT rules when the wireguard interface is up and delete them when the wireguard interface is taken down.

1. Stop the `wg0` interface

    ```bash
    $ sudo systemctl stop wg-quick@wg0.service
    ```

2. Edit the `/etc/wireguard/wg0.conf` file and add these lines under the `[Interface]` section

    ```bash
    PostUp = nft add table ip wireguard; nft add chain ip wireguard wireguard_chain {type nat hook postrouting priority srcnat\; policy accept\;}; nft add rule ip wireguard wireguard_chain counter packets 0 bytes 0 masquerade
    PostUp = nft add table ip6 wireguard; nft add chain ip6 wireguard wireguard_chain {type nat hook postrouting priority srcnat\; policy accept\;}; nft add rule ip6 wireguard wireguard_chain counter packets 0 bytes 0 masquerade
    PreDown = nft delete table ip wireguard
    PreDown = nft delete table ip6 wireguard
    ```

    ⚠️ You may need to change the `eth0` interface with the one you use to connect to the LAN and to internet (for example, if you use wifi instead of ethernet). You can find the correct interface with the command `ip route list default`

3. Restart the `wg0` interface

    ```bash
    $ sudo systemctl start wg-quick@wg0.service
    ```

>💡 Some details about the lines we have added to the configuration file:
> - `nft add table ip wireguard; ...`: this rule configures *masquerading*, i.e. rewrites IPv4 traffic that comes in on the wireguard `wg0` interface to make it appear like it originates directly from the Wireguard server’s public IPv4 address
> - `nft add table ip6 wireguard; ...`: same as the previous one, but for IPv6


#### Accessing your home LAN
>⚠️ The following assumes you have already prepared your server for [IP forwarding](#enable-ip-forwarding-on-the-server) and [enabled NAT](#enable-nat-on-the-server).

Simply edit the client's `.conf` file (**NOT the server's one**) and add your home network IP (generally, `192.168.1.0/24`) to the `AllowedIPs` setting in the `[Peer]` section.

Example of a client's `.conf` file configured with LAN access. Note the `AllowedIPs` setting:
```ini
[Interface]
Address = 10.100.0.[X]/32, fd08:4711::[X]/128
DNS = 10.100.0.1
PrivateKey = XYZ123456ABC=

[Peer]
AllowedIPs = 10.100.0.1/32, fd08:4711::1/128, 192.168.1.0/24
Endpoint = [your public IP or domain]:47111
PersistentKeepalive = 25
PublicKey = XYZ123456ABC=
PresharedKey = XYZ123456ABC=
```

With this configuration, when your client (outside the LAN) tries to send a packet to an IP address into your LAN (i.e. its destination address is in the `192.168.1.0` range), that packet will be intercepted by the wireguard interface and routed through the tunnel to the server. Then, the server will forward it to the right LAN client.

>💡 You can also add this only to some client's `.conf` files. In this way, you'll enable the LAN access feature for only some clients, leaving the other ones isolated from your LAN.

Use this command to quickly edit the `AllowedIPs` setting of a client's `.conf`file:

```bash
$ sudo sed -i '/^AllowedIPs =/ s/$/, 192.168.1.0\/24/' /etc/wireguard/clients/CLIENT-CONF.conf
```

Or, use this command to quickly enable the LAN access feature for all the client's `.conf` files in your `/etc/wireguard/clients/` folder

```bash
$ sudo find /etc/wireguard/clients/ -type f -name "*.conf" -exec sed -i '/^AllowedIPs =/ s/$/, 192.168.1.0\/24/' {} \;
```

>⚠️ Don't forget to share the edited `.conf` file with your client

#### Route the entire Internet traffic through the WireGuard tunnel
>⚠️ The following assumes you have already prepared your server for [IP forwarding](#enable-ip-forwarding-on-the-server) and [enabled NAT](#enable-nat-on-the-server).

Simply edit the client's `.conf` file (**NOT the server's one**) and add the default route (`0.0.0.0/0` for IPv4 and `::/0` for IPv6) to the `AllowedIPs` in the [Peer] section.

```ini
# you can remove all other IPs, since the default route already include them
AllowedIPs = 0.0.0.0/0, ::/0
```

With this configuration, all packets sent by the client will be intercepted by the wireguard interface and routed through the tunnel.

>💡 Instead of editing your existing client's configuration, you can create a copy with the modified `AllowedIPs` line as above. This will give you two VPN profiles and you can decide - at any time from mobile - which variant you want. 

Use this command to quickly create a copy of a client's `.conf` file, with the `AllowedIPs` setting edited to route all packets through the tunnel:

```bash
$ sudo sed '/AllowedIPs =/c\AllowedIPs = 0.0.0.0/0, ::/0' /etc/wireguard/clients/CLIENT-CONF.conf > /etc/wireguard/clients/CLIENT-CONF-full.conf
```

For example, if you have a `vincenzo-laptop.conf`:

```bash
sudo sed '/AllowedIPs =/c\AllowedIPs = 0.0.0.0/0, ::/0' /etc/wireguard/clients/vincenzo-laptop.conf > /etc/wireguard/clients/vincenzo-laptop-full.conf
```

>⚠️ Don't forget to share the edited `.conf` file with your client

### Bonus: Add new clients with a single command
Use this script to add new clients with a single command

<details>
<summary>✨ Click to see the code</summary>

```bash
#! /usr/bin/env bash
# Usage: ./wg-add.sh [-h] [-f] peer_id peer_name server_domain
# Add new peers to the wireguard network
#
#   -h                               Show this help and exits
#   -f               OPTIONAL        Force overwrite of (eventually) existing configuration files for the same peer_name
#
#   peer_id          REQUIRED        Peer ID. Must be an integer. Will be used for the peer's IP address
#   peer_name        REQUIRED        String to identify the peer
#   server_domain    REQUIRED        Public IP or DOMAIN of the server
#
# Examples:
#          ./wg-add.sh 2 "peters-laptop" "domain.example.com"
#          ./wg-add.sh -f 3 "annas-android" "domain.example.com"
#
# Note: This script must be run in a sudo -i session

# Global parameters
FORCE='false'
SUBNET_IPV4="10.100.0."
SUBNET_IPV6="fd08:4711::"
LOCAL_LAN="192.168.1.0\/24"

display_usage() {
	echo -e "Usage: $0 [-h] [-f] peer_id peer_name server_domain"
        echo -e "Add new peers to the wireguard network"
        echo -e "  "
        echo -e "  -h \t \t \t \t Show this help and exits"
        echo -e "  -f \t \t OPTIONAL \t Force overwrite of (eventually) existing configuration files for the same peer_name. Keys will NEVER be overwritten!"
        echo -e "  "
        echo -e "  peer_id \t REQUIRED \t Peer ID. Must be an integer. Will be used for the peer's IP address"
        echo -e "  peer_name \t REQUIRED \t String to identify the peer"
        echo -e "  server_domain  REQUIRED \t Public IP or DOMAIN of the server"
        echo -e "  "
        echo -e "Examples:"
        echo -e "         $0 2 \"peters-laptop\" \"domain.example.com\""
        echo -e "         $0 -f 3 \"annas-android\" \"domain.example.com\""
        echo -e "  "
        echo -e "Note: This script must be run in a sudo -i session"
}

# Parse command line options
while getopts ':fh' OPTION
do
  case "${OPTION}" in
    f)
      FORCE='true'
      ;;
    h)
      display_usage
      exit 0
      ;;
   \?) echo "$0: Error: Invalid option: -${OPTARG}" >&2; exit 1;;
    :) echo "$0: Error: option -${OPTARG} requires an argument" >&2; exit 1;;
  esac
done

shift $((OPTIND - 1))

# if less than three arguments supplied, display usage
if [ $# -le 2 ]
then
        echo -e "Wrong number of arguments!\n">&2
        display_usage
        exit 1
fi

# FROM NOW THE SCRIPT MUST BE RUNNED AS ROOT USER
# display usage if the script is not run as root user
if [[ "$EUID" -ne 0 ]]; then
        echo "This script must be run in a sudo -i session!">&2
        exit 1
fi

echo "** Wireguard peers configuration script **"

peer_id="$1"
ipv4="${SUBNET_IPV4}${peer_id}"
ipv6="${SUBNET_IPV6}${peer_id}"
server4="${SUBNET_IPV4}1"
server6="${SUBNET_IPV6}1"
target="${3}:47111"
name="$2"

serverpubkey="/etc/wireguard/server.pub"
privkey="/etc/wireguard/clients/${name}.key"
pubkey="/etc/wireguard/clients/${name}.pub"
pskey="/etc/wireguard/clients/${name}.psk"

serverconf="/etc/wireguard/wg0.conf"
clientconf="/etc/wireguard/clients/${name}.conf"
clientfullconf="/etc/wireguard/clients/${name}-full.conf"

echo -e "Generating configuration files for ${peer_id}/${name} [server: ${target}]"
echo -e "  "

umask 077

if ${FORCE}; then
    echo -e "Force Mode:"
    if [[ -f ${clientconf} ]]; then
      echo -e "  Existing configuration file '${name}.conf' will be overwritten"
      rm ${clientconf}
    fi
    if [[ -f ${clientfullconf} ]]; then
      echo -e "  Existing configuration file '${name}-full.conf' will be overwritten"
      rm ${clientfullconf}
    fi
    echo -e "  "
fi

if [[ ! -f ${clientconf} && ! -f ${clientfullconf} ]]; then
    # Generate client's keys
    echo "Generating client's keys:"

    if [[ ! -f ${privkey} && ! -f ${pubkey} ]]; then
      wg genkey | tee ${privkey} | wg pubkey > ${pubkey}
      echo "  Private Key and Public Key files generated successfully"
    else
      echo "  Private Key and Public Key files already exists"
    fi

    if [[ ! -f ${pskey} ]]; then
      wg genpsk > ${pskey}
      echo "  PreShared Key file generated successfully"
    else
      echo "  PreShared Key file already exists"
    fi
    echo -e "  "

    # Create client's config files
    echo "Generating client's configuration files:"

    # Split tunnel
    echo "[Interface]" > ${clientconf}
    echo "Address = ${ipv4}/32, ${ipv6}/128" >> ${clientconf}
    echo "DNS = ${server4}, ${server6}" >>  ${clientconf} #Specifying DNS Server
    echo "PrivateKey = $(cat ${privkey})" >> ${clientconf}
    echo "" >> ${clientconf}
    echo "[Peer]" >> ${clientconf}
    echo "PublicKey = $(cat ${serverpubkey})" >> ${clientconf}
    echo "PresharedKey = $(cat ${pskey})" >> ${clientconf}
    echo "Endpoint = ${target}" >> ${clientconf}
    echo "AllowedIPs = ${server4}/32, ${server6}/128" >> ${clientconf} # clients isolated from one another
    # echo "AllowedIPs = ${subnet_ipv4}0/24, ${subnet_ipv6}/64" >> ${clientconf} # clients can see each other
    echo "PersistentKeepalive = 25" >> ${clientconf}

    echo "  Split tunnel configuration file generated successfully"

    # Full tunnel
    sed '/AllowedIPs =/c\AllowedIPs = 0.0.0.0/0, ::/0' ${clientconf} > ${clientfullconf}

    echo "  Full tunnel configuration file generated successfully"
    echo -e "  "

    read -p "Enable LAN access? [y/n]? " -n 1 -r CONT
    echo
    if [[ $CONT =~ ^[Yy]$ ]]
    then
      sed -i "/^AllowedIPs =/ s/$/, ${LOCAL_LAN} /" ${clientconf}
      echo -e "Local LAN access enabled"
      echo -e "  "
    fi

    # Add to the server's configuration
    echo "Adding the client to the server's peers list"
    echo "# $name" >> "${serverconf}"
    echo "[Peer]" >> "${serverconf}"
    echo "PublicKey = $(cat ${pubkey})" >> "${serverconf}"
    echo "PresharedKey = $(cat ${pskey})" >> "${serverconf}"
    echo "AllowedIPs = ${ipv4}/32, ${ipv6}/128" >> "${serverconf}"
    echo "" >> "${serverconf}"

    echo -e "  "

    echo -e "Securing files"
    chown -R root:root ${privkey} ${pubkey} ${pskey} ${clientconf} ${clientfullconf}
    chmod -R og-rwx ${privkey} ${pubkey} ${pskey} ${clientconf} ${clientfullconf}

    echo -e " "

    echo "Reloading server configuration"
    wg syncconf wg0 <(wg-quick strip wg0)
else
    echo -e "Configuration files already exists"
fi

echo -e "  "

echo -e "New Wireguard client added successfully!"
echo -e "  Client IP Addresses: ${ipv4}, ${ipv6}"
echo -e "  Split Tunnel configuration file: ${clientconf}"
echo -e "  Full Tunnel configuration file: ${clientfullconf}"
echo -e "  "

# Print QR code scannable by the Wireguard mobile app on screen
read -p "Print scannable QR codes [y/n]? " -n 1 -r CONT
echo
if [[ $CONT =~ ^[Yy]$ ]]
then
  echo -e "Printing shareable QR codes"
  echo -e "  "

  echo -e "QR code for: ${clientconf}"
  qrencode -t ansiutf8 < ${clientconf}

  echo -e "  "
  echo -e "QR code for: ${clientfullconf}"
  qrencode -t ansiutf8 < ${clientfullconf}

  echo -e "  "
  echo -e "Scan them with the Wireguard mobile app"
fi
```

</details>

To install, paste the content of the script above in a `wg-add.sh` file elsewhere, then, copy it to the `/usr/local/bin`:

```bash
$ sudo chmod +x wg-add.sh
$ sudo cp wg-add.sh /usr/local/bin/wg-add
$ sudo chown root:root /usr/local/bin/wg-add
$ sudo chmod 755 /usr/local/bin/wg-add
```

Then, use it in a `sudo -i` session

```bash
$ sudo -i
$ wg-add id "name" "domain"
```

## Install Pi-hole
1. Make sure that your system is fully updated (with `apt`)
2. Install Pi-hole (from official [guide](https://docs.pi-hole.net/main/basic-install/))

    ```bash
	$ curl -sSL https://install.pi-hole.net | bash
	```

   **NB**: Take note of the admin password (shown at installation end).
   
3. 

4. Install [whitelist](https://github.com/anudeepND/whitelist)

    ```bash
	$ git clone https://github.com/anudeepND/whitelist.git
    $ sudo python3 whitelist/scripts/whitelist.py
	```

Notes: 
- Current block lists are taken from [here](https://www.andreadraghetti.it/block-list-e-white-list-per-pi-hole-e-ad-blocker/).

##

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
- ` wget -O /home/raspi/[JD_Install_dir]/jDownloader.png http://jdownloader.org/_media/knowledge/wiki/jdownloader.png`

- Go to desktop

- `nano jDownloader.desktop`

- Paste this:

``` bash
[Desktop Entry]
Encoding=UTF-8
Name=jDownloader2
Comment=jDownloader2
Exec=bash -c "java -jar /home/raspi/[JD_install_dir]/JDownloader.jar"
Icon=/home/raspi/[JD_install_dir]/jDownloader.png
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