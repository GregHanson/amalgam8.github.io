# Amalgam8 Microservices Fabric

## Overview

Amalgam8 is a multi-tenanted, platform and runtime agnostic, microservices integration framework.
The microservices in an Amalgam8 application can be polyglot, run in containers, VMs, or even on bare metal.

Amalgam8 provides proxy and registration sidecars and other adapter mechanisms that are used for registering and calling
microservices. Depending on a microservice's deployment runtime, integration with Amalgam8 can require little or no significant change
to the microservice impelementation code because Amalgam8 is designed to leverage the benefits of any given runtime
and then provide the easiest possible integration for each. For example, in Kubernetes, a microservice that adready has an associated
Kubernetes service definition can be registered automatically using an Amalgam8 plugin providing
an adapted view of the Kubernetes services in Amalgam8. 
Similar plugins are possible for other runtime environments, but where they are not, Amalgam8 also provides registration
and proxy functionality using sidecars in a number of predefined, but extensible, container images.

In addition to ease of integration, Amalgam8 also provides the integrated microservice-based applications with core functionality
enabling a great deal of control and testability, addressing some of the most challenging issues faced when moving to
a microservices-based system. This includes edge and mid-tier version-routing based on traffic percentage, user, and other
criteria. Delays and failures can also be injected into the path of calls to and between microservices, enabling advanced
end-to-end resiliency testing. And most importantly, all of this is designed for extensibility and is completely available in open source.

## How it Works

![high-level architecture](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/architecture.jpg)

At the heart of Amalgam8, are two mutli-tenanted services:

1. **Registry** - A high performance service registry that provides a centralized view of all the microservices in an application, regardless
   of where they are actually running.
2. **Controller** - Monitors the Registry and provides a REST API for registering routing and other microservice control-rules, which
   it uses to generate and send control information to proxy servers running within the application.

Application run as tenants of these two servers. They register their services in the Registry and use the Controller to manage proxies,
usually running in sidecars of the microservices.

![how it works](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/how-it-works.jpg)

1. Microservice instances are registered in the A8 Registry. There are several ways this may be accomplished (see below).
2. Administrator specifies routing rules and filters (e.g., version rules, test delays) to control traffic flow between microservices.
3. A8 Controller monitors the A8 Registry and administrator input and then generates control information that is sent to the A8 Proxies.
4. Requests to microservices are via an A8 Proxy (usually a client-side sidecar of another microservice)
5. A8 Proxy forwards request to an approriate microservice, depending on the request path and headers and the configuration specified by the controller.

## Tenant Library

Almagam8 includes a library containing a very flexible sidecar architecture that can be configured and used by tenants in a number of ways.

![sidecar architecture](https://github.com/amalgam8/amalgam8.github.io/blob/master/images/sidecar.jpg)

* **Service Registration** - Registers and heartbeats a microservice instance in the Amalgam8 registry.
* **Route Management** - Communicates with the A8 Controller in the control plane to manage configuration and control-rules of an A8 Proxy.
* **Supervisor** - Manages the lifecycle of an A8 Proxy server and optionally an application microservice itself.
* **Proxy** - An [Nginx OpenResty](https://openresty.org/en/) server implementing the A8 Proxy function.

Depending on the application, some or all of these components may be used. 
For example, a microservice that requires registration, makes outgoing calls, and app supervision might use all of them
in a single container as shown in the diagram. In Kubernetes, however, the app supervision would not be used. The app would instead
be likely running its own separate container in a pod. For leaf microservices (i.e., ones that make no outgoing calls), the proxy and
route management components would not be needed. Service Registration would not needed for services in a runtime that 
supports auto registration (e.g., amalgam8 registry adapter for kubernetes).

Since most microservices run in some kind of container environment, Amalgam8 provides convenient Docker images that can be used
as microservice application base images or simply to run access one or more of the services, the same components can also be packaged
in different ways, depending on the scenario (e.g., installation packages (deb, rpm, etc.), or plain executables.
