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
      - port: 8080
forwards:
  - service: frontend
    ports:
      - port: 9090
        sourcePort: 8080
      - port: 9091
        sourcePort: 8081
~~~

## Proposed YAML with defaults

~~~ yaml
bindings:
  - service: backend
    ports:
      - port: 8080
        targetHost: localhost
        targetPort: 8080
forwards:
  - service: frontend
    ports:
      - port: 9090
        sourceHost: localhost
        sourcePort: 8080
      - port: 9091
        sourcehost: localhost
        sourcePort: 8081
~~~

## Proposed YAML schema

~~~ yaml
bindings: <list of bindings> (optional)

forwards: <list of bindings> (optional)

<binding>:
  service: <string> (required)
  ports: <list of port mappings> (optional)

<binding port mapping>:
  port: <service port> (required)
  targetHost: <string> (optional)
  targetPort: <string or int> (optional)

<forward port mapping>:
  port: <service port> (required)
  soruceHost: <string> (optional)
  sourcePort: <string or int> (optional)
~~~
