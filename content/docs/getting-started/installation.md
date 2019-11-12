---
title: "Installation"
weight : 1010
menu:
  docs:
    parent: getting_started
---

To get up and running with <b>zenoh</b> you will have to install the infrastructure and then get hold of the API you would like to use to write your applications. 

## Installing zenoh's Infrastructure
At the present stage zenoh's infrastructure is supported only on Linux and MacOS. Below are the detailed information on how to install on supported platforms.

### MacOS
The first step is to tap our brew package repository:

```bash
    $ brew tap atolab/homebrew-atobin
```    

Then simply install zenoh as follows:

```bash
    $ brew install zenohd
```

### Linux
The Linux installation procedure depends on the package manager supported by your distribution. Below are detailed information for <b>apt</b> and <b>yum</b> based distros.

#### Debian and its derivatives
As a first step add our package repository to your package manager configuration by running the following command:

```bash
echo "deb [trusted=yes] http://pkgs.adlink-labs.tech/debian ./" | sudo tee -a /etc/apt/sources.list > /dev/null
```

Then update the packages list by:

```bash
sudo apt update
```

Now you can install zenoh:

```bash
sudo apt install yaks
```