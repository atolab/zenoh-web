---
title: "Installation"
weight : 1010
menu:
  docs:
    parent: getting_started
---

To get up and running with <b>zenoh</b> you will have to install the router and then get hold of the API you would like to use to write your applications. 

## Installing zenoh's router
Zenoh's router supports Linux, MacOS and Windows platforms.
For other platforms, you can use the [Docker image](../quick-test#run-zenoh-in-docker).

Below are the detailed information on how to install the binaries directly on supported platforms (i.e. without Docker).

### MacOS
Tap our brew package repository:

```bash
$ brew tap eclipse-zenoh/homebrew-zenoh
```    

Install zenoh:

```bash
$ brew install zenoh
```

Then you can start the zenoh router with this command:
```bash
$ zenohd -V
```

### Ubuntu (20.04)

Add Eclipse Zenoh private repository to the sources list:

```bash
$ echo "deb [trusted=yes] https://download.eclipse.org/zenoh/zenoh/master/ /" | sudo tee -a /etc/apt/sources.list > /dev/null
$ sudo apt update
```

Install zenoh:

```bash
$ sudo apt install zenoh 
```
 
Then you can start the zenoh router with this command:

```bash
$ zenohd -V
```

### Windows

Download the zenoh archive :
- Windows 32 bits: https://download.eclipse.org/zenoh/zenoh/master/eclipse-zenoh-0.5.0-beta.5-i686-pc-windows-gnu.zip
- Windows 64 bits: https://download.eclipse.org/zenoh/zenoh/master/eclipse-zenoh-0.5.0-beta.5-x86_64-pc-windows-gnu.zip

Unzip the archive.

Go to zenoh directory and start zenoh router:

```cmd
> cd C:\path\to\zenoh\dir
> zenohd.exe -V
```

## Testing Your Installation
To test the installation, try to see the zenoh man page by executing the following command:

```bash
$ zenohd --help
```
You should see the following output on your console:

```text
The zenoh router 

USAGE:
    zenohd [FLAGS] [OPTIONS]

FLAGS:
    -h, --help               Prints help information
        --no-backend         If true, no backend (and thus no storage) are created at startup. If false (default) the
                             Memory backend it present at startup.
        --no-timestamp       By default zenohd adds a HLC-generated Timestamp to each routed Data if there isn't already
                             one. This option desactivates this feature.
        --plugin-nolookup    When set, zenohd will not look for plugins nor try to load any plugin except the ones
                             explicitely configured with -P or --plugin.
    -V, --version            Prints version information

OPTIONS:
        --http-port <http-port>         The listening http port [default: 8000]
    -i, --id <hex_string>...            The identifier (as an hexadecimal string - e.g.: 0A0B23...) that zenohd must
                                        use. WARNING: this identifier must be unique in the system! If not set, a random
                                        UUIDv4 will be used.
    -l, --listener <LOCATOR>...         A locator on which this router will listen for incoming sessions. Repeat this
                                        option to open several listeners. [default: tcp/0.0.0.0:7447]
        --mem-storage <PATH_EXPR>...    A memory storage to be created at start-up. Repeat this option to created
                                        several storages
    -e, --peer <LOCATOR>...             A peer locator this router will try to connect to. Repeat this option to connect
                                        to several peers.
    -P, --plugin <PATH_TO_PLUGIN>...    A plugin that must be loaded. Repeat this option to load several plugins.
[...]
```

## Installing client library
To develop your application zenoh application, you need to install a zenoh client library.
Depending your programmation language, choose one of the following API and refer to the installation and usage instructions in here:

- [Python API](https://github.com/eclipse-zenoh/zenoh-python)
- [C API (only zenoh-net API)](https://github.com/eclipse-zenoh/zenoh-c)
