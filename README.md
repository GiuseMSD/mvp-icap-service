# Glasswall ICAP Service - Minimum Viable Prototype
ICAP Service with ICAP Resource that interfaces with GW Cloud products

## Getting started
The original baseline code has been cloned from the open source project
https://sourceforge.net/projects/c-icap/

Demonstration ICAP Resources have been removed.

## Building ICAP Service

These instructions guide the user through the steps involved in installing the Glasswall ICAP PoC on a Linux host.

Running the follow commands will ensure the necessary packages are installed.
```
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install git
sudo apt-get install gcc
sudo apt-get install -y doxygen
sudo apt-get install make
sudo apt-get install automake
sudo apt-get install automake1.11
```

### Build the Server
From where the repo was cloned to, navigate into the `c-icap/c-icap` folder and run the scripts to setup the Makefiles.
```
aclocal
autoconf
automake --add-missing
```
Run the configure script, specifying where the server should be installed, through the `prefix` argument.
```
./configure --prefix=/usr/local/c-icap
```
After running the configuration script, build and install the server.
```
make 
sudo make install
```
The option is available to generate the documentation, if required
```
make doc
```

### Build the Modules

Navigate to the modules folder (`c-icap/c-icap-modules`) and run the scripts to setup the Makefiles.
```
aclocal
autoconf
automake --add-missing
```
Run the configure script, specifing where the server was installed, in both the `with-c-icap` and `prefix` arguments.
```
./configure --with-c-icap=/usr/local/c-icap --prefix=/usr/local/c-icap
```
After running the configuration script, we can build and install
```
make 
sudo make install
```
> During the `make install` there will be some warnings about `libtools`, these can be ignored.

After installation, the configuration files for each module/service are available in the c-icap server configuration directory, `/usr/local/c-icap/etc/` using the location folder specified in the 'configure' commands above.  

For a module/service to be recognisd by the C-ICAP server its configuration file needs to be included into the main c-icap server configuration file. The following command adds the `gw_rebuild.conf` file
```
sudo sh -c 'echo "Include gw_rebuild.conf" >>  /usr/local/c-icap/etc/c-icap.conf'
```

## Testing the Installation

On the host server run the ICAP Server with the following command
```
sudo /usr/local/c-icap/bin/c-icap -N -D -d 10
```

From a separate command prompt, run the client utility to send an options request. The module specified in the `-s` argument must have been `Included` into the `gw_test.conf` file in the step above.
```
/usr/local/c-icap/bin/c-icap-client -s gw_rebuild
```

Run the client utility sending a file through the ICAP Server. This requires sufficient configuration to have been provided for the
```
/usr/local/c-icap/bin/c-icap-client -f <full path to source file>  -o <full path to output file> -s gw_test
/usr/local/c-icap/bin/c-icap-client -f <full path to source file>  -o <full path to output file> -s gw_rebuild
```

A full list of the command line options available to the client utility are available from the application's `help` option.
```
/usr/local/c-icap/bin/c-icap-client  --help
```

## Retrieving Statistics

The ICAP Server provides an `info` resource that can be used to query for processing statistics. The `c-icap-client` can be used to query for this information. The address of the ICAP Server host can be substituted for `localhost` in this example, if necessary.
```
sudo /usr/local/c-icap/bin/c-icap-client -s "info?view=text" -i localhost -req any

```
If `view=text` is not included, then the response is formatted as an HTML table.
The `-req` attribute is required in order to avoid an `options` request from being sent (the default).

Excerpt of typical statistics request
```
Service gw_rebuild Statistics
==================
Service gw_rebuild REQMODS : 0
Service gw_rebuild RESPMODS : 0
Service gw_rebuild OPTIONS : 1
Service gw_rebuild ALLOW 204 : 0
Service gw_rebuild REQUESTS SCANNED : 0
Service gw_rebuild REBUILD FAILURES : 0
Service gw_rebuild REBUILD ERRORS : 0
Service gw_rebuild SCAN REBUILT : 0
Service gw_rebuild UNPROCESSED : 0
Service gw_rebuild UNPROCESSABLE : 0
Service gw_rebuild BYTES IN : 0 Kbs 133 bytes
Service gw_rebuild BYTES OUT : 0 Kbs 255 bytes
Service gw_rebuild HTTP BYTES IN : 0 Kbs 0 bytes
Service gw_rebuild HTTP BYTES OUT : 0 Kbs 0 bytes
Service gw_rebuild BODY BYTES IN : 0 Kbs 0 bytes
Service gw_rebuild BODY BYTES OUT : 0 Kbs 0 bytes
Service gw_rebuild BODY BYTES SCANNED : 0 Kbs 0 bytes

```


