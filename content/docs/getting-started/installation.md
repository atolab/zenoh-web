---
title: "Installation"
weight : 1010
menu:
  docs:
    parent: getting_started
---

To get up and running with <b>zenoh</b> you will have to install the router and then get hold of the API you would like to use to write your applications. 

## Installing the zenoh router
The zenoh router (a.k.a. `zenohd`) and its plugins are currently available as a pre-built binaries for various platforms. All release packages can be downloaded from:  
  -  **https://download.eclipse.org/zenoh/zenoh/latest/**

Each sub-directory has the name of the Rust target. See the platforms each target corresponds to on https://doc.rust-lang.org/stable/rustc/platform-support.html

You can also install it via a package manager on MacOS (homebrew) or Linux Debian (apt). See instructions below.

For other platforms, you can use the [Docker image](../quick-test#run-zenoh-in-docker) or [build it](https://github.com/eclipse-zenoh/zenoh#how-to-build-it) directly on your platform.

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

### Ubuntu or any Debian

Add Eclipse Zenoh private repository to the sources list:

```bash
$ echo "deb [trusted=yes] https://download.eclipse.org/zenoh/debian-repo/ /" | sudo tee -a /etc/apt/sources.list > /dev/null
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

Download the zenoh archive from https://download.eclipse.org/zenoh/zenoh/latest/ :
- For Windows 64 bits: get the `x86_64-pc-windows-msvc/zenoh-<version>-x86_64-pc-windows-msvc.zip`file

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
The zenoh router v0.5.0-beta.X

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
        --backend-search-dir <DIRECTORY>...    A directory where to search for backends libraries to load. Repeat this
                                               option to specify several search directories'. By default, the backends
                                               libraries will be searched in: '/usr/local/lib:/usr/lib:~/.zenoh/lib:.'
    -c, --config <FILE>                        The configuration file.
    -i, --id <hex_string>                      The identifier (as an hexadecimal string - e.g.: 0A0B23...) that zenohd
                                               must use. WARNING: this identifier must be unique in the system! If not
                                               set, a random UUIDv4 will be used.
    -l, --listener <LOCATOR>...                A locator on which this router will listen for incoming sessions. Repeat
                                               this option to open several listeners. [default: tcp/0.0.0.0:7447]
        --mem-storage <PATH_EXPR>...           A memory storage to be created at start-up. Repeat this option to created
                                               several storages
    -e, --peer <LOCATOR>...                    A peer locator this router will try to connect to. Repeat this option to
                                               connect to several peers.
    -P, --plugin <PATH_TO_PLUGIN_LIB>...       A plugin that must be loaded. Repeat this option to load several plugins.
        --plugin-search-dir <DIRECTORY>...     A directory where to search for plugins libraries to load. Repeat this
                                               option to specify several search directories'. By default, the plugins
                                               libraries will be searched in: '/usr/local/lib:/usr/lib:~/.zenoh/lib:.'
        --rest-http-port <rest-http-port>      The REST plugin's http port [default: 8000]
[...]
```

## Installing client library
To develop your application zenoh application, you need to install a zenoh client library.
Depending your programmation language, choose one of the following API and refer to the installation and usage instructions in here:

- [Rust API](https://crates.io/crates/zenoh)
- [Python API](https://github.com/eclipse-zenoh/zenoh-python)
- [C API](https://github.com/eclipse-zenoh/zenoh-c) (only zenoh-net API)
- [Pico API ](https://github.com/eclipse-zenoh/zenoh-pico): C API for constrained devices (only zenoh-net API, no peer-to-peer mode)
