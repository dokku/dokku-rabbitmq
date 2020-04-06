# dokku rabbitmq [![Build Status](https://img.shields.io/circleci/project/github/dokku/dokku-rabbitmq.svg?branch=master&style=flat-square "Build Status")](https://circleci.com/gh/dokku/dokku-rabbitmq/tree/master) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg?style=flat-square "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official rabbitmq plugin for dokku. Currently defaults to installing [rabbitmq 3.7.21-management](https://hub.docker.com/_/rabbitmq/).

## Requirements

- dokku 0.12.x+
- docker 1.8.x

## Installation

```shell
# on 0.12.x+
sudo dokku plugin:install https://github.com/dokku/dokku-rabbitmq.git rabbitmq
```

## Commands

```
rabbitmq:app-links <app>                        # list all rabbitmq service links for a given app
rabbitmq:create <service> [--create-flags...]   # create a rabbitmq service
rabbitmq:destroy <service> [-f|--force]         # delete the rabbitmq service/data/container if there are no links left
rabbitmq:enter <service>                        # enter or run a command in a running rabbitmq service container
rabbitmq:exists <service>                       # check if the rabbitmq service exists
rabbitmq:expose <service> <ports...>            # expose a rabbitmq service on custom port if provided (random port otherwise)
rabbitmq:info <service> [--single-info-flag]    # print the service information
rabbitmq:link <service> <app> [--link-flags...] # link the rabbitmq service to the app
rabbitmq:linked <service> <app>                 # check if the rabbitmq service is linked to an app
rabbitmq:links <service>                        # list all apps linked to the rabbitmq service
rabbitmq:list                                   # list all rabbitmq services
rabbitmq:logs <service> [-t|--tail]             # print the most recent log(s) for this service
rabbitmq:promote <service> <app>                # promote service <service> as RABBITMQ_URL in <app>
rabbitmq:restart <service>                      # graceful shutdown and restart of the rabbitmq service container
rabbitmq:start <service>                        # start a previously stopped rabbitmq service
rabbitmq:stop <service>                         # stop a running rabbitmq service
rabbitmq:unexpose <service>                     # unexpose a previously exposed rabbitmq service
rabbitmq:unlink <service> <app>                 # unlink the rabbitmq service from the app
rabbitmq:upgrade <service> [--upgrade-flags...] # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to rabbitmq:help. Please consult the `rabbitmq:help` command for any undocumented commands.

### Basic Usage

### create a rabbitmq service

```shell
# usage
dokku rabbitmq:create <service> [--create-flags...]
```

Create a rabbitmq service named lolipop:

```shell
dokku rabbitmq:create lolipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the ${plugin_image} image.

```shell
export RABBITMQ_IMAGE="${PLUGIN_IMAGE}"
export RABBITMQ_IMAGE_VERSION="${PLUGIN_IMAGE_VERSION}"
dokku rabbitmq:create lolipop
```

You can also specify custom environment variables to start the rabbitmq service in semi-colon separated form.

```shell
export RABBITMQ_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku rabbitmq:create lolipop
```

### print the service information

```shell
# usage
dokku rabbitmq:info <service> [--single-info-flag]
```

Get connection information as follows:

```shell
dokku rabbitmq:info lolipop
```

You can also retrieve a specific piece of service info via flags:

```shell
dokku rabbitmq:info lolipop --config-dir
dokku rabbitmq:info lolipop --data-dir
dokku rabbitmq:info lolipop --dsn
dokku rabbitmq:info lolipop --exposed-ports
dokku rabbitmq:info lolipop --id
dokku rabbitmq:info lolipop --internal-ip
dokku rabbitmq:info lolipop --links
dokku rabbitmq:info lolipop --service-root
dokku rabbitmq:info lolipop --status
dokku rabbitmq:info lolipop --version
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
dokku rabbitmq:logs <service> [-t|--tail]
```

You can tail logs for a particular service:

```shell
dokku rabbitmq:logs lolipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku rabbitmq:logs lolipop --tail
```

### link the rabbitmq service to the app

```shell
# usage
dokku rabbitmq:link <service> <app> [--link-flags...]
```

A rabbitmq service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our 'playground' app.

> NOTE: this will restart your app

```shell
dokku rabbitmq:link lolipop playground
```

The following environment variables will be set automatically by docker (not on the app itself, so they won’t be listed when calling dokku config):

