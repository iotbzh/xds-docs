# Installing XDS server

Depending of your configuration, this step is necessary or not.

In other words **you are a developer and plan to use/connect to an existing `xds-server`**
running on your local network (On-Premise config) or in the Cloud (SaaS config),
**you don't need to install the server part and you can skip this step**.

For others (standalone config or administrators that want to install an
On-Premise solution) xds-server must be installed.

Several installation types are supported :

| Install type | Supported OS | Section to refer |
|--------------|--------------|------------------|
| Container | Linux or MacOS | [see Installation based on Docker container](#installation-based-on-docker-container) |
| Virtual Machine | Linux, MacOS or Windows | [see Installation based on VirtualBox appliance](#installation-based-on-virtual-machine-appliance) |
| Native | Linux | [see Native installation](#native-installation) |

## Installation based on Docker container

### Prerequisites

Docker is installed on the host machine, please refer to [Docker documentation](https://docs.docker.com/engine/installation/) for more details.

### Get the container

Load the pre-build AGL SDK docker image including `xds-server`:

```bash
wget -O - http://iot.bzh/download/public/XDS/docker/docker_agl_worker-xds-latest.tar.xz | docker load
```

You should get `docker.automotivelinux.org/agl/worker-xds:X.Y` image

```bash
# List image that we just load
docker images "docker.automotivelinux.org/agl/worker-xds*"

docker.automotivelinux.org/agl/worker-xds       4.0              786d65b2792c        6 days ago          654MB
```

### Create and start a new container

Use provided script to create a new docker container and start it:

```bash
# Get script
wget https://raw.githubusercontent.com/iotbzh/xds-server/master/scripts/xds-docker-create-container.sh

# Create new XDS worker container
bash ./xds-docker-create-container.sh

# Check that new container is running
docker ps | grep worker-xds

b985d81af40c        docker.automotivelinux.org/agl/worker-xds:3.99.1       "/usr/bin/wait_for..."   6 days ago           Up 4 hours          0.0.0.0:8000->8000/tcp, 0.0.0.0:69->69/udp, 0.0.0.0:10809->10809/tcp, 0.0.0.0:2222->22/tcp    agl-xds-seb@laptop-0-seb
```

Note that you can also add a new shared directory using `--volume` option in order
to use for example with Path-Mapping folder type.

```bash
# Create new XDS worker container and share extra '$HOME/my-workspace' directory
bash ./xds-docker-create-container.sh --volume /my-workspace:$HOME/my-workspace
```

### Check if xds-server is running

`xds-server` is automatically started as a service on container startup.

To check if xds-server is correctly install and running, you can access the basic
web page that gives you some instructions:

```bash
# if container is running on your local host
# (else replace localhost by the name or the ip of the machine running the container)
xdg-open http://localhost:8000
```

`xds-server` is now up and running, you can now install AGL SDKs, please refer
to next chapter named [Installing AGL SDKs](3_install-sdks.md#installing-agl-sdks)

### Container settings

This container (ID=0) exposes following ports:

- 8000 : `xds-server` to serve XDS basic web page
- 69   : TFTP
- 2222 : ssh

This container also creates the following volumes (sharing directories between
inside and outside docker):

| Directory on host | Directory inside docker | Comment |
|-------------------|-------------------------|---------|
| $HOME/xds-workspace | /home/devel/xds-workspace | XDS projects workspace location|
| $HOME/xds-workspace/.xdt_0 | /xdt | location to store SDKs |
| $USER_VOLUME | $USER_VOLUME | user path, see `--volume` option of `xds-docker-create-container.sh` script |

<!-- note -->
Please refer to [part 2 - xds-server](../part-2/1_xds-server.md#sdk-cross-toolchain-management) documentation
for additional info.
<!-- endnote -->

## Installation based on Virtual Machine appliance

### Prerequisites

> VirtualBox is installed on the host machine

please refer to [VirtualBox documentation](https://www.virtualbox.org/wiki/Downloads) for more details.

### Get the appliance

Load the pre-build AGL SDK appliance image including `xds-server`:

```bash
wget http://iot.bzh/download/public/XDS/appliance/xds-vm-debian9_latest.ova
```

### Clean old appliance

You must have one and one xds appliance only.

So, first remove the oldest xds appliance if needed.

```bash
# Get the virtual machine name
VDS_VMNAME=$(VBoxManage list vms | grep xds-vm-debian | cut -d "\"" -f2)
echo ${VDS_VMNAME}

# Remove old XDS appliance
[ -n ${VDS_VMNAME} ] && VBoxManage unregistervm ${VDS_VMNAME} --delete
```

### Create and start a new appliance

Used provided script to create a new appliance or you can use VirtualBox GUI:

```bash
# Import image into VirtualBox
VBoxManage import ./xds-vm-debian9_latest.ova

# Check import result
VDS_VMNAME=$(VBoxManage list vms | grep xds-vm-debian | cut -d "\"" -f2)
echo ${VDS_VMNAME}

# Start XDS appliance
[ -n ${VDS_VMNAME} ] && VBoxManage startvm ${VDS_VMNAME}
```

### Appliance settings

This image exposes following network ports (NAT mode):

- 8000 : `xds-server` to serve XDS basic web page
- 69   : TFTP
- 2222 : ssh

## Check if xds-server is running

`xds-server` is automatically started as a service on container startup.

To check if xds-server is correctly install and running, you can access the basic web page that gives you some instructions:

```bash
# if container/appliance is running on your local host
# (else replace localhost by the name or the ip of the machine running the container)
xdg-open http://localhost:8000
```

`xds-server` is now up and running, you can now install AGL SDKs, please refer
to next chapter named [Installing AGL SDKs](3_install-sdks.md#installing-agl-sdks)

## Native installation

You can chose to install xds-server 'natively' instead of within a docker
container but note that only Linux host OSes are supported and tested for native
installation !

### Install packages for debian distro type

```bash
# 'DISTRO' can be set to { xUbuntu_16.04, xUbuntu_16.10, xUbuntu_17.04, Debian_8.0, Debian_9.0}
export DISTRO="xUbuntu_16.04"

wget -O - http://download.opensuse.org/repositories/isv:/LinuxAutomotive:/app-Development/${DISTRO}/Release.key | sudo apt-key add -
sudo bash -c "cat >> /etc/apt/sources.list.d/AGL.list <<EOF
deb http://download.opensuse.org/repositories/isv:/LinuxAutomotive:/app-Development/${DISTRO}/ ./
EOF"

sudo apt-get update
sudo apt-get install agl-xds-server
```

### Install packages for openSUSE distro type

```bash
# DISTRO can be set to {openSUSE_Leap_42.2, openSUSE_Leap_42.3, openSUSE_Tumbleweed}
export DISTRO="openSUSE_Leap_42.2"

sudo zypper ar http://download.opensuse.org/repositories/isv:/LinuxAutomotive:/app-Development/${DISTRO}/isv:LinuxAutomotive:app-Development.repo

sudo zypper ref
sudo zypper install agl-xds-server
```

### Configure xds-server

<!-- note -->
**Optional step**: skip this chapter if you plan to use default settings
<!-- endnote -->

When `xds-server` is started as a systemd service, default environment variables
are set into `/etc/default/xds-server` file.

`xds-server` configuration is also driven by a JSON config file (`server-config.json`),
and default JSON config is `/etc/xds/server/server-config.json`.

<!-- note -->
**Note:** you can use your own JSON config by settings `APP_CONFIG` variable of
`/etc/default/xds-server` file to your file, for example `/home/MYUSER/.xds/server/server-config.json`
<!-- endnote -->

Supported fields in JSON configuration file are :

- **httpPort** : HTTP port of client webapp/REST API
- **webAppDir** : location of client web application (default: webapp/dist)
- **shareRootDir** : root directory where projects will be copied
- **logsDir**  : directory to store logs (eg. syncthing output)
- **sdkScriptsDir** : directory where scripts, used to managed SDKs, are installed
- **syncthing.binDir** : syncthing binaries directory (default: executable directory)
- **syncthing.home"** : syncthing home directory (usually .../syncthing-config)
- **syncthing.gui-address** : syncthing gui url (default <http://localhost:8385>)
- **syncthing.gui-apikey** : syncthing api-key to use (default auto-generated)

All fields are optional and example below corresponds to the default values:

```json
{
    "httpPort": 8000,
    "webAppDir": "webapp/dist",
    "shareRootDir": "${HOME}/.xds/server/projects",
    "logsDir": "/tmp/logs",
    "sdkScriptsDir": "${EXEPATH}/scripts/sdks",
    "syncthing": {
        "binDir": "./bin",
        "home": "${HOME}/.xds/server/syncthing-config",
        "gui-address": "http://localhost:8385",
        "gui-apikey": "123456789",
    }
}
```

>**Note:** environment variables are supported by using `${MY_VAR}` syntax.

### Start/Stop xds-server

`xds-server` can be managed as a systemd service with the following commands:

```bash
# Status XDS server:
systemctl --user status xds-server.service

# Stop XDS server
systemctl --user stop xds-server.service

# Start XDS server
systemctl --user start xds-server.service

# Get XDS server logs
systemctl --user --unit=xds-server.service --output=cat
```

To check if xds-server is correctly install and running, you can access the web
interface, using a web browser :

```bash
xdg-open http://localhost:8000
```

or get the current version using the following curl command:

```bash
curl http://localhost:8000/api/v1/version
```
