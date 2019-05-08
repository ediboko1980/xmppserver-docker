Kontalk server for Docker
=========================

Here you may find a few scripts and files to build a Kontalk container
(actually 3 containers) for running a Kontalk server.

## Requirements

* Recent 64-bit Linux distro (we suggest Debian)
* Docker
* Docker Compose
* git

## Setting up environment

Setup a new user account (or use an existing one, root is not needed and not recommended)
on a system with Docker installed and enabled. The user account should be part of the docker
group so it will be able to use Docker.

Clone this repository in any location (we recommend the home directory of your user account):

```bash
git clone https://github.com/kontalk/xmppserver-docker.git kontalk-server
cd kontalk-server
```

## Configure the containers

A setup script can be used to generate a local configuration:

```bash
./kontalk-setup
```

Answer all questions or skip them by pressing enter: the default value will be used (unless specified otherwise).

```
Please give a name to your new Kontalk server instance.
Kontalk instance name [prime]:

Kontalk is available in the following branches:
  master: bleeding-edge, latest modifications. Possibly unstable. Use for development only.
  staging: used in Kontalk test server. It should have the latest features with a certain degree of stability.
  production: used in Kontalk production server. Stable and tested.
You can also specify a version tag.

Kontalk version/branch to build [staging]:
XMPP service name [prime.kontalk.net]:
XMPP plain listen port [5222]:
XMPP secure listen port [5223]:
XMPP s2s listen port [5269]:
MySQL root password [root]:
MySQL kontalk password [kontalk]:
MySQL time zone [Europe/Rome]:

The HTTP file upload service is used by clients to upload and download media.
It should be exposed directly or better through a reverse proxy (e.g. Nginx).
We will ask you for the listen port exposed to the host system and the URLs seen by clients.
Note that those URLs should be available from the outside, i.e. from the Internet

HTTP file upload service listen port [8828]:
HTTP file upload URL [https://10.0.2.2:8828/]:
HTTP file download URL [https://10.0.2.2:8828/]:
Max HTTP upload file size [20971520]:

You can now build the images by running ./launcher bootstrap
```

A file called `local.properties` will be created. Please note that any modification
to this file will require a rebuild ([see below](#upgraderebuild-the-containers)).
If all went ok, you can now do the bootstrap:

```bash
./launcher bootstrap
```

This should take some time: it will build the images and setup the containers.

At this point, if you want to just run a development server for tests, just
go straight below to [the actual startup](#start-the-containers).

## Setting up the server

A more serious environment will require prior creation of a SSL certificate and
a GPG key for the server. For the containers to use them, certificate and keys must
be located in:

* `config/server-private.key`: GPG server private key (**must not be password-protected!**)
* `config/server-public.key`: GPG server public key
* `config/privatekey.pem`: SSL private key
* `config/certificate.pem`: SSL certificate
* `config/cachain.pem`: SSL certificate authorities chain (optional)

Your setup will also surely include some tweaking on the server configuration.
You can edit the default configuration file `config/init.properties.in`.
The local server tutorial can be used to configure some of the parameters,
e.g. [registration providers](//github.com/kontalk/tigase-kontalk/blob/master/docs/local-server-howto.md#registration) or
[push notifications](//github.com/kontalk/tigase-kontalk/blob/master/docs/local-server-howto.md#push-notifications). A quick
reference to Tigase parameters can be found [here](http://docs.tigase.org/tigase-server/7.1.0/Properties_Guide/html/),
while the administration guide is [here](http://docs.tigase.org/tigase-server/7.1.0/Administration_Guide/html/).
We strongly recommend giving it a look to know if the default configuration suits your needs.

> When configuring Tigase init.properties.in file, leave placeholders for
variables untouched (e.g. `{{ .Env.FINGERPRINT }}`): they will be replaced
automatically.

Another file to look into would be `config/tigase.conf`. Some environment variables are defined,
you should especially check JAVA\_OPTIONS which contains the JVM parameters (heap memory, GC parameters, etc.).
Note that despite the extension, it's a simple shell script file sourced by the server startup script,
so bash syntax applies.

### Trust store for outgoing SSL connection

You can use a custom trust store for outgoing SSL connections (e.g. from registration or push providers). The launcher
script will produce one automatically for you. Just create a directory called `trustedcerts` and put one certificate
per file in there. Be sure the files end in `.cer`. Then update JAVA\_OPTIONS in `config/tigase.conf` and add this at the end:

```
 -Djavax.net.ssl.trustStore=truststore -Djavax.net.ssl.trustStorePassword=truststore
```

## Start the containers

Run this command:

```bash
./launcher start
```

The script will start 3 containers in the background:

* **xmpp**: the actual [Tigase XMPP server](http://tigase.net/content/tigase-xmpp-server)
* **db**: MySQL database server
* **httpupload**: the HTTP upload component needed to upload media (pictures, audio files, etc.)

The actual container names will be prefixed with the instance name given to `kontalk-setup`.
For example, if you typed "prime" as the instance name, containers will be called prime\_xmpp, prime\_db, prime\_httpupload.

The instance name can be changed in local.properties (INSTANCE\_NAME), but before doing that you
must destroy the old containers (`./launcher destroy`), alter INSTANCE\_NAME and then run `./launcher bootstrap`.

The launcher script has some other useful commands that can be used. You can see
a quick help by launching it without any arguments.

## Upgrade/rebuild the containers

You must rebuild the containers if you:

* modified local.properties (run `./launcher rebuild`)
* want to upgrade the Kontalk server (meaning modifying the VERSION property in local.properties)

You don't need to rebuild if you modify `config/init.properties.in`, but you will need to restart it:

```bash
./launcher restart
```

## Help and support

You can ask for further information on how to deploy a Kontalk server and share your experiences on our community forum:
https://forum.kontalk.org/c/support/server