```
DOKKU_RABBITMQ_LOLIPOP_NAME=/lolipop/DATABASE
DOKKU_RABBITMQ_LOLIPOP_PORT=tcp://172.17.0.1:5672
DOKKU_RABBITMQ_LOLIPOP_PORT_5672_TCP=tcp://172.17.0.1:5672
DOKKU_RABBITMQ_LOLIPOP_PORT_5672_TCP_PROTO=tcp
DOKKU_RABBITMQ_LOLIPOP_PORT_5672_TCP_PORT=5672
DOKKU_RABBITMQ_LOLIPOP_PORT_5672_TCP_ADDR=172.17.0.1
```

The following will be set on the linked application by default:

```
RABBITMQ_URL=amqp://lolipop:SOME_PASSWORD@dokku-rabbitmq-lolipop:5672/lolipop
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the 'expose' subcommand. Another service can be linked to your app:

```shell
dokku rabbitmq:link other_service playground
```

It is possible to change the protocol for rabbitmq_url by setting the environment variable rabbitmq_database_scheme on the app. Doing so will after linking will cause the plugin to think the service is not linked, and we advise you to unlink before proceeding.

```shell
dokku config:set playground RABBITMQ_DATABASE_SCHEME=amqp2
dokku rabbitmq:link lolipop playground
```

This will cause rabbitmq_url to be set as:

```
amqp2://lolipop:SOME_PASSWORD@dokku-rabbitmq-lolipop:5672/lolipop
```

### unlink the rabbitmq service from the app

```shell
# usage
dokku rabbitmq:unlink <service> <app>
```

You can unlink a rabbitmq service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku rabbitmq:unlink lolipop playground
```

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### enter or run a command in a running rabbitmq service container

```shell
# usage
dokku rabbitmq:enter <service>
```

A bash prompt can be opened against a running service. Filesystem changes will not be saved to disk.

```shell
dokku rabbitmq:enter lolipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk.

```shell
dokku rabbitmq:enter lolipop touch /tmp/test
```

### expose a rabbitmq service on custom port if provided (random port otherwise)

```shell
# usage
dokku rabbitmq:expose <service> <ports...>
```

Expose the service on the service's normal ports, allowing access to it from the public interface (0. 0. 0. 0):

```shell
dokku rabbitmq:expose lolipop ${PLUGIN_DATASTORE_PORTS[@]}
```

### unexpose a previously exposed rabbitmq service

```shell
# usage
dokku rabbitmq:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (0. 0. 0. 0):

```shell
dokku rabbitmq:unexpose lolipop
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

This will replace rabbitmq_url with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
RABBITMQ_URL=amqp://other_service:ANOTHER_PASSWORD@dokku-rabbitmq-other-service:5672/other_service
DOKKU_RABBITMQ_BLUE_URL=amqp://other_service:ANOTHER_PASSWORD@dokku-rabbitmq-other-service:5672/other_service
DOKKU_RABBITMQ_SILVER_URL=amqp://lolipop:SOME_PASSWORD@dokku-rabbitmq-lolipop:5672/lolipop
```

### start a previously stopped rabbitmq service

```shell
# usage
dokku rabbitmq:start <service>
```

Start the service:

```shell
dokku rabbitmq:start lolipop
```

### stop a running rabbitmq service

```shell
# usage
dokku rabbitmq:stop <service>
```

Stop the service and the running container:

```shell
dokku rabbitmq:stop lolipop
```

### graceful shutdown and restart of the rabbitmq service container

```shell
# usage
dokku rabbitmq:restart <service>
```

Restart the service:

```shell
dokku rabbitmq:restart lolipop
```

### upgrade service <service> to the specified versions

```shell
# usage
dokku rabbitmq:upgrade <service> [--upgrade-flags...]
```

You can upgrade an existing service to a new image or image-version:

```shell
dokku rabbitmq:upgrade lolipop
```

### Service Automation

Service scripting can be executed using the following commands:

### list all rabbitmq service links for a given app

```shell
# usage
dokku rabbitmq:app-links <app>
```

List all rabbitmq services that are linked to the 'playground' app.

```shell
dokku rabbitmq:app-links playground
```

### check if the rabbitmq service exists

```shell
# usage
dokku rabbitmq:exists <service>
```

Here we check if the lolipop rabbitmq service exists.

```shell
dokku rabbitmq:exists lolipop
```

### check if the rabbitmq service is linked to an app

```shell
# usage
dokku rabbitmq:linked <service> <app>
```

Here we check if the lolipop rabbitmq service is linked to the 'playground' app.

```shell
dokku rabbitmq:linked lolipop playground
```

### list all apps linked to the rabbitmq service

```shell
# usage
dokku rabbitmq:links <service>
```

List all apps linked to the 'lolipop' rabbitmq service.

```shell
dokku rabbitmq:links lolipop
```

### Disabling `docker pull` calls

If you wish to disable the `docker pull` calls that the plugin triggers, you may set the `RABBITMQ_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker pull` is disabled.