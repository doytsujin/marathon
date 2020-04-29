@@@ index

  * [Getting Started](getting-started/index.md)
  * [Using Marathon](using-marathon/index.md)

@@@

<div class="jumbotron text-center">
  <h1>Marathon</h1>
  <p class="lead">
    A container orchestration platform for Mesos and DC/OS
  </p>
  <p>

    <a href="https://downloads.mesosphere.io/marathon/builds/1.8.222-86475ddac/marathon-1.8.222-86475ddac.tgz"
        class="btn btn-lg btn-primary">
      Download Marathon v1.8.222
    </a>
  </p>
  <a class="btn btn-link"
      href="https://downloads.mesosphere.io/marathon/builds/1.8.222-86475ddac/marathon-1.8.222-86475ddac.tgz.sha1">
    v1.8.222 SHA Checksum
  </a> &middot;
  <a class="btn btn-link"
      href="https://github.com/mesosphere/marathon/releases/tag/v1.8.222">
    v1.8.222 Release Notes
  </a>
</div>

# Introduction

## Overview

Marathon is a production-grade container orchestration platform for Mesosphere's [Datacenter Operating System (DC/OS)](https://dcos.io) and [Apache Mesos](https://mesos.apache.org/).

## Features

- [High Availability](https://mesosphere.github.io/marathon/docs/high-availability.html). Marathon runs as an active/passive cluster with leader election for 100% uptime.
- Multiple container runtimes. Marathon has first-class support for both [Mesos containers](https://mesosphere.github.io/marathon/docs/application-basics.html) (using cgroups) and [Docker](https://mesosphere.github.io/marathon/docs/native-docker.html).
- [Stateful apps](https://mesosphere.github.io/marathon/docs/persistent-volumes.html). Marathon can bind persistent storage volumes to your application. You can run databases like MySQL and Postgres, and have storage accounted for by Mesos.
- [Beautiful and powerful UI](https://mesosphere.github.io/marathon/docs/marathon-ui.html).
- [Constraints](https://mesosphere.github.io/marathon/docs/constraints.html). These allow to e.g. place only one instance of an application per rack, node, etc.
- [Service Discovery & Load Balancing](https://mesosphere.github.io/marathon/docs/service-discovery-load-balancing.html). Several methods available.
- [Health Checks](https://mesosphere.github.io/marathon/docs/health-checks.html). Evaluate your application's health using HTTP or TCP checks.
- [Event Subscription](https://mesosphere.github.io/marathon/docs/event-bus.html#subscription-to-events-via-the-event-stream). Supply an HTTP endpoint to receive notifications - for example to integrate with an external load balancer.
- [Metrics](https://mesosphere.github.io/marathon/docs/metrics.html). Query them at `/metrics` in JSON format, push them to systems like Graphite, StatsD and DataDog, or scrape them using Prometheus.
- [Deprecated Metrics](https://mesosphere.github.io/marathon/docs/deprecated-metrics.html). Query them at `/metrics` in JSON format, or push them to systems like Graphite, StatsD and DataDog.
- [Complete REST API](https://mesosphere.github.io/marathon/api-console/index.html) for easy integration and scriptability.

## DC/OS features

Running on [DC/OS](https://dcos.io/), Marathon gains the following additional features:

- Virtual IP routing. Allocate a dedicated, virtual address to your app. Your app is now reachable anywhere in the cluster, wherever it might be scheduled. Load balancing and rerouting around failures are done automatically.
- Authorization (DC/OS Enterprise Edition only). True multitenancy with each user or group having access to their own applications and groups.

## Examples

### Marathon orchestrates both apps and frameworks

The graphic below shows how Marathon runs on <a href="https://mesos.apache.org/">Apache Mesos</a> acting as the orchestrator for other applications and services.

Marathon is the first framework to be launched, running directly alongside Mesos. This means the Marathon scheduler processes are started directly using `init`, `upstart`, or a similar tool.

Marathon is a powerful way to run other Mesos frameworks: in this case, [Chronos](https://github.com/mesos/chronos). Marathon launches two instances of the Chronos scheduler using the Docker image `mesosphere/chronos`. The Chronos instances appear in orange on the top row.

If either of the two Chronos containers fails for any reason, then Marathon will restart them on another agent. This approach ensures that two Chronos processes are always running.

Since Chronos itself is a framework and receives resource offers, it can start tasks on Mesos.
In the use case below, Chronos is running two scheduled jobs, shown in blue. One dumps a production MySQL database to S3, while another sends an email newsletter to all customers via Rake.

Meanwhile, Marathon also runs the other application containers - either Docker or Mesos - that make up our website: JBoss servers, Jetty, Sinatra, Rails, and so on.

We have shown that Marathon is responsible for running other frameworks, helps them maintain 100% uptime, and coexists with them creating workloads in Mesos.

![architecture](.../architecture.png)

### Scaling and fault recovery

The next three images illustrate scaling and container placement.

Below we see Marathon running three applications, each scaled to a different number of containers: Search (1), Jetty (3), and Rails (5).

![marathon1](.../marathon1.png)

As the website gains traction, we decide to scale out the Search service and our Rails-based application.

We use the Marathon REST API call to to add more instances. Marathon will take care of placing the new containers on machines with spare capacity, honoring the constraints we previously set. We can see the containers are dynamically placed:

![marathon2](.../marathon2.png)

Finally, imagine that one of the datacenter workers trips over a power cord and a server is unplugged. No problem for Marathon: it moves the affected Search and Rails containers to a node that has spare capacity. Marathon has maintained our uptime in the face of machine failure.

![marathon3](.../marathon3.png)