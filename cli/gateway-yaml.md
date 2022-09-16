# Gateway YAML

## Scenario

~~~
skupper service create backend 8080
skupper service create frontend 8080 8081
skupper gateway init
skupper gateway bind backend localhost 8080
skupper gateway forward frontend 9090 9091
skupper gateway export-config abc .
~~~

## Current YAML

~~~ yaml
name: abc
qdr-listeners:
    - name: amqp
      host: localhost
      port: 5672
bindings:
    - name: egress-backend:8080
      host: localhost
      service:
        address: backend:8080
        protocol: tcp
        ports:
            - 8080
        enabletls: false
        tlscredentials: ""
        publishnotreadyaddresses: false
      target_ports:
        - 8080
forwards:
    - name: ingress-frontend:8081
      service:
        address: frontend:8081
        protocol: tcp
        ports:
            - 9091
        enabletls: false
        tlscredentials: ""
        publishnotreadyaddresses: false
      target_ports:
        - 9091
    - name: ingress-frontend:8080
      service:
        address: frontend:8080
        protocol: tcp
        ports:
            - 9090
        enabletls: false
        tlscredentials: ""
        publishnotreadyaddresses: false
      target_ports:
        - 9090
~~~

## Proposed YAML

~~~ yaml
bindings:
  - service: backend
    ports:
      - servicePort: 8080
forwards:
  - service: frontend
    ports:
      - servicePort: 8080
        gatewayPort: 9090
      - servicePort: 8081
        gatewayPort: 9091
~~~

## Proposed YAML with defaults

~~~ yaml
bindings:
  - service: backend
    hosts:
      - localhost
    ports:
      - servicePort: 8080
forwards:
  - service: frontend
    hosts:
      - localhost
    ports:
      - servicePort: 8080
        gatewayPort: 9090
      - servicePort: 8081
        gatewayPort: 9091
~~~

## Proposed YAML schema

~~~ yaml
bindings: <list of bindings> (optional)

forwards: <list of bindings> (optional)

<binding>:
  service: <string> (required)
  hosts: <list of string hosts> (optional)
  ports: <list of port mappings> (optional)

<port mapping>:
  servicePort: <port> (required)
  gatewayPort: <port> (optional)
~~~
