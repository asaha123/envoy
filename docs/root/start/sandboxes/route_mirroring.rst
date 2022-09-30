.. _install_sandboxes_request_mirroring_policies:

Request mirroring policies
==========================

.. sidebar:: Requirements

   .. include:: _include/docker-env-setup-link.rst

This simple example demonstrates Envoy's request mirroring capability using 
`request mirror policies <https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-routeaction-requestmirrorpolicy>`__.

Incoming requests are received by ``front-envoy`` Envoy proxy. 

The ``front-envoy.yaml`` envoy configuration configures two routes::

        routes:
          - match:
              prefix: "/service/1"
            route:
              cluster: service1
              request_mirror_policies:
                cluster: "service1-mirror"
          - match:
              prefix: "/service/2"
            route:
              cluster: service2
              request_mirror_policies:
                cluster_header: "x-mirror-cluster"

A request for the path ``/service/1`` is forwarded to the ``service1`` cluster.
In addition, the request is also forwarded to the ``service1-mirror`` cluster.

A request for the path ``/service/2`` is forwarded to the ``service2`` cluster.
If a header ``x-mirror-cluster`` is specified in the request, envoy extracts the
header value and forwards the request to a cluster with the same name, if found.
For example, if we send a header with a request as ``x-mirror-cluster: service2-mirror``,
the request will be forwarded to the ``service2-mirror`` cluster.

Step 1: Build the sandbox
*************************

Change to the ``examples/route-mirroring`` directory.

Terminal 1 ::

    $ pwd
    envoy/examples/route-mirroring
    $ docker-compose build
    $ docker-compose up 


Terminal 2 ::

    $ pwd
    envoy/examples/route-mirroring
    $ docker-compose ps

    NAME                                COMMAND                  SERVICE             STATUS              PORTS
    ---------------------------------------------------------------------------------------------------------------------------

    route-mirroring-front-envoy-1       "/docker-entrypoint.…"   front-envoy         running             0.0.0.0:8001->8001/tcp,
                                                                                                         :::8001->8001/tcp,
                                                                                                         0.0.0.0:8080->8080/tcp,
                                                                                                         :::8080->8080/tcp,
                                                                                                         0.0.0.0:8443->8443/tcp,
                                                                                                         :::8443->8443/tcp,
                                                                                                         10000/tcp
    route-mirroring-service1-1          "/usr/local/bin/star…"   service1            running (healthy)
    route-mirroring-service1-mirror-1   "/usr/local/bin/star…"   service1-mirror     running (healthy)
    route-mirroring-service2-1          "/usr/local/bin/star…"   service2            running (healthy)
    route-mirroring-service2-mirror-1   "/usr/local/bin/star…"   service2-mirror     running (healthy)

Step 2: Demonstrate static mirror cluster name
**********************************************

Terminal 2 ::

  $ pwd
  envoy/examples/request-mirroring
  $ curl localhost:8080/service/1

The command above sends a request to the `front-envoy` service which forwards the request to `service1`
and also sends the request to the service 1 mirror, `service1-mirror`.


Step 3: See Logs
****************

Terminal 1 ::

   ...
   route-mirroring-service1-1         | 127.0.0.1 - - [30/Sep/2022 00:41:27] "GET /service/1 HTTP/1.1" 200 -
   route-mirroring-service1-mirror-1  | 127.0.0.1 - - [30/Sep/2022 00:41:27] "GET /service/1 HTTP/1.1" 200 -
   

Both the `service1` and `service1-mirror` services got the request.


Step 4: Demonstrate mirror cluster via header
*********************************************

In this step, we will see a demonstration where the request specifies via a header, `x-mirror-cluster`,
the cluster that `front-envoy` will mirror the request to.

Terminal 2 ::

  $ pwd
  envoy/examples/request-mirroring
  $ curl --header "x-mirror-cluster: service2-mirror" localhost:8080/service/2

The command above sends a request to the `front-envoy` service which forwards the request to `service2`
and also mirrors the request to the cluster named, `service2-mirror`


Step 4: See Logs
****************

Terminal 1 ::


  ...
  route-mirroring-service2-1         | 127.0.0.1 - - [30/Sep/2022 00:46:05] "GET /service/2 HTTP/1.1" 200 -
  route-mirroring-service2-mirror-1  | 127.0.0.1 - - [30/Sep/2022 00:46:05] "GET /service/2 HTTP/1.1" 200 -
  ...

