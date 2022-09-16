# Gateway bind and forward

This applies the [port option pattern](ports.md).

## Proposed bind interface and help

~~~
skupper gateway bind <service-name> [--host <host>] [--port [<target-port>:]<service-port>]]

# Route traffic from service backend port 8080 to localhost:9090.
skupper gateway bind backend --port 9090:8080

# Route traffic from service backend ports 8080 and 8081 to localhost:8080 and localhost:8081.
skupper service create backend 8080 8081
skupper gateway bind backend

# Route traffic from service backend port 8080 to localhost:8080.
skupper gateway bind backend --port 8080

# Route traffic from service backend ports to corresponding ports on local interface othernet.
skupper gateway bind backend --host othernet
~~~

## Proposed forward interface and help

~~~
skupper gateway forward <service-name> [--host <interface>] [--port [<source-port>:]<service-port>]]

# Listen on localhost:9091.  Forward traffic to service frontend, port 8080.
skupper gateway forward frontend --port 9091:8080

# Listen on othernet:9091.  Forward traffic to service frontend, port 8080.
skupper gateway forward frontend --host othernet --port 9091:8080

# Listen on localhost:8080 and localhost:8081.  Forward traffic to the frontend service, ports 8080 and 8081.
skupper service create frontend 8080 8081
skupper gateway forward frontend
~~~

## Proposed export interface and help

The port is required in this instance, but host is still optional.

~~~
skupper gateway expose <service-name> [<target-port>:]<service-port>... [--host <host>]

# Route traffic from service backend port 8080 to localhost:9090.
skupper gateway expose backend 9090:8080

# Route traffic from service backend ports 8080 and 8081 to localhost:8080 and localhost:8081.
skupper gateway expose backend 8080 8081

# Route traffic from service backend port 8080 to othernet:8080.
skupper gateway expose backend 8080 --host othernet
~~~
