# dokku rabbitmq [![Build Status](https://img.shields.io/github/actions/workflow/status/dokku/dokku-rabbitmq/ci.yml?branch=master&style=flat-square "Build Status")](https://github.com/dokku/dokku-rabbitmq/actions/workflows/ci.yml?query=branch%3Amaster) [![IRC Network](https://img.shields.io/badge/irc-libera-blue.svg?style=flat-square "IRC Libera")](https://webchat.libera.chat/?channels=dokku)

Official rabbitmq plugin for dokku. Currently defaults to installing [rabbitmq 4.3.6-management](https://hub.docker.com/_/rabbitmq/).

## Requirements

- dokku 0.35.x+
- docker 1.8.x

## Installation

```shell
# on 0.35.x+
sudo dokku plugin:install https://github.com/dokku/dokku-rabbitmq.git --name rabbitmq
```

## Commands

```
rabbitmq:app-links [<app>]                         # list all RabbitMQ service links for a given app
rabbitmq:create <service> [--create-flags...]      # create a RabbitMQ service
rabbitmq:destroy <service> [-f|--force]            # delete the RabbitMQ service/data/container if there are no links left
rabbitmq:enter <service>                           # enter or run a command in a running RabbitMQ service container
rabbitmq:exists <service>                          # check if the RabbitMQ service exists
rabbitmq:expose <service> <ports...>               # expose a RabbitMQ service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)
rabbitmq:info [<service>] [--info-flags...]        # print the service information
rabbitmq:link <service> [<app>] [--link-flags...]  # link the RabbitMQ service to the app
rabbitmq:linked <service> [<app>]                  # check if the RabbitMQ service is linked to an app
rabbitmq:links <service>                           # list all apps linked to the RabbitMQ service
rabbitmq:list                                      # list all RabbitMQ services
rabbitmq:logs <service> [-t|--tail [<tail-num>]]   # print the most recent log(s) for this service
rabbitmq:mount [--replace] <service> <source:container-dir[:options]>... # mount a host path or docker volume into the service container
rabbitmq:pause <service>                           # pause a running RabbitMQ service
rabbitmq:promote <service> [<app>]                 # promote service <service> as RABBITMQ_URL in <app>
rabbitmq:reexpose <service>                        # reexpose a RabbitMQ service, applying its expose settings without restarting it
rabbitmq:restart <service>                         # graceful shutdown and restart of the RabbitMQ service container
rabbitmq:set <service> <key> <value>               # set or clear a property for a service
rabbitmq:start <service>                           # start a previously stopped RabbitMQ service
rabbitmq:stop <service>                            # stop a running RabbitMQ service
rabbitmq:unexpose <service>                        # unexpose a previously exposed RabbitMQ service
rabbitmq:unlink <service> [<app>] [-n|--no-restart] # unlink the RabbitMQ service from the app
rabbitmq:unmount [--all] <service> [<source:container-dir>...] # remove one or all mounts from the service container
rabbitmq:upgrade <service> [--upgrade-flags...]    # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to rabbitmq:help. Plugin help output in conjunction with any files in the `docs/` folder is used to generate the plugin documentation. Please consult the `rabbitmq:help` command for any undocumented commands.

### Basic Usage

### create a RabbitMQ service

```shell
# usage
dokku rabbitmq:create <service> [--create-flags...]
```

flags:

- `-c|--config-options <string>`: extra arguments for the process the service container runs, not docker flags; use mount for mounts
- `-C|--custom-env <string>`: semi-colon delimited environment variables to start the service with
- `-i|--image <string>`: the image name to start the service with
- `-I|--image-version <string>`: the image version to start the service with
- `-N|--initial-network <string>`: the initial network to attach the service to
- `--log-driver <string>`: the docker logging driver to run the service container with (default: the daemon's own)
- `--log-opt <strings>`: a comma-separated list of key=value docker log options for the service container
- `-m|--memory <int>`: container memory limit in megabytes (default: unlimited)
- `-p|--password <string>`: override the user-level service password, for datastores that have one
- `-P|--post-create-network <strings>`: a comma-separated list of networks to attach the service container to after service creation
- `-S|--post-start-network <strings>`: a comma-separated list of networks to attach the service container to after service start
- `--restart <string>`: the docker restart policy to run the service container with (default: always)
- `-r|--root-password <string>`: override the root-level service password, for datastores that have one
- `-s|--shm-size <string>`: override shared memory size for the service docker container
- `--volume <stringArray>`: a host path or docker volume to mount into the service container, as <source>:<container-dir>[:<options>], repeatable
- `--wait-timeout <string>`: seconds to wait for the service to become ready (default: the datastore's own)

Create a rabbitmq service named lollipop:

```shell
dokku rabbitmq:create lollipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the rabbitmq image.

```shell
export RABBITMQ_IMAGE="rabbitmq"
export RABBITMQ_IMAGE_VERSION="4.3.6-management"
dokku rabbitmq:create lollipop
```

An image other than rabbitmq has no version to fall back on, because the version this plugin pins belongs to rabbitmq, so name one alongside it.

```shell
dokku rabbitmq:create lollipop --image <image> --image-version <version>
```

You can also specify custom environment variables to start the rabbitmq service in semicolon-separated form.

```shell
export RABBITMQ_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku rabbitmq:create lollipop
```

The container log is bounded by whatever `dokku logs:set --global max-size` says, and by dokku's own default where it says nothing, which a service may override for itself.

```shell
dokku rabbitmq:create lollipop --log-opt max-size=20m,max-file=3
```

The container is restarted by docker whenever it stops, which a service may change for itself.

```shell
dokku rabbitmq:create lollipop --restart unless-stopped
```

The service is waited on until it answers, for as long as the datastore's own default, which a slow host may raise for every service with `RABBITMQ_WAIT_TIMEOUT` or a service may raise for itself.

```shell
dokku rabbitmq:create lollipop --wait-timeout 120
```

The config options are handed to the process the container runs, not to docker, so a host path or docker volume is mounted with --volume, which may be repeated.

```shell
dokku rabbitmq:create lollipop --volume /var/lib/dokku/data/storage/lollipop:/opt/extra:ro
```

The service passwords are generated unless they are given. A datastore without a root password refuses --root-password rather than dropping it.

```shell
dokku rabbitmq:create lollipop --password <password> --root-password <root-password>
```

### delete the RabbitMQ service/data/container if there are no links left

```shell
# usage
dokku rabbitmq:destroy <service> [-f|--force]
```

flags:

- `-f|--force`: force the destruction of the service

Destroy the service, it's data, and the running container:

```shell
dokku rabbitmq:destroy lollipop
```

A service that is still linked to an app is not destroyed, and the apps it is linked to are named. Unlink them first.

### print the service information

```shell
# usage
dokku rabbitmq:info [<service>] [--info-flags...]
```

flags:

- `--backend`: show the execution backend the service was created with
- `--backup-auth-fingerprint`: show a sha256 fingerprint of the stored backup access key id and secret
- `--backup-authenticated`: show whether backup credentials are stored for the service
- `--backup-bucket`: show the bucket scheduled backups are shipped to
- `--backup-default-region`: show the region backups authenticate against
- `--backup-encrypted`: show whether scheduled backups are encrypted with a passphrase
- `--backup-encryption-fingerprint`: show a sha256 fingerprint of the stored backup passphrase
- `--backup-endpoint-url`: show the s3-compatible endpoint backups are shipped to
- `--backup-keyserver`: show the keyserver backup public keys are fetched from
- `--backup-public-key-id`: show the gpg public key id backups are encrypted with
- `--backup-schedule`: show the cron schedule backups run on
- `--backup-signature-version`: show the signature version backups authenticate with
- `--backup-use-iam`: show whether scheduled backups authenticate with an instance role
- `--config-dir`: show the service configuration directory
- `--config-options`: show the config options the service container is run with
- `--custom-env`: show the custom environment the service container is run with
- `--data-dir`: show the service data directory
- `--database-name`: show the name of the database inside the service
- `--definition`: show the definition the service was created with
- `--dsn`: show the service DSN
- `--export-args`: show the extra arguments every export of the service is run with
- `--expose-address`: show the address exposed ports without one of their own are published on
- `--expose-source-range`: show the only range of client addresses the exposed ports accept
- `--exposed-ports`: show service exposed ports
- `--id`: show the service container id
- `--image`: show the image the service runs
- `--image-version`: show the image version the service was created with
- `--import-args`: show the extra arguments every import into the service is run with
- `--initial-network`: show the initial network being connected to
- `--internal-ip`: show the service internal ip
- `--links`: show the service app links
- `--log-driver`: show the docker logging driver the service container is run with
- `--log-opt`: show the docker log options the service container is run with
- `--memory`: show the memory limit the service container is run with
- `--mounts`: show the host paths and docker volumes mounted into the service container
- `--post-create-network`: show the networks to attach to after service container creation
- `--post-start-network`: show the networks to attach to after service container start
- `--restart-policy`: show the restart policy the service container is run with
- `--service`: show the name of the service
- `--service-root`: show the service root directory
- `--shm-size`: show the shared memory size the service container is run with
- `--status`: show the service running status
- `--version`: show the service image version
- `--wait-timeout`: show the seconds the service is waited on to become ready

Get connection information as follows:

```shell
dokku rabbitmq:info lollipop
```

Alongside the connection information this reports the properties set on the service, the state it was created with, and its backup settings. A property that was never set, or that was unset, reports as empty. Omit the service to report on every rabbitmq service:

```shell
dokku rabbitmq:info
```

The information can be read by machine, one json object per service:

```shell
dokku rabbitmq:info lollipop --format json
```

You can also retrieve a specific piece of service info via a flag, which prints it on its own:

```shell
dokku rabbitmq:info lollipop --dsn
dokku rabbitmq:info lollipop --status
dokku rabbitmq:info lollipop --initial-network
```

> NOTE: a flag cannot be combined with --format, and only one may be given

The properties rabbitmq:set writes are reported under the names it takes, so a value read here can be written back:

```shell
dokku rabbitmq:set lollipop initial-network my-network
```

The stored backup credentials and passphrase are never printed. Each is reported as a lowercase hex sha256 fingerprint of the stored value, with surrounding whitespace trimmed, so a copy of the values can be compared against it:

```shell
dokku rabbitmq:info lollipop --backup-auth-fingerprint
dokku rabbitmq:info lollipop --backup-encryption-fingerprint
```

The same fingerprints can be computed from the values that were passed to backup-auth and backup-set-encryption:

```
printf '%s\n%s' "$AWS_ACCESS_KEY_ID" "$AWS_SECRET_ACCESS_KEY" | sha256sum
printf '%s' "$PASSPHRASE" | sha256sum
```

### list all RabbitMQ services

```shell
# usage
dokku rabbitmq:list
```

List all services:

```shell
dokku rabbitmq:list
```

### print the most recent log(s) for this service

```shell
# usage
dokku rabbitmq:logs <service> [-t|--tail [<tail-num>]]
```

flags:

- `-t|--tail <int>`: tail the logs, optionally showing this many lines

You can tail logs for a particular service:

```shell
dokku rabbitmq:logs lollipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku rabbitmq:logs lollipop --tail
```

By default the last 100 lines are shown, but a different count can be specified:

```shell
dokku rabbitmq:logs lollipop --tail=5
```

### link the RabbitMQ service to the app

```shell
# usage
dokku rabbitmq:link <service> [<app>] [--link-flags...]
```

flags:

- `-a|--alias <string>`: the prefix of the config variable the service url is set as on the app, which is suffixed with _URL
- `-n|--no-restart`: whether to skip restarting the app
- `-q|--querystring <string>`: ampersand delimited querystring arguments to append to the service url after a ?

A rabbitmq service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our `playground` app.

> NOTE: this will restart your app

```shell
dokku rabbitmq:link lollipop playground
```

The following environment variables will be set automatically by docker (not on the app itself, so they won’t be listed when calling dokku config):

```
DOKKU_RABBITMQ_LOLLIPOP_NAME=/lollipop/DATABASE
DOKKU_RABBITMQ_LOLLIPOP_PORT=tcp://172.17.0.1:5672
DOKKU_RABBITMQ_LOLLIPOP_PORT_5672_TCP=tcp://172.17.0.1:5672
DOKKU_RABBITMQ_LOLLIPOP_PORT_5672_TCP_PROTO=tcp
DOKKU_RABBITMQ_LOLLIPOP_PORT_5672_TCP_PORT=5672
DOKKU_RABBITMQ_LOLLIPOP_PORT_5672_TCP_ADDR=172.17.0.1
```

The following will be set on the linked application by default:

```
RABBITMQ_URL=amqp://:SOME_PASSWORD@dokku-rabbitmq-lollipop:5672
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the `expose` subcommand. Another service can be linked to your app:

```shell
dokku rabbitmq:link other_service playground
```

The url can be set under another name with the `--alias` flag. The value given is the prefix of the config variable, which is suffixed with `_URL` and holds the same url:

```shell
dokku rabbitmq:link lollipop playground --alias BLUE_RABBITMQ
```

This will set the following on the linked application instead of `RABBITMQ_URL`:

```
BLUE_RABBITMQ_URL=amqp://:SOME_PASSWORD@dokku-rabbitmq-lollipop:5672
```

An alias whose variable is already set on the app is refused, and unlink removes the variable whatever alias it was set under. Arguments can be appended to the url as a querystring with the `--querystring` flag:

```shell
dokku rabbitmq:link lollipop playground --querystring "foo=bar&baz=qux"
```

This will cause `RABBITMQ_URL` to be set as:

```
amqp://:SOME_PASSWORD@dokku-rabbitmq-lollipop:5672?foo=bar&baz=qux
```

It is possible to change the protocol for `RABBITMQ_URL` by setting the environment variable `RABBITMQ_DATABASE_SCHEME` on the app. Doing so after linking means unlink no longer finds the variable it set, and leaves it in place, so we advise you to unlink before proceeding.

```shell
dokku config:set playground RABBITMQ_DATABASE_SCHEME=amqp2
dokku rabbitmq:link lollipop playground
```

This will cause `RABBITMQ_URL` to be set as:

```
amqp2://:SOME_PASSWORD@dokku-rabbitmq-lollipop:5672
```

### unlink the RabbitMQ service from the app

```shell
# usage
dokku rabbitmq:unlink <service> [<app>] [-n|--no-restart]
```

flags:

- `-n|--no-restart`: whether to skip restarting the app

You can unlink a rabbitmq service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku rabbitmq:unlink lollipop playground
```

An app is still linked after its `RABBITMQ_URL` is changed to point elsewhere, and is unlinked the same way. The variable it now holds is not the service's, so it is left alone, nothing is unset, the app is not restarted, and a warning says so.

### set or clear a property for a service

```shell
# usage
dokku rabbitmq:set <service> <key> <value>
```

Set the network to attach after the service container is started:

```shell
dokku rabbitmq:set lollipop post-create-network custom-network
```

Set multiple networks:

```shell
dokku rabbitmq:set lollipop post-create-network custom-network,other-network
```

Unset the post-create-network value:

```shell
dokku rabbitmq:set lollipop post-create-network
```

Set the keyserver a public key for backup encryption is fetched from:

```shell
dokku rabbitmq:set lollipop backup-keyserver hkp://keys.example.com
```

Cap the container log at a size of your own rather than the one it inherits:

```shell
dokku rabbitmq:set lollipop log-opt max-size=20m,max-file=3
```

Keep the log unbounded, which is what a service had before there was anything to say here:

```shell
dokku rabbitmq:set lollipop log-opt max-size=unlimited
```

Send the container log somewhere other than the daemon's own driver:

```shell
dokku rabbitmq:set lollipop log-driver journald
```

Restart the container unless it was stopped on purpose, including across a docker restart:

```shell
dokku rabbitmq:set lollipop restart-policy unless-stopped
```

Go back to always restarting the container:

```shell
dokku rabbitmq:set lollipop restart-policy
```

Wait up to two minutes for the service to answer, used the next time it is started:

```shell
dokku rabbitmq:set lollipop wait-timeout 120
```

Go back to the wait timeout the host or the datastore sets:

```shell
dokku rabbitmq:set lollipop wait-timeout
```

Publish exposed ports that have no address of their own on one address rather than on every interface:

```shell
dokku rabbitmq:set lollipop expose-address 10.0.0.5
```

Only accept connections to the exposed ports from clients in one `IP` address or `CIDR`:

```shell
dokku rabbitmq:set lollipop expose-source-range 10.0.0.0/8
```

Go back to accepting every client:

```shell
dokku rabbitmq:set lollipop expose-source-range
```

> NOTE: a log setting or a restart policy reaches the container the next time one is built. rabbitmq:restart keeps the container it has, so use rabbitmq:stop and then rabbitmq:start on a service that is already running.
> NOTE: an expose-address or expose-source-range reaches an exposed service with rabbitmq:reexpose, which replaces the container publishing its ports and leaves the service container running.

### mount a host path or docker volume into the service container

```shell
# usage
dokku rabbitmq:mount [--replace] <service> <source:container-dir[:options]>...
```

flags:

- `--replace`: replace the service's entire set of mounts with the ones given
- `--volume-chown <string>`: who to hand the mounted directory to, for a host path inside the service's directory; not valid with --replace
- `--volume-options <string>`: comma-separated docker mount options, such as z or nocopy; not valid with --replace
- `--volume-readonly`: mount the volume read only; not valid with --replace
- `--volume-subpath <string>`: a subpath within the source to mount rather than the source itself; not valid with --replace

Mount a host directory into the service container:

```shell
dokku rabbitmq:mount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra
```

The source is an absolute host path, which must already exist, or the name of a docker volume. Options follow a second colon: ro or rw, docker's own mount options, volume-subpath=<path> and volume-chown=<option>:

```shell
dokku rabbitmq:mount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra:ro,z
```

A subpath mounts a directory within the source rather than the source itself. A docker volume mounted from a subpath needs Docker Engine 26.0 or newer, and takes no mount option but nocopy.

```shell
dokku rabbitmq:mount lollipop my-volume:/opt/extra:volume-subpath=uploads
```

A chown hands the mounted directory to a user before the container is made: herokuish, heroku, paketo, root or a uid. It is only taken for a host path inside the service's own directory.

```shell
dokku rabbitmq:mount lollipop /var/lib/dokku/services/rabbitmq/lollipop/extra:/opt/extra:volume-chown=heroku
```

The same can be said with flags instead:

```shell
dokku rabbitmq:mount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra --volume-readonly --volume-options z
```

Mounting the same source at the same directory again rewrites its options:

```shell
dokku rabbitmq:mount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra
```

Replace every mount the service has with the ones given:

```shell
dokku rabbitmq:mount --replace lollipop /srv/a:/opt/a:ro /srv/b:/opt/b
```

> NOTE: a mount reaches the container the next time one is built. rabbitmq:restart keeps the container it has, so use rabbitmq:stop and then rabbitmq:start on a service that is already running.

### remove one or all mounts from the service container

```shell
# usage
dokku rabbitmq:unmount [--all] <service> [<source:container-dir>...]
```

flags:

- `--all`: remove every mount the service has

Remove a mount, naming it the way it was mounted:

```shell
dokku rabbitmq:unmount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra
```

Remove every mount the service has:

```shell
dokku rabbitmq:unmount --all lollipop
```

> NOTE: the mount is removed from the container the next time one is built. rabbitmq:restart keeps the container it has, so use rabbitmq:stop and then rabbitmq:start on a service that is already running.

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### enter or run a command in a running RabbitMQ service container

```shell
# usage
dokku rabbitmq:enter <service>
```

A shell can be opened against a running service. Filesystem changes will not be saved to disk.

> NOTE: disconnecting from ssh while running this command may leave zombie processes due to moby/moby#9098

```shell
dokku rabbitmq:enter lollipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk.

```shell
dokku rabbitmq:enter lollipop touch /tmp/test
```

### expose a RabbitMQ service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)

```shell
# usage
dokku rabbitmq:expose <service> <ports...>
```

Expose the service on the service's normal ports, allowing access to it from the public interface (`0.0.0.0`):

```shell
dokku rabbitmq:expose lollipop 5672 4369 35197 15672
```

Expose the service on the service's normal ports, with the first on a specified ip address (127.0.0.1):

```shell
dokku rabbitmq:expose lollipop 127.0.0.1:5672 4369 35197 15672
```

Expose the service on random ports on a single address, and only to clients in one network:

```shell
dokku rabbitmq:set lollipop expose-address 10.0.0.5
dokku rabbitmq:set lollipop expose-source-range 10.0.0.0/8
dokku rabbitmq:expose lollipop
```

### unexpose a previously exposed RabbitMQ service

```shell
# usage
dokku rabbitmq:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (`0.0.0.0`):

```shell
dokku rabbitmq:unexpose lollipop
```

### reexpose a RabbitMQ service, applying its expose settings without restarting it

```shell
# usage
dokku rabbitmq:reexpose <service>
```

Apply a changed expose-address or expose-source-range to an exposed service, on the ports it is already exposed on:

```shell
dokku rabbitmq:set lollipop expose-source-range 10.0.0.0/8
dokku rabbitmq:reexpose lollipop
```

> NOTE: only the container publishing the service's ports is replaced, so the service keeps running, though connections made through the exposed ports are dropped. A service that is not exposed, or is not running, is refused.

### promote service <service> as RABBITMQ_URL in <app>

```shell
# usage
dokku rabbitmq:promote <service> [<app>]
```

If you have a rabbitmq service linked to an app and try to link another rabbitmq service another link environment variable will be generated automatically:

```
DOKKU_RABBITMQ_AQUA_URL=amqp://:ANOTHER_PASSWORD@dokku-rabbitmq-other-service:5672/other_service
```

You can promote the new service to be the primary one:

> NOTE: this will restart your app

```shell
dokku rabbitmq:promote other_service playground
```

This will replace `RABBITMQ_URL` with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
RABBITMQ_URL=amqp://:ANOTHER_PASSWORD@dokku-rabbitmq-other-service:5672/other_service
DOKKU_RABBITMQ_AQUA_URL=amqp://:ANOTHER_PASSWORD@dokku-rabbitmq-other-service:5672/other_service
DOKKU_RABBITMQ_BLACK_URL=amqp://:SOME_PASSWORD@dokku-rabbitmq-lollipop:5672/lollipop
```

### start a previously stopped RabbitMQ service

```shell
# usage
dokku rabbitmq:start <service>
```

Start the service:

```shell
dokku rabbitmq:start lollipop
```

A service comes back on the version it was created with, or was last upgraded to, whatever version the plugin ships now. The image is fetched if the host no longer has it. A service that has never recorded a version and has no container left to read one from cannot be placed, and is reported rather than started on a guess. Use rabbitmq:upgrade to say which version it should run.

### stop a running RabbitMQ service

```shell
# usage
dokku rabbitmq:stop <service>
```

Stop the service and removes the running container:

```shell
dokku rabbitmq:stop lollipop
```

### pause a running RabbitMQ service

```shell
# usage
dokku rabbitmq:pause <service>
```

Pause the running container for the service:

```shell
dokku rabbitmq:pause lollipop
```

### graceful shutdown and restart of the RabbitMQ service container

```shell
# usage
dokku rabbitmq:restart <service>
```

Restart the service:

```shell
dokku rabbitmq:restart lollipop
```

### upgrade service <service> to the specified versions

```shell
# usage
dokku rabbitmq:upgrade <service> [--upgrade-flags...]
```

flags:

- `-c|--config-options <string>`: extra arguments for the process the service container runs, not docker flags; use mount for mounts
- `-C|--custom-env <string>`: semi-colon delimited environment variables to start the service with
- `-i|--image <string>`: the image to upgrade the service to
- `-I|--image-version <string>`: the image version to upgrade the service to
- `-N|--initial-network <string>`: the initial network to attach the service to
- `--log-driver <string>`: the docker logging driver to run the service container with (default: the daemon's own)
- `--log-opt <strings>`: a comma-separated list of key=value docker log options for the service container
- `-m|--memory <int>`: container memory limit in megabytes, 0 for unlimited
- `-P|--post-create-network <strings>`: a comma-separated list of networks to attach the service container to after service creation
- `-S|--post-start-network <strings>`: a comma-separated list of networks to attach the service container to after service start
- `--restart <string>`: the docker restart policy to run the service container with (default: always)
- `-R|--restart-apps`: whether to stop and start the linked apps around the upgrade
- `-s|--shm-size <string>`: override shared memory size for the service docker container
- `--volume <stringArray>`: a host path or docker volume to mount into the service container, as <source>:<container-dir>[:<options>], repeatable
- `--wait-timeout <string>`: seconds to wait for the service to become ready (default: the datastore's own)

You can upgrade an existing service to a new image or image-version:

```shell
dokku rabbitmq:upgrade lollipop
```

This is the only command that changes the version a service runs. With no version named it moves to the newest the service's own major version ships, which leaves the data where it is.

```shell
dokku rabbitmq:upgrade lollipop --image-version 1.2.3
```

Moving across a major version has to be asked for by name, because it is not a tag change: the data is mounted somewhere different under the new one, and pointing the version back does not undo it. A service keeps the mounts it has unless --volume is passed, which replaces them, and each one is checked against the new container before the old one is taken away.

```shell
dokku rabbitmq:upgrade lollipop --volume /var/lib/dokku/data/storage/lollipop:/opt/extra:ro
```

A service keeps its memory limit unless --memory is passed, and --memory 0 removes it.

```shell
dokku rabbitmq:upgrade lollipop --memory 512
```

### Service Automation

Service scripting can be executed using the following commands:

### list all RabbitMQ service links for a given app

```shell
# usage
dokku rabbitmq:app-links [<app>]
```

List all rabbitmq services that are linked to the `playground` app.

```shell
dokku rabbitmq:app-links playground
```

### check if the RabbitMQ service exists

```shell
# usage
dokku rabbitmq:exists <service>
```

Here we check if the lollipop rabbitmq service exists.

```shell
dokku rabbitmq:exists lollipop
```

### check if the RabbitMQ service is linked to an app

```shell
# usage
dokku rabbitmq:linked <service> [<app>]
```

Here we check if the lollipop rabbitmq service is linked to the `playground` app.

```shell
dokku rabbitmq:linked lollipop playground
```

### list all apps linked to the RabbitMQ service

```shell
# usage
dokku rabbitmq:links <service>
```

List all apps linked to the `lollipop` rabbitmq service.

```shell
dokku rabbitmq:links lollipop
```

Renaming an app moves its link onto the new name, and cloning an app links the clone as well as the original.

### Limiting where and to whom a service is exposed

An exposed service's ports are published on every interface unless they are given an address of their own. To publish them on one address instead, set the service's `expose-address` property with `dokku rabbitmq:set`, and to accept connections only from clients in one IP address or CIDR, set its `expose-source-range` property. Either reaches a running service with `dokku rabbitmq:reexpose`, which leaves the service running.

Only one source range can be given. The range is checked against the address a connection reaches the service from, which for a connection to the exposed port on the loopback interface, or an IPv6 connection to a service network without IPv6, is the docker network's gateway rather than the client, so with a range that leaves the gateway out, connecting to `127.0.0.1` from the dokku host itself is refused.

### Waiting for a service to become ready

A service is waited on until it answers on its port after it is created, cloned, started, restarted, upgraded or exposed. If it takes longer than that to start - on a slow host, or with an image that does more on its first boot - the command fails with `ERROR: unable to connect`.

To wait longer for every rabbitmq service on the host, set the `RABBITMQ_WAIT_TIMEOUT` environment variable to a number of seconds. To wait longer for a single service, set its `wait-timeout` property with `dokku rabbitmq:set` or pass `--wait-timeout` to `create`, `clone` or `upgrade`. The service's own setting is used first, then the environment variable, then the datastore's default.

### Disabling `docker image pull` calls

If you wish to disable the `docker image pull` calls that the plugin triggers, you may set the `RABBITMQ_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker image pull` is disabled.
