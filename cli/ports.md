# Ports in the CLI

## Rationale

Kube ports, and Skupper's modeling of ports as well, allow multiple
ports associated with one service (and therefore with one DNS name).

When a command works on the basis of an existing service, it is able
to discover the ports of that service and use them.  (This can also
apply when you haven an existing deployment, which sometimes has an
embedded containerPort.)

Since for some of Skupper's commands these port specifications become
optional, we should use the option syntax for them.

It's best to avoid an ordinal mapping of source ports to target ports
because it creates a dependency on that specific ordering of ports in
the service definition.  If someone inserts a port, any dependent code
is broken as a result.  For this reason, we should use an explicit
mapping.

## General pattern for port options

~~~
skupper <command> --port [<source-port>:]<target-port>
~~~

Apply to commands where specifying the port is optional:

~~~
skupper service bind <service-name> <target-type>/<target-name> \
    [--port [<service-port>:]<target-port>]

skupper expose <target-type>/<target-name> [--port [<service-port>:]<target-port>]

skupper gateway bind <service-name> [--port [<local-port>:]<service-port>]]

skupper gateway forward <service-name> [--port [<local-port>:]<service-port>]]
~~~

The `--port` option is in general one you can repeat to specify multiple ports.

Use subcommand-specific names in the `--port` option spec to increase
clarity.

This pattern does not apply to commands where the ports are required,
as for instance with `skupper service create`:

~~~
skupper service create <name> <port> [<port>...]
~~~
