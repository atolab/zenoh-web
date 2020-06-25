---
title: "Installation"
weight : 1010
menu:
  docs:
    parent: getting_started
---

To get up and running with <b>zenoh</b> you will have to install the router and then get hold of the API you would like to use to write your applications. 

## Installing zenoh's router
At the present stage zenoh's router is supported only on Linux and MacOS. 

However, for quick tests, a Docker image is available:  
https://hub.docker.com/r/eclipse/zenoh  
Install and run it with those commands:  
```bash
$ docker pull eclipse/zenoh:latest
$ docker run --init -p 7447:7447/tcp -p 7447:7447/udp -p 8000:8000/tcp eclipse/zenoh:latest
```

Below are the detailed information on how to install the binaries directly on supported platforms (i.e. without Docker).

### MacOS
The first step is to tap our brew package repository:

```bash
$ brew tap atolab/homebrew-atobin
```    

And simply install zenoh as follows:

```bash
$ brew install zenoh
```

Then you can start the zenoh router with this command:
```bash
$ zenohd -v
```

### Ubuntu (20.04)
Download the pre-built binaries:  
https://download.eclipse.org/zenoh/zenoh/0.4.2-M1/eclipse-zenoh-0.4.2-M1-Ubuntu-20.04-x64.tgz

Extract it and start the zenoh router with this command:
```bash
$ eclipse-zenoh/bin/zenohd.exe -v
```

The Linux installation procedure depends on the package manager supported by your distribution. Below are detailed information for <b>apt</b> and <b>yum</b> based distros.


## Testing Your Installation
To test the installation, try to see the zenoh man page by executing the following command:

```bash
$ zenohd --help
```
You should see the following output on your console:

```text
ZENOHD(1)                        Zenohd Manual                       ZENOHD(1)

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

       -d <discovery>, --discovery=<discovery> (absent=auto)
           The ip-address of the interface over which scouting should be ran.

       --help[=FMT] (default=auto)
[...]           
```

## Pick Your Programming Language
When you install the zenoh router you will get installed the developer SDK for  [C](https://en.wikipedia.org/wiki/The_C_Programming_Language). Yet, zenoh already supports quite a few programming languages, below is the list of supported programming languages as well as links to the installation instruction:

- [Python SDK](https://github.com/eclipse-zenoh/zenoh-python)
- [Java SDK](https://github.com/eclipse-zenoh/zenoh-java)
- [Go SDK](https://github.com/eclipse-zenoh/zenoh-go)

