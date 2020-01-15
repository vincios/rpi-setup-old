# Common configurations for Raspberry Pi with Raspbian

## VNC "cannot currently show the desktop" in headless mode
Run `raspi-config` and change screen resolution to 1920x1080

## AutoMount Nas folders
- Follow points 1, 2, 3 [here](http://timlehr.com/auto-mount-samba-cifs-shares-via-fstab-on-linux/)
- Run `sudo /etc/fstab` and add these lines (changes paths as done in point 2)

``` 
//192.168.1.200/home	/media/nas/home/	cifs	credentials=/home/pi/.nascredentials,uid=pi,gid=pi,iocharset=utf8,file_mode=0777,dir_mode=0777,noperm,vers=1.0	0	0
//192.168.1.200/home_2	/media/nas/home_2/	cifs	credentials=/home/pi/.nascredentials,uid=pi,gid=pi,iocharset=utf8,file_mode=0777,dir_mode=0777,noperm,vers=1.0	0	0
//192.168.1.200/share	/media/nas/share/	cifs	credentials=/home/pi/.nascredentials,uid=pi,gid=pi,iocharset=utf8,file_mode=0777,dir_mode=0777,noperm,vers=1.0	0	0
```

- Run `raspi-config` and enable "Wait for Network at Boot" under "Boot options"
- Check if OK with `sudo mount -a`


## Samba share
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

``` bash 
timestamp() {
  date +"%Y-%m-%d %H-%M-%S"
}

echo url="https://www.duckdns.org/update?domains=vncs10&token=b6e9eba3-1b42-4c66-898a-0d5204833f36&ip=" | curl -k -o /home/pi/duckdns/log.log -K -
echo " | Last run: $(timestamp)" >> /home/pi/duckdns/log.log
```

- Change crontab string as follows:

``` bash 
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
First install *TOR* (See [Build Tor](#build-tor)) and *obfs4proxy*

**NEW**:
- Simply copy BridTools folder on Pi and run `./install.sh` inside it
- In `config.ini` file add these lines in *BASE* section:

```
TOR_DATA_DIR = /usr/local/share/tor
TOR_DIR = /usr/local/bin
TOR_PLUG_DIR = /usr/bin
```

OLD:

- Copy scripts folder to Pi and cd to that folder

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
See [here](https://support.jdownloader.org/Knowledgebase/Article/View/52/0/install-jdownloader-on-nas-and-embedded-devices).
- Run in headless mode with `java -jar JDownloader.jar &` 

OR

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
 *OR* compile it manually (better):

From [here](https://www.ixsystems.com/community/threads/guide-jdownloader2-in-11-1-release-iocage-with-rar5-working.74073/) (or [here](https://board.jdownloader.org/showpost.php?p=446708&postcount=465)):

- Install cmake and openjdk8-jdk

``` bash
sudo apt-get install cmake
sudo apt-get install openjdk8-jdk
```

- Go to JD folder and make folders:

``` bash
mkdir rar5
cd rar5
```

- Clone repository

``` bash
git clone https://github.com/borisbrodski/sevenzipjbinding.git sevenzipbinding
```

**NB**: would be better to choose one branch between `bind_16.02` and `migrate-to-15.09-try2` but it should work also with `master` branch.
So first try with `master` then, in case of failure, try `bind_16.02` and, as last chance, `migrate-to-15.09-try2`.

You can change branch with the `git checkout [branch_name]` command.

- Run CMake

``` bash
cmake . -DJAVA_JDK=/usr/lib/jvm/java-8-openjdk-armhf`
make
make package
```

- Move jar library to JD lib folder

``` bash
unzip sevenzipjbinding-16.02-2.01beta-Linux-arm.zip
cd sevenzipjbinding-16.02-2.01beta-Linux-arm/lib/
mv sevenzipjbinding.jar [JD_install_dir]/libs/sevenzipjbinding1509.jar
mv sevenzipjbinding-Linux-arm.jar [JD_Install_dir]/libs/sevenzipjbinding1509LinuxArmVersion.jar
```

**NB**: If you change branch, rename commands above with correct version numbers. But **DON'T** change destination filenames (*sevenzipjbinding1509.jar* and *sevenzipjbinding1509LinuxArmVersion.jar*)


## Update Node and npm
- Remove old node

``` bash
sudo npm uninstall -g npm
sudo apt remove npm
sudo apt-get autoremove
```

- Install new version (From [here](https://github.com/nodesource/distributions/blob/master/README.md))

``` bash
curl -sL https://deb.nodesource.com/setup_13.x | sudo -E bash -
sudo apt-get install -y nodejs
```