# dokku rabbitmq (beta)

Official rabbitmq plugin for dokku. Currently installs rabbitmq 3.5.4-management.

## requirements

- dokku 0.3.25+
- docker 1.6.x

## installation

```
cd /var/lib/dokku/plugins
git clone https://github.com/dokku/dokku-rabbitmq.git rabbitmq
dokku plugins-install-dependencies
dokku plugins-install
```

## commands

```
rabbitmq:alias <name> <alias>     Set an alias for the docker link
rabbitmq:clone <name> <new-name>  NOT IMPLEMENTED
rabbitmq:connect <name>           NOT IMPLEMENTED
rabbitmq:create <name>            Create a rabbitmq service
rabbitmq:destroy <name>           Delete the service and stop its container if there are no links left
rabbitmq:export <name>            NOT IMPLEMENTED
rabbitmq:expose <name> [port]     Expose a rabbitmq service on custom port if provided (random port otherwise)
rabbitmq:import <name> <file>     NOT IMPLEMENTED
rabbitmq:info <name>              Print the connection information
rabbitmq:link <name> <app>        Link the rabbitmq service to the app
rabbitmq:list                     List all rabbitmq services
rabbitmq:logs <name> [-t]         Print the most recent log(s) for this service
rabbitmq:restart <name>           Graceful shutdown and restart of the rabbitmq service container
rabbitmq:start <name>             Start a previously stopped rabbitmq service
rabbitmq:stop <name>              Stop a running rabbitmq service
rabbitmq:unexpose <name>          Unexpose a previously exposed rabbitmq service
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

# get connection information as follows
dokku rabbitmq:info lolipop

# lets assume the ip of our rabbitmq service is 172.17.0.1

# a rabbitmq service can be linked to a
# container this will use native docker
# links via the docker-options plugin
# here we link it to our 'playground' app
# NOTE: this will restart your app
dokku rabbitmq:link lolipop playground

# the above will expose the following environment variables
#
#   RABBITMQ_URL=rabbitmq://172.17.0.1:5672
#   RABBITMQ_NAME=/lolipop/DATABASE
#   RABBITMQ_PORT=tcp://172.17.0.1:5672
#   RABBITMQ_PORT_5672_TCP=tcp://172.17.0.1:5672
#   RABBITMQ_PORT_5672_TCP_PROTO=tcp
#   RABBITMQ_PORT_5672_TCP_PORT=5672
#   RABBITMQ_PORT_5672_TCP_ADDR=172.17.0.1

# you can customize the environment
# variables through a custom docker link alias
dokku rabbitmq:alias lolipop RABBITMQ_DATABASE

# you can also unlink a rabbitmq service
# NOTE: this will restart your app
dokku rabbitmq:unlink lolipop playground

# you can tail logs for a particular service
dokku rabbitmq:logs lolipop
dokku rabbitmq:logs lolipop -t # to tail

# finally, you can destroy the container
dokku rabbitmq:destroy lolipop
```

## todo

- implement rabbitmq:clone
- implement rabbitmq:import
