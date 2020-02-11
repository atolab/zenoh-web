---
title: "Installation"
weight : 1010
menu:
  docs:
    parent: getting_started
---

To get up and running with <b>zenoh</b> you will have to install the infrastructure and then get hold of the API you would like to use to write your applications. 

# Installing zenoh's Infrastructure
At the present stage zenoh's infrastructure is supported only on Linux and MacOS. Below are the detailed information on how to install on supported platforms.

## MacOS
The first step is to tap our brew package repository:

```bash
$ brew tap atolab/homebrew-atobin
```    

Then simply install zenoh as follows:

```bash
$ brew install zenoh
```

## Linux
The Linux installation procedure depends on the package manager supported by your distribution. Below are detailed information for <b>apt</b> and <b>yum</b> based distros.

### Ubuntu (16.04 and 18.04)
As a first step add our package repository to your package manager configuration by running one of the following command:

On Ubuntu 16.04:
```bash
$ echo "deb [trusted=yes] http://pkgs.adlink-labs.tech/debian/16.04 ./" | sudo tee -a /etc/apt/sources.list > /dev/null
```

On Ubuntu 18.04:
```bash
$ echo "deb [trusted=yes] http://pkgs.adlink-labs.tech/debian/18.04 ./" | sudo tee -a /etc/apt/sources.list > /dev/null
```

Then update the packages list by:

```bash
$ sudo apt update
```

Now you can install zenoh:

```bash
$ sudo apt install zenoh
```

### CentOS 7
The first step is to add the ATOLab repository to <b>yum</b>, this can be done by editing the file:

```bash
$ sudo vi /etc/yum.repos.d/atolab.repo
```

and writing the following content:

```toml,ignore
[atolab-repo]
name=Atolab RPM Package Repo
baseurl=http://pkgs.adlink-labs.tech/centos/7     # server-name or repo-server-ip
enabled=1
gpgcheck=0
```

At this point update the packages and install as follows:

```bash
$ yum repolist
$ yum update
$ yum install zenoh
```

# Testing Your Installation
To test the installation, try to see the zenoh man page by executing the following command:

```bash
$ zenohd --help
```
You should see the following output on your console:

```text
zenohd(1)                        Zenohd Manual                       zenohd(1)


NAME
       zenohd

SYNOPSIS
       zenohd [OPTION]...

OPTIONS
       --<plugin_name>.<plugin_option>=<option_value>
           Pass to the plugin with name '<plugin_name>' the option
           --<plugin_option>=<option_value>. Example of usage:
           --zenoh-storages.storage=/demo/example/**
           --zenoh-http.httpport=8080

       --color=WHEN (absent=auto)
           Colorize the output. WHEN must be one of `auto', `always' or
           `never'.

       --help[=FMT] (default=auto)
[...]           
```
# Pick Your Programming Language
When you install the zenoh infrastructure you will get installed the developer SDK for  [C](https://en.wikipedia.org/wiki/The_C_Programming_Language). Yet, zenoh already supports quite a few programming languages, below is the list of supported programming languages as well as links to the installation instruction:

- [Python SDK](https://github.com/atolab/zenoh-python)
- [Java SDK](https://github.com/atolab/zenoh-java)
- [Go SDK](https://github.com/atolab/zenoh-go)
- [OCaml SDK](https://github.com/atolab/zenoh)

