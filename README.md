Microservice Consul Sample
===================

[Deutsche Anleitung zum Starten des Beispiels](WIE-LAUFEN.md)


This sample is like the sample that you can find at
 https://github.com/ewolff/microservice-consul .

This demo uses [Hashicorp Consul](https://www.consul.io) for service
discovery and Apache httpd as a reverse proxy to route the calls to
the services and as a load balancer. This demo uses Consul as
a DNS server. So the services use a hostname to lookup other
services. [Registrator](https://github.com/gliderlabs/registrator)
automatically registers all Docker
Containers. Therefore the code has no dependencies on Consul.

This project creates a complete microservice demo system in Docker
containers. The services are implemented in Java using Spring and
Spring Cloud.

It uses three microservices:
- `Order` to process orders. (http://localhost:8080 when started locally)
- `Customer` to handle customer data. (http://localhost:8080)
- `Catalog` to handle the items in the catalog. (http://localhost:8080)

Consul
------

Consul has a Web UI. You can access it at port 8500 of your Docker
host. Also the homepage at port 8080 contains a link to the Consul UI

Also you can use Consul's DNS interface with e.g. dig:

```
dig @localhost order.service.consul.
dig @localhost order.service.consul. ANY
dig @localhost order.service.consul. SRV
```

Note that the demo uses the original
[Consul Docker image](https://hub.docker.com/_/consul/) provided by
Hashicorp. However, the demo does not use a Consul cluster and only
stores the data in memory i.e. it is certainly not fit for production.

The Consul DNS interface is mapped to port 53 on the Docker Host. The
Docker containers are configured with to use the IP adress of the
Docker Host as the DNS server.

[Registrator](https://github.com/gliderlabs/registrator) registers all
Docker containers including the Spring Cloud microservices (customer,
catalog and order)

Apache HTTP Load Balancer
------------------------

Apache HTTP is used to provide the web page of the demo at
port 8080. It also forwards HTTP requests to the microservices whose ports
are not exposed! Apache HTTP is configured as a reverse proxy for this - and
as a load balancer i.e. if you start multiple instances of a microservices
e.g. via `docker-compose scale catalog=2`, Apache will recognize the new instance.

To configure this Apache HTTP needs to get all registered services from
Consul. [Consul Template](https://github.com/hashicorp/consul-template)
is used for this. It uses a template for the Apache HTTP
configuration and fills in the IP addresses of the registered services.

The Docker container therefore runs two processes: Apache HTTP and
Consul Template. Consul Template starts Apache httpd and also restarts
Apache httpd when new services are registered in the Consul server.

Please refer to the subdirectory `apache` to see how this works.


Technologies
------------

- Consul for Lookup/ Discovery
- Apache as a reverse proxy to route calls to the appropriate SCS.
- [Hystrix](https://github.com/netflix/hystrix) is used for resilience. See `CatalogClient` in
  `com.ewolff.microservice.order.clients` in the microservice-demo-order
  project . Note that the `CustomerClient` won't use Hystrix. This way
  you can see how a crash of the Customer microservices makes the
  Order microservice useless.


How To Run
----------

The demo can be run with [Docker Machine and Docker
Compose](docker/README.md) via `docker-compose up`

See [How to run](HOW-TO-RUN.md) for details.

Remarks on the Code
-------------------

The servers for the infrastructure components are pretty simple thanks to Spring Cloud:

- microservice-consul-demo-turbine can be used to consolidate the Hystrix metrics and has a Hystrix dashboard.

The microservices are:

- [microservice-consuldns-demo-catalog](microservice-consuldns-demo/microservice-consuldns-demo-catalog) is the application to take care of items.
- [microservice-consuldns-demo-customer](microservice-consuldns-demo/microservice-consuldns-demo-customer) is responsible for customers.
- [microservice-consuldns-demo-order](microservice-consuldns-demo/microservice-consuldns-demo-order) does order processing. It uses microservice-demo-catalog and microservice-demo-customer.


The microservices have a Java main application in `src/test/java` to run them stand alone. `microservice-demo-order` uses a stub for the other services then. Also there are tests that use _consumer-driven contracts_. That is why it is ensured that the services provide the correct interface. These CDC tests are used in microservice-demo-order to verify the stubs. In `microservice-demo-customer` and `microserivce-demo-catalog` they are used to verify the implemented REST services.
