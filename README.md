# Killing this mode of building/housing ARC Singularity Images. 
It seems as though Singularity Hub has some pretty low limits on number of interactions per image such that it isn't going to scale to what we need.  Looking at options, we could a) stand up a Singularity Registry Server etc etc or b) convert to using DockerHub.  I am choosing (b).  

# Singularity RStudio Server

[![Build Status](https://travis-ci.org/rsettlag/singularity-rstudio.svg?branch=master)](https://travis-ci.org/rsettlag/singularity-rstudio)
[![Singularity Hub](https://www.singularity-hub.org/static/img/hosted-singularity--hub-%23e32929.svg)](https://singularity-hub.org/collections/463)
[![GitHub License](https://img.shields.io/badge/license-MIT-green.svg)](https://opensource.org/licenses/MIT)

Singularity image for [RStudio Server] used in [Advanced Research Computing at Virginia Tech](https://www.arc.vt.edu).  This was forked from https://github.com/nickjer/singularity-r with the intent that pacakges/libraries/environment customizations more inline with our user base will be added.

This will always be a work in progress as versions and requirements change.  Note that released versions will be static as I will not look backwards at requirements, but new/altered requirements will be implemented as we version up in R.

Finally, there will be built on all 3 base companion repos, [rsettlag/singularity-r], [rsettlag/singularity-bioR], and [rsettlag/singularity-geoR] all representing layers important to specific user communities at VT.  

## Build

You can build a local Singularity image named `singularity-rstudio.simg` with:

```sh
sudo singularity build singularity-rstudio.simg Singularity
```

## Deploy

Instead of building it yourself you can download the pre-built image from
[Singularity Hub](https://www.singularity-hub.org) with:

```sh
singularity pull --name singularity-rstudio.simg shub://rsettlag/singularity-rstudio
```

## Run

### RStudio Server

The `rserver` command is launched using the default run command:

```sh
singularity run singularity-rstudio.simg
```

or as an explicit app:

```sh
singularity run --app rserver singularity-rstudio.simg
```

Example:

```console
$ singularity run --app rserver singularity-rstudio.simg --help
command-line options:

verify:
  --verify-installation arg (=0)        verify the current installation

server:
  --server-working-dir arg (=/)         program working directory
  --server-user arg (=rstudio-server)   program user
  --server-daemonize arg (=0)           run program as daemon
  --server-app-armor-enabled arg (=1)   is app armor enabled for this session
  --server-set-umask arg (=1)           set the umask to 022 on startup

...
```

#### Simple Password Authentication

To secure the RStudio Server you will need to:

1. Launch the container with the environment variable `RSTUDIO_PASSWORD` set to
   a password of your choosing.
2. Launch the `rserver` command with the PAM helper script `rstudio_auth`.

An example is given as:

```sh
RSTUDIO_PASSWORD="password" singularity run singularity-rstudio.simg \
  --auth-none 0 \
  --auth-pam-helper rstudio_auth
```

Now when you attempt to access the RStudio Server you will be presented with a
log in form. You can log in with your current user name and password you set in
`RSTUDIO_PASSWORD`.

#### LDAP Authentication

Another option is using an LDAP (or Active Directory) server for
authentication. Configuration of the LDAP authentication script `ldap_auth` is
handled through the following environment variables:

- `LDAP_HOST` - the host name of the LDAP server
- `LDAP_USER_DN` - the formatted string (where `%s` is replaced with the
  username supplied during log in) of the bind DN used for LDAP authentication
- `LDAP_CERT_FILE` - the file containing the CA certificates used by
  the LDAP server (default: use system CA certificates)

An example for an LDAP server with signed SSL certificate from a trusted CA:

```sh
export LDAP_HOST=ldap.example.com
export LDAP_USER_DN='cn=%s,dc=example,dc=com'
singularity run singularity-rstudio.simg \
  --auth-none 0 \
  --auth-pam-helper-path ldap_auth
```

An example for an LDAP server with a self-signed SSL certificate:

```sh
export LDAP_HOST=ldap.example.com
export LDAP_USER_DN='cn=%s,dc=example,dc=com'
export LDAP_CERT_FILE=/ca-certs.pem
singularity run \
  --bind /path/to/ca-certs.pem:/ca-certs.pem \
  singularity-rstudio.simg \
    --auth-none 0 \
    --auth-pam-helper-path ldap_auth
```

Note that we had to bind mount the CA certificates file from the host machine
into the container and specify the container's path in `LDAP_CERT_FILE` (not
the host's path).

### R and Rscript

See [rsettlag/singularity-r] for more information on how to run `R` and
`Rscript` from within this Singularity image.

## Contributing

Bug reports and pull requests are welcome on GitHub at
https://github.com/rsettlag/singularity-rstudio.  Or you can pull from the original source at https://github.com/rsettlag/singularity-rstudio

## License

The code is available as open source under the terms of the [MIT License].


[RStudio Server]: https://www.rstudio.com/products/rstudio/
[nickjer/singularity-r]: https://github.com/rsettlag/singularity-r
[MIT License]: http://opensource.org/licenses/MIT
