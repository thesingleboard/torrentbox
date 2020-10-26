# Torrent box

An automated torrent box that can be used from anywhere. Dump your torrents into a sync folder and automatically pull down torrents.

## Background

In a mobile world it can be cumbersome to download a large torrent file, especially while out on the go. It can also be against the rules of some public wifi access points to do so.
So, in oder to get around these limitations you will need to have a torrent base station set up at home that can act like a torrent concierge.

## Parts Needed

1. Single board computer(SBC) - in this walkthrough Raspberry PI 3(PI3) is used, but any should work.
2. USB flash drive or HD - we will be using a sandisk 64GB flash drive but any size should work.
3. Owncloud Community edition or Google Drive account.
4. Transmission torrent client
5. Raspberry Pi OS - used on PI3

## Overall architecture

![Torrentbox](architecture.png)

## Configuration

In this example we will set up OwnCloud as our sync mechanism. If you want to use Dropbox or GDrive, the set up will be very similar.

### Set up OwnCloud
In order to run OwnCloud efficently, it will run in Docker.

**Install Docker on RaspberryPi OS**

*NOTE: In this example we will use Docker on a PI3 armv7l architecture.*

Full Docker install instructions can be found here. [Docker Install](https://docs.docker.com/engine/install/debian/)

```
$ apt-get update

$ apt-get upgrade

# Docker provided install script
$ curl -sSL https://get.docker.com | sh

# Set up the docker user
$ sudo usermod -aG docker pi

$ reboot
```

Once Docker is set up and running, we will install OwnCloud

**Set Up OwnCloud**

Starting OwnCloud with Docker is very straight forward. Run the following command.

*NOTE: You will need to choose a storage location on your local file system.*

```
$docker pull owncloud:latest

# Make sure the pulled image is arm architecture.
$docker inspect <image id>

$ docker run -d -p 443:443 owncloud:latest -v /location/on/local/host:/var/www/html
```

### Set up Transmission

Transmission is the Bit Torrent client we will use to process our torrent files. We are useing transmission because it is reliable, easy to set up, and runs as a daemon.

```
$ sudo apt-get install transmission-daemon

# The daemon will start up by default, and needs to be stopped.
$ sudo systemctl stop transmission-daemon

# Edit the JSON config
$ vi /etc/transmission-daemon/settings.json

# Edit the following keys
"bind-address-ipv4": "0.0.0.0"
"dht-enabled": true
"download-dir": "/home/pi/Downloads/torrentbox"
"incomplete-dir": "/home/pi/Downloads/torrentbox/torrent-inprogress"
"rpc-bind-address": "0.0.0.0"
"rpc-port": 9091
"rpc-url": "/transmission/"
"rpc-username": "transmission"
"rpc-whitelist": "127.0.0.1"

# Do not restart the daemon yet.
```

### Create volume mount (optional)

If you have an external NAS device, you can create a share and mount it to your Raspberry PI. In this example we are using NFS to share a folder from the NAS.
We then mount it to a folder in the home directory. To do this, and make sure it is automatically mounted at boot we will need to edit the fstab folder. This folder will be
used to pull the torrent files to.
```
$ mkdir -p /home/pi/Downloads/torrentbox

$ sudo echo 'nashost_ip:/data/torrentbox /home/pi/Downloads/torrentbox nfs defaults 0 0' >> vi /etc/fstab

$ sudo mount -a
```

## Hook everything together

Once everything is setup, we need to hook it all together.

### Create an owncloud sync dir



### Start up Transmission

When you start Transmission the write directory will be torrentbox directory, and the read directory will be the OwnCloud Torrent directory. In Daemon
mode Transmission will automatically pick up the torrent file from the Torrent directory, and pull the payload down to the torrentbox directory.

```
#Start Transmission as a daemon and make sure it can see your owncloud sync directory
$ transmission-daemon --help

$ transmission-daemon -c /home/pi/ownCloud/Torrent/ -w /home/pi/Downloads/torrentbox/ -f -ep -gsr 0.0 -T --log-debug
```

Once Transmission is set up you can access it via a web interface to check the status.

http://localhost:9091/transmission

## Set up your clients




## Check Downloads