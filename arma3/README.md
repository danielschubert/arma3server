# Arma 3 server Dockerfile

## Introduction

This dockerfile install Steam CLI and download Arma 3 server content and execute it. 
The container needs two environment variables to work properly:
```
STEAM_LOGIN
STEAM_PASSWORD
```
containing the login and password to connect to a steam account.
Because it use a volume to store the Steam and Arma 3 data, you will need to create a directory on the host filesystem to bind the volume.

## Building from source

To build from source you need to clone the git repo and run docker build:
```
git clone https://github.com/poooldo/arma3server.git
cd arma3server
docker build -t arma3server .
```

## Running

First, you need to create the mountpoint used by the docker volume (I use /home/steam but you can change this accordingly in the Dockerfile):
```
# mkdir /home/steam
# chown 2000:2000 /home/steam/
```

**Note:** We use 2000 as uid/guid which is the same when creating the steam user in the container. We will need around 2GB of memory to download Arma 3 content.

Run the container using root or sudo:
```
docker run -e STEAM_LOGIN='your_steam_login' -e STEAM_PASSWORD='your_steam_password' -p 0.0.0.0:2302:2302/udp -v /home/steam:/home/steam -d -t arma3server
```

After the first run, the arma3 won't be touched. To manually tells the container you want to update all ARMA3 data, just declare $ARMA3_UPDATE_DATA on the next container run.

## Customizing server.cfg

In this vanilla version, Arma 3 server.cfg is a basic configuration file. You need to define $ARMA3_SERVER_CFG to overload its behavior.

You can also define $SERVER_UPDATE_PASSWORD when running the container to generate automatic random password for admin and command. These passwords will be printed into the logs.

## Running Mods

To create a new image running a specific mod:
- create an image based on this one.
- add a script setting up things for the mod (ADD mymod.sh /tmp/mymod.sh)
- declare $MOD_SCRIPT pointing to the setup script (ENV MOD_SCRIPT /tmp/mymod.sh)
- overload server.cfg by adding yours (ADD server.cfg /tmp/server.cfg)
- install needed services (i.e. MySQL,...)

As soon as AltisLife image is pushed, you'll got an example.

## Logging

All logs print out in stdout/stderr and are available via the docker logs command:
```
docker logs <CONTAINER_NAME>
```

If everything works fine (after start.sh has finished to download Steam and Arma 3), you should see something like:
```
15:36:29 Game Port: 2302, Steam Query Port: 2303
15:36:29 Initializing Steam server - Game Port: 2302, Steam Query Port: 2303
Arma 3 Console version 1.54 : port 2302
15:36:30 Connected to Steam servers
```

## Versions

- GNU/Linux Debian Jessie: **8.x**
