# Setup

Minimum machine configuration required

| Key              | Value        |
| ---------------- | ------------ |
| CPU cores        | 2GHz, 1      |
| RAM              | 1 Gigabyte   |
| Disk space       | 10 Gigabytes |
| Disk type        | HDD          |
| Operating System | Ubuntu 20.04 |

## Install Docker

:light-bulb: To install Docker engine you can refer to its [official documentation](https://docs.docker.com/engine/install/). You can find various ways to install it and specific instructions according to your OS / distrib. For instance, if you want to [install Docker engine on Debian](https://docs.docker.com/engine/install/debian/) or Ubuntu, you can setup official docker's repositories to get it from your package manager.

1. Update the list of available software packages

    ``` sh
    sudo apt-get update
    ```

2. Install cURL package

    ``` sh
    sudo apt-get install --yes curl
    ```

3. Get the official Docker installation script

    ``` sh
    curl -fsSL get.docker.com -o ${HOME}/get-docker.sh
    ```

4. Install Docker

    ``` sh
    sudo sh ${HOME}/get-docker.sh
    ```

5. Add user to Docker group

    ``` sh
    sudo usermod -aG docker $(whoami)
    ```

6. Reboot the machine

## Enable IPv6 support for Docker (optional)

1. Open the file `/etc/docker/daemon.json` with a text editor

2. Paste the following configuration

    ``` text
    {
        "ipv6": true,
        "fixed-cidr-v6": "2001:db8:1::/64"
    }
    ```

3. Save the file

4. Restart the Docker process

    ``` sh
    sudo systemctl restart docker
    ```

5. Install `iptables-persistent` package

    ``` sh
    sudo apt-get install --yes iptables-persistent
    ```

6. Enable NAT for the private Docker subnet on the host

    ``` sh
    rule="POSTROUTING -s 2001:db8:1::/64 ! -o docker0 -j MASQUERADE" && \
    sudo ip6tables -t nat -C ${rule} || \
    sudo ip6tables -t nat -A ${rule} && \
    sudo sh -c "ip6tables-save > /etc/iptables/rules.v6"
    ```

## Preparing the Docker image

### Prebuilt

1. Pull the image

    ``` sh
    docker pull ghcr.io/sentinel-official/dvpn-node:latest
    ```

2. Tag the image

    ``` sh
    docker tag ghcr.io/sentinel-official/dvpn-node:latest sentinel-dvpn-node
    ```

### From source

1. Install Git package

    ``` sh
    sudo apt-get install --yes git
    ```

2. Clone the GitHub repository

    ``` sh
    git clone https://github.com/sentinel-official/dvpn-node.git \
        ${HOME}/dvpn-node/
    ```

3. Checkout to the latest tag

    ``` sh
    cd ${HOME}/dvpn-node/ && \
    commit=$(git rev-list --tags --max-count=1) && \
    git checkout $(git describe --tags ${commit})
    ```

4. Build the image

    ``` sh
    docker build --file Dockerfile \
        --tag sentinel-dvpn-node \
        --force-rm \
        --no-cache \
        --compress .
    ```

## Create a self-signed TLS certificate

1. Install `openssl` package

    ``` sh
    sudo apt-get install --yes openssl
    ```

2. Create a certificate

    ``` sh
    openssl req -new \
        -newkey ec \
        -pkeyopt ec_paramgen_curve:prime256v1 \
        -x509 \
        -sha256 \
        -days 365 \
        -nodes \
        -out ${HOME}/tls.crt \
        -keyout ${HOME}/tls.key
    ```
    
## Install Wireguard (Debian)

Error while running docker image 

```
[#] ip link add wg0 type wireguard
RTNETLINK answers: Not supported
Unable to access interface: Protocol not supported
[#] ip link delete dev wg0
Cannot find device "wg0"
Error: exit status 1
```

In order to run the docker image propperly on debian you have to install wireguard.  for Debian 10.

``` sh
echo 'deb http://ftp.debian.org/debian buster-backports main' | sudo tee /etc/apt/sources.list.d/buster-backports.list
sudo apt update
sudo apt install wireguard
```

If the error persist or if you have the error msg "RTNETLINK answers: Not supported" you can install linux headers to make wireguard work

``` sh
apt-get install wireguard-dkms wireguard-tools linux-headers-$(uname -r)
```
