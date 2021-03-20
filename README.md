# Pi-Hole + Unbound

This project is based on the [original repository by Chris Crowe](https://github.com/chriscrowe/docker-pihole-unbound).
The adjusments I made follow:

1. using `host` networking mode in docker, this is **crucial** if you are using the Pi-hole as a **DHCP server**, see the [official Pi-hole documentation](https://docs.pi-hole.net/docker/DHCP/) for further details
2. `Docker Hub` is no longer used, the docker image is built locally from the `Dockerfile`
3. `pihole/pihole:latest` is used as the original image, making updating easy
4. added `update-pihole.sh` script - just run it and both `pihole` and `unbound` should update (if there is an update to begin with)
5. Bugfix 1: added `cap_add: NET_ADMIN` to the `docker-compose`, required if you want to use the Pi-hole as a DHCP server
6. Bugfix 2: using the CLI `apt-get` instead of `apt` which prevents a warning (and potentially something worse later)

## Description

This Docker deployment runs both Pi-Hole and Unbound in a single container.

The base image for the container is the [official Pi-Hole container](https://hub.docker.com/r/pihole/pihole), with an extra build step added to install the Unbound resolver directly into to the container based on [instructions provided directly by the Pi-Hole team](https://docs.pi-hole.net/guides/unbound/).

## Usage

First create a `.env` file to substitute variables for your deployment.

### Required environment variables

> Vars and descriptions replicated from the [official pihole container](https://github.com/pi-hole/docker-pi-hole/):

| Docker Environment Var | Description|
| --- | --- |
| `ServerIP: <Host's IP>`<br/> | **--net=host mode requires** Set to your server's LAN IP, used by web block modes and lighttpd bind address
| `TZ: <Timezone>`<br/> | Set your [timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) to make sure logs rotate at local midnight instead of at UTC midnight.
| `WEBPASSWORD: <Admin password>`<br/> | http://pi.hole/admin password. Run `docker logs pihole \| grep random` to find your random pass.
| `REV_SERVER: <"true"\|"false">`<br/> | Enable DNS conditional forwarding for device name resolution
| `REV_SERVER_DOMAIN: <Network Domain>`<br/> | If conditional forwarding is enabled, set the domain of the local network router
| `REV_SERVER_TARGET: <Router's IP>`<br/> | If conditional forwarding is enabled, set the IP of the local network router
| `REV_SERVER_CIDR: <Reverse DNS>`<br/>| If conditional forwarding is enabled, set the reverse DNS zone (e.g. `192.168.0.0/24`)
| `HOSTNAME=pihole`<br />| hostname
| `DOMAIN_NAME=pihole.local`<br />| domain name

Example `.env` file in the same directory as your `docker-compose.yaml` file:

```
ServerIP=192.168.0.128
TZ=Europe/Prague
REV_SERVER=true
REV_SERVER_DOMAIN=local
REV_SERVER_TARGET=192.168.0.1
REV_SERVER_CIDR=192.168.0.0/16
HOSTNAME=pihole
DOMAIN_NAME=pihole.local
```

### Running

```bash
docker-compose up -d
```