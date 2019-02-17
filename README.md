# dockernymous-xrdp

This is a fork of [bcapptain](https://github.com/bcapptain)'s [dockernymous](https://github.com/bcapptain/dockernymous) project, whereas the vnc functionality has been replaced with a rdp implementation.

## About

Dockernymous is a start script for Docker that runs and configures two individual Linux containers in order act as a anonymisation workstation-gateway set up.

The gateway container acts as a Anonymizing Middlebox (see
[trac.torproject.org/projects/tor/wiki/doc/TransparentProxy](https://trac.torproject.org/projects/tor/wiki/doc/TransparentProxy)) and routes ALL traffic from the workstation container through the Tor Network.

The idea was to create a whonix-like setup (see [whonix.org](https://www.whonix.org)) that runs on
systems which aren't able to efficiently run two hardware virtualized machines or don't have virtualization capacities at all.

## Usage / Instructions

It's aimed towards experienced Linux/Docker users, security professionals and penetration testers!

### Host Machine

**Basic Requirements:**

- Docker with linux-based vm
- Remote Desktop viewer (based on the RDC protocol, e.g. [FreeRDP](https://github.com/FreeRDP/FreeRDP))
- Xterm
- Curl

**Example Construction:**

```bash
# Clone the repository
$ git clone https://github.com/dotWee/dockernymous-xrdp dockernymous-xrdp
$ cd ./dockernymous-xrdp

# Create a non-default isolated docker network
$ docker network create --driver=bridge --subnet=192.168.0.0/24 docker_internal
```

### The Gateway

**Basic Requirements:**

- Linux (e.g. Alpine, Debian)
- tor
- procps
- ncat
- iptables

**Example Construction:**

```bash
# This gateway will be based upon Aline; lets fetch the latest image
$ docker pull alpine

# Run the image and install required tools/packages:
$ docker run --cidfile ./gateway.cid alpine /bin/sh -c "apk add --update tor iptables iproute2 && exit"

# Remember the fresh and clean container state:
$ docker commit $(cat ./gateway.cid) dockernymous-xrdp_gateway
```

### The Workstation

**Basic Requirements:**

- Linux (e.g. Kali)
- A desktop environment for remote desktop access (preferred Xfce4)
- Xrdp server (configured to listen on port 3390)

**Example Construction:**

```bash
# Pull Kali Linux as workstation base
$ docker pull kalilinux/kali-linux-docker

# Run the image and install required tools/packages (also, make xrdp listen on port 3390)
$ docker run --cidfile ./workstation.cid kalilinux/kali-linux-docker /bin/bash -c "apt-get update; apt-get upgrade -y; apt-get dist-upgrade -y --force-yes; apt-get --yes --force-yes install kali-desktop-xfce xorg xrdp curl kali-linux-top10; sed -i 's/port=3389/port=3390/g' /etc/xrdp/xrdp.ini; /etc/init.d/xrdp start; exit"

# Remember the fresh and clean container state
$ docker commit $(cat ./workstation.cid) dockernymous-xrdp_workstation
```

### Run Dockernymous

```bash
# Allow script to be executed
$ chmod +x dockernymous.sh

# Run!
$ ./dockernymous.sh
```
