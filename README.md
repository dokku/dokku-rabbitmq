# dokku rabbitmq [![Build Status](https://img.shields.io/github/actions/workflow/status/dokku/dokku-rabbitmq/ci.yml?branch=master&style=flat-square "Build Status")](https://github.com/dokku/dokku-rabbitmq/actions/workflows/ci.yml?query=branch%3Amaster) [![IRC Network](https://img.shields.io/badge/irc-libera-blue.svg?style=flat-square "IRC Libera")](https://webchat.libera.chat/?channels=dokku)

Official rabbitmq plugin for dokku. Currently defaults to installing [rabbitmq 4.1.2-management](https://hub.docker.com/_/rabbitmq/).

## Requirements

- dokku 0.19.x+
- docker 1.8.x

## Installation

```shell
# on 0.19.x+
sudo dokku plugin:install https://github.com/dokku/dokku-rabbitmq.git --name rabbitmq
```

## Commands

```
rabbitmq:app-links <app>                           # list all rabbitmq service links for a given app
rabbitmq:backup-set-public-key-encryption <service> <public-key-id> # set GPG Public Key encryption for all future backups of rabbitmq service
rabbitmq:backup-unset-public-key-encryption <service> # unset GPG Public Key encryption for future backups of the rabbitmq service
rabbitmq:create <service> [--create-flags...]      # create a rabbitmq service
rabbitmq:destroy <service> [-f|--force]            # delete the rabbitmq service/data/container if there are no links left
rabbitmq:enter <service>                           # enter or run a command in a running rabbitmq service container
rabbitmq:exists <service>                          # check if the rabbitmq service exists
rabbitmq:expose <service> <ports...>               # expose a rabbitmq service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)
rabbitmq:info <service> [--single-info-flag]       # print the service information
rabbitmq:link <service> <app> [--link-flags...]    # link the rabbitmq service to the app
rabbitmq:linked <service> <app>                    # check if the rabbitmq service is linked to an app
rabbitmq:links <service>                           # list all apps linked to the rabbitmq service
rabbitmq:list                                      # list all rabbitmq services
rabbitmq:logs <service> [-t|--tail] <tail-num-optional> # print the most recent log(s) for this service
rabbitmq:pause <service>                           # pause a running rabbitmq service
rabbitmq:promote <service> <app>                   # promote service <service> as RABBITMQ_URL in <app>
rabbitmq:restart <service>                         # graceful shutdown and restart of the rabbitmq service container
rabbitmq:set <service> <key> <value>               # set or clear a property for a service
rabbitmq:start <service>                           # start a previously stopped rabbitmq service
rabbitmq:stop <service>                            # stop a running rabbitmq service
rabbitmq:unexpose <service>                        # unexpose a previously exposed rabbitmq service
rabbitmq:unlink <service> <app>                    # unlink the rabbitmq service from the app
rabbitmq:upgrade <service> [--upgrade-flags...]    # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to rabbitmq:help. Plugin help output in conjunction with any files in the `docs/` folder is used to generate the plugin documentation. Please consult the `rabbitmq:help` command for any undocumented commands.

### Basic Usage

### create a rabbitmq service

```shell
# usage
dokku rabbitmq:create <service> [--create-flags...]
```

flags:

- `-c|--config-options "--args --go=here"`: extra arguments to pass to the container create command (default: `None`)
- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-m|--memory MEMORY`: container memory limit in megabytes (default: unlimited)
- `-N|--initial-network INITIAL_NETWORK`: the initial network to attach the service to
- `-p|--password PASSWORD`: override the user-level service password
- `-P|--post-create-network NETWORKS`: a comma-separated list of networks to attach the service container to after service creation
- `-r|--root-password PASSWORD`: override the root-level service password
- `-S|--post-start-network NETWORKS`: a comma-separated list of networks to attach the service container to after service start
- `-s|--shm-size SHM_SIZE`: override shared memory size for rabbitmq docker container

Create a rabbitmq service named lollipop:

```shell
dokku rabbitmq:create lollipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the rabbitmq image.

```shell
export RABBITMQ_IMAGE="rabbitmq"
export RABBITMQ_IMAGE_VERSION="${PLUGIN_IMAGE_VERSION}"
dokku rabbitmq:create lollipop
```

You can also specify custom environment variables to start the rabbitmq service in semicolon-separated form.

```shell
export RABBITMQ_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku rabbitmq:create lollipop
```

### print the service information

```shell
# usage
dokku rabbitmq:info <service> [--single-info-flag]
```

flags:

- `--config-dir`: show the service configuration directory
- `--data-dir`: show the service data directory
- `--dsn`: show the service DSN
- `--exposed-ports`: show service exposed ports
- `--id`: show the service container id
- `--internal-ip`: show the service internal ip
- `--initial-network`: show the initial network being connected to
- `--links`: show the service app links
- `--post-create-network`: show the networks to attach to after service container creation
- `--post-start-network`: show the networks to attach to after service container start
- `--service-root`: show the service root directory
- `--status`: show the service running status
- `--version`: show the service image version

Get connection information as follows:

```shell
dokku rabbitmq:info lollipop
```

You can also retrieve a specific piece of service info via flags:

```shell
dokku rabbitmq:info lollipop --config-dir
dokku rabbitmq:info lollipop --data-dir
dokku rabbitmq:info lollipop --dsn
dokku rabbitmq:info lollipop --exposed-ports
dokku rabbitmq:info lollipop --id
dokku rabbitmq:info lollipop --internal-ip
dokku rabbitmq:info lollipop --initial-network
dokku rabbitmq:info lollipop --links
dokku rabbitmq:info lollipop --post-create-network
dokku rabbitmq:info lollipop --post-start-network
dokku rabbitmq:info lollipop --service-root
dokku rabbitmq:info lollipop --status
dokku rabbitmq:info lollipop --version
```

### list all rabbitmq services

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
dokku rabbitmq:logs <service> [-t|--tail] <tail-num-optional>
```

flags:

- `-t|--tail [<tail-num>]`: do not stop when end of the logs are reached and wait for additional output

You can tail logs for a particular service:

```shell
dokku rabbitmq:logs lollipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku rabbitmq:logs lollipop --tail
```

The default tail setting is to show all logs, but an initial count can also be specified:

```shell
dokku rabbitmq:logs lollipop --tail 5
```

### link the rabbitmq service to the app

```shell
# usage
dokku rabbitmq:link <service> <app> [--link-flags...]
```

flags:

- `-a|--alias "BLUE_DATABASE"`: an alternative alias to use for linking to an app via environment variable
- `-q|--querystring "pool=5"`: ampersand delimited querystring arguments to append to the service link
- `-n|--no-restart "false"`: whether or not to restart the app on link (default: true)

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
RABBITMQ_URL=amqp://lollipop:SOME_PASSWORD@dokku-rabbitmq-lollipop:5672/lollipop
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the `expose` subcommand. Another service can be linked to your app:

```shell
dokku rabbitmq:link other_service playground
```

It is possible to change the protocol for `RABBITMQ_URL` by setting the environment variable `RABBITMQ_DATABASE_SCHEME` on the app. Doing so will after linking will cause the plugin to think the service is not linked, and we advise you to unlink before proceeding.

```shell
dokku config:set playground RABBITMQ_DATABASE_SCHEME=amqp2
dokku rabbitmq:link lollipop playground
```

This will cause `RABBITMQ_URL` to be set as:

```
amqp2://lollipop:SOME_PASSWORD@dokku-rabbitmq-lollipop:5672/lollipop
```

### unlink the rabbitmq service from the app

```shell
# usage
dokku rabbitmq:unlink <service> <app>
```

flags:

- `-n|--no-restart "false"`: whether or not to restart the app on unlink (default: true)

You can unlink a rabbitmq service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku rabbitmq:unlink lollipop playground
```

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

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### enter or run a command in a running rabbitmq service container

```shell
# usage
dokku rabbitmq:enter <service>
```

A bash prompt can be opened against a running service. Filesystem changes will not be saved to disk.

> NOTE: disconnecting from ssh while running this command may leave zombie processes due to moby/moby#9098

```shell
dokku rabbitmq:enter lollipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk.

```shell
dokku rabbitmq:enter lollipop touch /tmp/test
```

### expose a rabbitmq service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)

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

### unexpose a previously exposed rabbitmq service

```shell
# usage
dokku rabbitmq:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (`0.0.0.0`):

```shell
dokku rabbitmq:unexpose lollipop
```

### promote service <service> as RABBITMQ_URL in <app>

```shell
# usage
dokku rabbitmq:promote <service> <app>
```

If you have a rabbitmq service linked to an app and try to link another rabbitmq service another link environment variable will be generated automatically:

```
DOKKU_RABBITMQ_BLUE_URL=amqp://other_service:ANOTHER_PASSWORD@dokku-rabbitmq-other-service:5672/other_service
```

You can promote the new service to be the primary one:

> NOTE: this will restart your app

```shell
dokku rabbitmq:promote other_service playground
```

This will replace `RABBITMQ_URL` with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
RABBITMQ_URL=amqp://other_service:ANOTHER_PASSWORD@dokku-rabbitmq-other-service:5672/other_service
DOKKU_RABBITMQ_BLUE_URL=amqp://other_service:ANOTHER_PASSWORD@dokku-rabbitmq-other-service:5672/other_service
DOKKU_RABBITMQ_SILVER_URL=amqp://lollipop:SOME_PASSWORD@dokku-rabbitmq-lollipop:5672/lollipop
```

### start a previously stopped rabbitmq service

```shell
# usage
dokku rabbitmq:start <service>
```

Start the service:

```shell
dokku rabbitmq:start lollipop
```

### stop a running rabbitmq service

```shell
# usage
dokku rabbitmq:stop <service>
```

Stop the service and removes the running container:

```shell
dokku rabbitmq:stop lollipop
```

### pause a running rabbitmq service

```shell
# usage
dokku rabbitmq:pause <service>
```

Pause the running container for the service:

```shell
dokku rabbitmq:pause lollipop
```

### graceful shutdown and restart of the rabbitmq service container

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

- `-c|--config-options "--args --go=here"`: extra arguments to pass to the container create command (default: `None`)
- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-N|--initial-network INITIAL_NETWORK`: the initial network to attach the service to
- `-P|--post-create-network NETWORKS`: a comma-separated list of networks to attach the service container to after service creation
- `-R|--restart-apps "true"`: whether or not to force an app restart (default: false)
- `-S|--post-start-network NETWORKS`: a comma-separated list of networks to attach the service container to after service start
- `-s|--shm-size SHM_SIZE`: override shared memory size for rabbitmq docker container

You can upgrade an existing service to a new image or image-version:

```shell
dokku rabbitmq:upgrade lollipop
```

### Service Automation

Service scripting can be executed using the following commands:

### list all rabbitmq service links for a given app

```shell
# usage
dokku rabbitmq:app-links <app>
```

List all rabbitmq services that are linked to the `playground` app.

```shell
dokku rabbitmq:app-links playground
```

### check if the rabbitmq service exists

```shell
# usage
dokku rabbitmq:exists <service>
```

Here we check if the lollipop rabbitmq service exists.

```shell
dokku rabbitmq:exists lollipop
```

### check if the rabbitmq service is linked to an app

```shell
# usage
dokku rabbitmq:linked <service> <app>
```

Here we check if the lollipop rabbitmq service is linked to the `playground` app.

```shell
dokku rabbitmq:linked lollipop playground
```

### list all apps linked to the rabbitmq service

```shell
# usage
dokku rabbitmq:links <service>
```

List all apps linked to the `lollipop` rabbitmq service.

```shell
dokku rabbitmq:links lollipop
```
### Backups

Datastore backups are supported via AWS S3 and S3 compatible services like [minio](https://github.com/minio/minio).

You may skip the `backup-auth` step if your dokku install is running within EC2 and has access to the bucket via an IAM profile. In that case, use the `--use-iam` option with the `backup` command.

Backups can be performed using the backup commands:

### set GPG Public Key encryption for all future backups of rabbitmq service

```shell
# usage
dokku rabbitmq:backup-set-public-key-encryption <service> <public-key-id>
```

Set the `GPG` Public Key for encrypting backups:

```shell
dokku rabbitmq:backup-set-public-key-encryption lollipop
```

### unset GPG Public Key encryption for future backups of the rabbitmq service

```shell
# usage
dokku rabbitmq:backup-unset-public-key-encryption <service>
```

Unset the `GPG` Public Key encryption for backups:

```shell
dokku rabbitmq:backup-unset-public-key-encryption lollipop
```

### Disabling `docker image pull` calls

If you wish to disable the `docker image pull` calls that the plugin triggers, you may set the `RABBITMQ_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker image pull` is disabled.
