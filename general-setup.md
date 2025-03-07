# General Setup
⚠️ Please read before [First operations](#first-operations).

💡 Services with the duck [🦆] symbol have your duckdns domain hardcoded in some configuration files. You may need to reconfigure them if you change your duckdns domain.

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
    # $ sudo mkdir /media/dnas
    $ sudo mkdir /media/qnas
    $ sudo mkdir /media/snas
    $ sudo mkdir /media/qnas/Download
    $ sudo mkdir /media/qnas/Media
    $ sudo mkdir /media/qnas/Vincenzo
    $ sudo mkdir /media/qnas/Vincenzo-Home
    $ sudo mkdir /media/snas/Immich-Library  # Only if Immich is installed
    $ sudo mkdir /media/snas/Vincenzo
    ```

- Create the credentials files, in a `~/.credentials` folder (create if not exists)
    One for each network device (if they have different credentials)

    - `nano ~/.credentials/.qnas-<user>`

        ```bash
        user=<YOUR-USER>
        password=<YOUR-PASSWORD>
        domain=WORKGROUP
        ```

    - Give access only to the user

        ```bash
        $ chmod 600 ~/.credentials/.qnas-<user>
        ```

    - Repeat for each network share you want to login

- Run `sudo nano /etc/fstab` and add these lines (changes paths as done in point 2)

    ``` 
    #//192.168.1.200/Volume_1         /media/dnas/                   cifs    credentials=/home/raspi/.credentials/.dnascredentials,uid=raspi,gid=raspi,iocharset=utf8,file_mode=0755,dir_mode=0755,noperm,vers=1.0   0   0
    //192.168.1.210/Multimedia        /media/qnas/Media/             cifs    credentials=/home/raspi/.credentials/.qnas-vincenzo,uid=raspi,gid=raspi,iocharset=utf8,file_mode=0755,dir_mode=0755,noperm            0   0
    //192.168.1.210/Download          /media/qnas/Download/          cifs    credentials=/home/raspi/.credentials/.qnas-vincenzo,uid=raspi,gid=raspi,iocharset=utf8,file_mode=0755,dir_mode=0755,noperm            0   0
    //192.168.1.210/homes/vincenzo    /media/qnas/Vincenzo-Home/     cifs    credentials=/home/raspi/.credentials/.qnas-vincenzo,uid=raspi,gid=raspi,iocharset=utf8,file_mode=0755,dir_mode=0755,noperm            0   0
    //192.168.1.210/Vincenzo          /media/qnas/Vincenzo/          cifs    credentials=/home/raspi/.credentials/.qnas-vincenzo,uid=raspi,gid=raspi,iocharset=utf8,file_mode=0755,dir_mode=0755,noperm            0   0
    //192.168.1.200/Vincenzo          /media/snas/Vincenzo/          cifs    credentials=/home/raspi/.credentials/.snas-vincenzo,uid=raspi,gid=raspi,iocharset=utf8,file_mode=0755,dir_mode=0755,noperm            0   0
    //192.168.1.200/Immich-Library    /media/snas/Immich-Library/    cifs    credentials=/home/raspi/.credentials/.snas-vincenzo,uid=raspi,gid=raspi,iocharset=utf8,file_mode=0755,dir_mode=0755,noperm            0   0

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


  > [!WARNING]
  > Don't forget to edit the `<YOUR_DUCKDNS_DOMAINS>` and `<YOUR_DUCKDNS_TOKEN>` fields. No quotes needed. 
    
  > [!TIP]
  > `<YOUR_DUCKDNS_DOMAINS>` can be a comma separated (**NO spaces**) list of domains

4. Make the script executable

    ```bash
    $ chmod 700 duck.sh duck.conf.sh
    ```

5. Test the script

    ```bash
    $ . $HOME/.duckdns/duck.conf.sh; $HOME/.duckdns/duck.sh
    ```

  > [!WARNING]
  > Note the leading dot `.`

  > [!TIP]
  > 💡Check the result in the `log.log` file

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

5. Start the interface

    ```bash
    $ sudo systemctl start wg-quick@wg0.service
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
