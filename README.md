# dokku rabbitmq (beta) [![Build Status](https://img.shields.io/travis/dokku/dokku-rabbitmq.svg?branch=master "Build Status")](https://travis-ci.org/dokku/dokku-rabbitmq) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official rabbitmq plugin for dokku. Currently defaults to installing [rabbitmq 3.5.4-management](https://hub.docker.com/_/rabbitmq/).

## requirements

- dokku 0.4.0+
- docker 1.8.x

## installation

```shell
cd /var/lib/dokku/plugins
git clone https://github.com/dokku/dokku-rabbitmq.git rabbitmq
dokku plugins-install

# on 0.4.x
dokku plugin:install https://github.com/dokku/dokku-rabbitmq.git rabbitmq
```

## commands

```
rabbitmq:create <name>, Create a RabbitMQ service
rabbitmq:create-user <name> <username> <password>, Create a RabbitMQ user
rabbitmq:delete-user <name> <username>, Delete a RabbitMQ user
rabbitmq:add-vhost <name> <vhost>, Create a RabbitMQ vhost
rabbitmq:delete-vhost <name> <vhost>, Delete a RabbitMQ vhost
rabbitmq:set-permissions <name> <vhost> <username> <conf> <write> <read>, Sets a RabbitMQ permissions for vhost
rabbitmq:destroy <name>, Delete the RabbitMQ service and stop its container if there are no links left
rabbitmq:link <name> <app>, Link the RabbitMQ service to the app
rabbitmq:unlink <name> <app>, Unlink the RabbitMQ service from the app
rabbitmq:export <name>, NOT IMPLEMENTED
rabbitmq:import <name> <file>, NOT IMPLEMENTED
rabbitmq:connect <name>, NOT IMPLEMENTED
rabbitmq:logs <name> [-t], Print the most recent log(s) for this service
rabbitmq:restart <name>, Graceful shutdown and restart of the RabbitMQ service container
rabbitmq:info <name>, Print the connection information
rabbitmq:list, List all RabbitMQ services
rabbitmq:clone <name> <new-name>, NOT IMPLEMENTED
rabbitmq:expose <name> [port], Expose a RabbitMQ service on custom port if provided (random port otherwise)
rabbitmq:unexpose <name>, Unexpose a previously exposed RabbitMQ service
rabbitmq:start <name>, Start a previously stopped RabbitMQ service
rabbitmq:stop <name>, Stop a running RabbitMQ service
rabbitmq:promote <name> <app>, Promote service <name> as RABBITMQ_URL in <app>
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

# you can also specify custom environment
# variables to start the rabbitmq service
# in semi-colon separated forma
export RABBITMQ_CUSTOM_ENV="USER=alpha;HOST=beta"

# create a rabbitmq service
dokku rabbitmq:create lolipop

# get connection information as follows
dokku rabbitmq:info lolipop

# create user
dokku rabbitmq:create-user lolipop username

# delete user
dokku rabbitmq:delete-user lolipop username

# add vhost
dokku rabbitmq:add-vhost lolipop vhost

# delete vhost
dokku rabbitmq:delete-vhost lolipop vhost

# set permissions
# * need to be escaped
rabbitmq:set-permissions lolipop vhost username ".\*" ".\*" ".\*"
 
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

## todo

- implement rabbitmq:clone
- implement rabbitmq:import
