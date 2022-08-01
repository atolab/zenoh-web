---
title: "Installation"
weight : 1010
menu:
  docs:
    parent: getting_started
---

To get up and running with <b>zenoh</b> you will have to install the router and then get hold of the API you would like to use to write your applications. 

## Installing the zenoh router
The zenoh router (a.k.a. `zenohd`) and its plugins are currently available as pre-built binaries for various platforms. All release packages can be downloaded from:  
  -  **https://download.eclipse.org/zenoh/zenoh/latest/**

Each subdirectory has the name of the Rust target. See the platforms each target corresponds to on https://doc.rust-lang.org/stable/rustc/platform-support.html

You can also install it via a package manager on macOS (homebrew) or Linux Debian (apt). See instructions below.

For other platforms, you can use the [Docker image](./quick-test#run-zenoh-in-docker) or [build it](https://github.com/eclipse-zenoh/zenoh#how-to-build-it) directly on your platform.

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
The zenoh router v0.6.0-beta.X

USAGE:
    zenohd [OPTIONS]

OPTIONS:
    -c, --config [<FILE>...]
            The configuration file. Currently, this file must be a valid JSON5 or YAML file.

        --cfg <KEY:VALUE>
            Allows arbitrary configuration changes as column-separated KEY:VALUE pairs, where:
              - KEY must be a valid config path.
              - VALUE must be a valid JSON5 string that can be deserialized to the expected type for
            the KEY field.
            Examples:
            --cfg='startup/subscribe:["demo/**"]'
            --cfg='plugins/storage_manager/storages/demo:{key_expr:"demo/example/**",volume:"memory"}'

    -e, --connect [<ENDPOINT>...]
            A peer locator this router will try to connect to.
            Repeat this option to connect to several peers.

    -h, --help
            Print help information

    -i, --id [<HEX_STRING>]
            The identifier (as an hexadecimal string, with odd number of chars - e.g.: 0A0B23...)
            that zenohd must use. If not set, a random UUIDv4 will be used.
            WARNING: this identifier must be unique in the system and must be 16 bytes maximum (32
            chars)!

    -l, --listen [<ENDPOINT>...]
            A locator on which this router will listen for incoming sessions.
            Repeat this option to open several listeners.

        --no-multicast-scouting
            By default zenohd replies to multicast scouting messages for being discovered by peers
            and clients. This option disables this feature.

        --no-timestamp
            By default zenohd adds a HLC-generated Timestamp to each routed Data if there isn't
            already one. This option disables this feature.

    -P, --plugin [<PLUGIN>...]
            A plugin that MUST be loaded. You can give just the name of the plugin, zenohd will
            search for a library named 'libzplugin_<name>.so' (exact name depending the OS). Or you
            can give such a string: "<plugin_name>:<library_path>".
            Repeat this option to load several plugins. If loading failed, zenohd will exit.

        --plugin-search-dir [<DIRECTORY>...]
            A directory where to search for plugins libraries to load.
            Repeat this option to specify several search directories.

        --rest-http-port [<SOCKET>]
            Configures HTTP interface for the REST API (enabled by default). Accepted values:
              - a port number
              - a string with format `<local_ip>:<port_number>` (to bind the HTTP server to a
            specific interface)
              - `none` to disable the REST API
             [default: 8000]

    -V, --version
            Print version information
```

## Installing client library
To develop your application zenoh application, you need to install a zenoh client library.
Depending on your programmation language of choice, pick one of the following API and refer to the installation and usage instructions in here:

- [Rust API](https://crates.io/crates/zenoh)
- [Python API](https://github.com/eclipse-zenoh/zenoh-python)
- [C API](https://github.com/eclipse-zenoh/zenoh-c)
- [Pico API](https://github.com/eclipse-zenoh/zenoh-pico): A port of Zenoh in C, targeted at low-power devices.

Note that if you wish to always have access to all of Zenoh's latest features, Rust is Zenoh's original language, and will therefore always be the most feature-complete version.