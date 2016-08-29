# dokku rabbitmq (beta) [![Build Status](https://img.shields.io/travis/dokku/dokku-rabbitmq.svg?branch=master "Build Status")](https://travis-ci.org/dokku/dokku-rabbitmq) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official rabbitmq plugin for dokku. Currently defaults to installing [rabbitmq 3.6.5-management](https://hub.docker.com/_/rabbitmq/).

## requirements

- dokku 0.4.x+
- docker 1.8.x

## installation

```shell
# on 0.4.x+
sudo dokku plugin:install https://github.com/dokku/dokku-rabbitmq.git rabbitmq
```

## commands

```
rabbitmq:clone <name> <new-name>  NOT IMPLEMENTED
rabbitmq:connect <name>           NOT IMPLEMENTED
rabbitmq:create <name>            Create a rabbitmq service with environment variables
rabbitmq:destroy <name>           Delete the service and stop its container if there are no links left
rabbitmq:export <name> > <file>   NOT IMPLEMENTED
rabbitmq:expose <name> [port]     Expose a rabbitmq service on custom port if provided (random port otherwise)
rabbitmq:import <name> <file>     NOT IMPLEMENTED
rabbitmq:info <name>              Print the connection information
rabbitmq:link <name> <app>        Link the rabbitmq service to the app
rabbitmq:list                     List all rabbitmq services
rabbitmq:logs <name> [-t]         Print the most recent log(s) for this service
rabbitmq:promote <name> <app>     Promote service <name> as RABBITMQ_URL in <app>
rabbitmq:restart <name>           Graceful shutdown and restart of the rabbitmq service container
rabbitmq:start <name>             Start a previously stopped rabbitmq service
rabbitmq:stop <name>              Stop a running rabbitmq service
rabbitmq:unexpose <name>          Unexpose a previously exposed rabbitmq service
rabbitmq:unlink <name> <app>      Unlink the rabbitmq service from the app
```

## usage

```shell
# create a rabbitmq service named lolipop
dokku rabbitmq:create lolipop

# you can also specify the image and image
# version to use for the service
# it *must* be compatible with the
# official rabbitmq image
export RABBITMQ_IMAGE="rabbitmq"
export RABBITMQ_IMAGE_VERSION="3.5"
dokku rabbitmq:create lolipop

# you can also specify custom environment
# variables to start the rabbitmq service
# in semi-colon separated forma
export RABBITMQ_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku rabbitmq:create lolipop

# get connection information as follows
dokku rabbitmq:info lolipop

# you can also retrieve a specific piece of service info via flags
dokku rabbitmq:info lolipop --config-dir
dokku rabbitmq:info lolipop --data-dir
dokku rabbitmq:info lolipop --dsn
dokku rabbitmq:info lolipop --exposed-ports
dokku rabbitmq:info lolipop --id
dokku rabbitmq:info lolipop --internal-ip
dokku rabbitmq:info lolipop --links
dokku rabbitmq:info lolipop --status
dokku rabbitmq:info lolipop --version

# a rabbitmq service can be linked to a
# container this will use native docker
# links via the docker-options plugin
# here we link it to our 'playground' app
# NOTE: this will restart your app
dokku rabbitmq:link lolipop playground

# the following environment variables will be set automatically by docker (not
# on the app itself, so they won’t be listed when calling dokku config)
#
#   DOKKU_RABBITMQ_LOLIPOP_NAME=/lolipop/DATABASE
#   DOKKU_RABBITMQ_LOLIPOP_PORT=tcp://172.17.0.1:5672
#   DOKKU_RABBITMQ_LOLIPOP_PORT_5672_TCP=tcp://172.17.0.1:5672
#   DOKKU_RABBITMQ_LOLIPOP_PORT_5672_TCP_PROTO=tcp
#   DOKKU_RABBITMQ_LOLIPOP_PORT_5672_TCP_PORT=5672
#   DOKKU_RABBITMQ_LOLIPOP_PORT_5672_TCP_ADDR=172.17.0.1
#
# and the following will be set on the linked application by default
#
#   RABBITMQ_URL=amqp://lolipop:SOME_PASSWORD@dokku-rabbitmq-lolipop:5672/lolipop
#
# NOTE: the host exposed here only works internally in docker containers. If
# you want your container to be reachable from outside, you should use `expose`.

# another service can be linked to your app
dokku rabbitmq:link other_service playground

# since DATABASE_URL is already in use, another environment variable will be
# generated automatically
#
#   DOKKU_RABBITMQ_BLUE_URL=amqp://other_service:ANOTHER_PASSWORD@dokku-rabbitmq-other-service:5672/other_service

# you can then promote the new service to be the primary one
# NOTE: this will restart your app
dokku rabbitmq:promote other_service playground

# this will replace RABBITMQ_URL with the url from other_service and generate
# another environment variable to hold the previous value if necessary.
# you could end up with the following for example:
#
#   RABBITMQ_URL=amqp://other_service:ANOTHER_PASSWORD@dokku-rabbitmq-other-service:5672/other_service
#   DOKKU_RABBITMQ_BLUE_URL=amqp://other_service:ANOTHER_PASSWORD@dokku-rabbitmq-other-service:5672/other_service
#   DOKKU_RABBITMQ_SILVER_URL=amqp://lolipop:SOME_PASSWORD@dokku-rabbitmq-lolipop:5672/lolipop

# you can also unlink a rabbitmq service
# NOTE: this will restart your app and unset related environment variables

# you can tail logs for a particular service
dokku rabbitmq:logs lolipop
dokku rabbitmq:logs lolipop -t # to tail

# finally, you can destroy the container
dokku rabbitmq:destroy lolipop
```

## Changing database adapter

It's possible to change the protocol for DATABASE_URL by setting
the environment variable RABBITMQ_DATABASE_SCHEME on the app:

```
dokku config:set playground RABBITMQ_DATABASE_SCHEME=amqp2
dokku rabbitmq:link lolipop playground
```

Will cause RABBITMQ_URL to be set as
amqp2://dokku-rabbitmq-other-service:5672/0

CAUTION: Changing RABBITMQ_DATABASE_SCHEME after linking will cause dokku to
believe the rabbitmq is not linked when attempting to use `dokku rabbitmq:unlink`
or `dokku rabbitmq:promote`.
You should be able to fix this by

- Changing RABBITMQ_URL manually to the new value.

OR

- Set RABBITMQ_DATABASE_SCHEME back to its original setting
- Unlink the service
- Change RABBITMQ_DATABASE_SCHEME to the desired setting
- Relink the service

## todo

- implement rabbitmq:clone
- implement rabbitmq:import
