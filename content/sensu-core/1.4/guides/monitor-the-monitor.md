---
title: "Monitoring Sensu with Sensu"
description: "Strategies and best practices for monitoring Sensu with Sensu"
version: "1.4"
weight: 10
next: ../securing-rabbitmq
menu:
  sensu-core-1.4:
    parent: guides
---

_NOTE: In order to completely monitor a Sensu stack (Sensu services, Redis, RabbitMQ), you will need to have at least one other independent Sensu stack to do so. This is due to a single Sensu stack can not monitor itself completely, as if some components are down, Sensu will not be able to alert on itself properly_

In this guide, we'll walk you through the best practices and strategies for monitoring Sensu with Sensu. By the end of the guide, you should have a thorough understanding of what is required to ensure your Sensu components are properly monitored, including:

* How to monitor your Sensu server and Sensu API instance(s)
* How to monitor your Uchiwa instance(s)
* How to monitor your RabbitMQ instance(s)
* How to monitor your Redis instance(s)

## Objectives

We'll cover the following in this guide:

* [Monitoring Sensu](#monitoring-sensu)
  * [Monitoring Sensu Server](#monitoring-sensu-server)
  * [Monitoring Sensu API](#monitoring-sensu-api)
  * [Monitoring Uchiwa](#monitoring-sensu-uchiwa)
* [Monitoring RabbitMQ](#monitoring-RabbitMQ)
* [Monitoring Redis](#monitoring-redis)

## Monitoring Sensu

### Monitoring Sensu Server{#monitoring-sensu-server}

The host running the `sensu-server` service should be monitored in two ways:

* Locally by a `sensu-client` process for Operating System checks/metrics and Sensu services like Uchiwa that are not part of the Sensu event system.
* Remotely from an entirely independent Sensu stack to monitor pieces of the Sensu stack that if down, will make Sensu unable to create events. 

#### Monitoring Sensu Server Locally

Monitoring the host that the `sensu-server` process runs on should be done just like any other node in your infrastructure. This includes, but not limited to, checks and metrics for [CPU][1], [memory][2], [disk][3], and [networking][4]. You can find more plugins at the [Sensu Community Homepage][5].

#### Monitoring Sensu Server Remotely

To monitor the `sensu-server` Server process, you will need another independent Sensu stack that can reach in. This can be done by reaching out to Sensu's [API health endpoint][6] and using the [check-http sensu plugin][7].

{{< highlight json >}}
{
  "checks": {
    "check_sensu_server_port": {
      "command": "check-http.rb -h remote.api.hostname -P 4567 -p /health?consumers=1 --response-code 204",
      "subscribers": [
        "monitor_remote_sensu_api"
      ],
      "interval": 60
    }
  }
}
{{< /highlight >}}

### Monitoring Sensu API{#monitoring-sensu-api}

To monitor the `sensu-api` Server process, you will need another independent Sensu stack that can reach in. This can be done by reach out to the port that the API is listening on using the [check-port sensu plugin][8]

Sensu API needs to be monitored remotely via port checks or API health endpoints as events were not handled in testing.

{{< highlight json >}}
{
  "checks": {
    "check_sensu_api_port": {
      "command": "check-ports.rb -H remote.sensu.server.hostname -p 4567",
      "subscribers": [
        "monitor_remote_sensu_api"
      ],
      "interval": 60
    }
  }
}
{{< /highlight >}}

### Monitoring Uchiwa{#monitoring-sensu-uchiwa}

Examples:

Needs testing
{{< highlight json >}}
{
  "checks": {
    "check_uchiwa_process": {
      "command": "check-process.rb -p uchiwa",
      "subscribers": [
        "uchiwa"
      ],
      "interval": 60
    }
  }
}
{{< /highlight >}}

Add for more information on this plugin, visit: https://github.com/sensu-plugins/sensu-plugins-process-checks

AND/OR?

Needs testing and note about port assumption maybe?
Context about or proxy check? Proxy check would clash with already defined host through real sensu client.
{{< highlight json >}}
{
  "checks": {
    "check_uchiwa_port": {
      "command": "check-ports.rb -H remote.uchiwa.server.hostname -p 3000",
      "subscribers": [
        "monitor_remote_uchiwa_port"
      ],
      "interval": 60
    }
  }
}
{{< /highlight >}}

Add for more information on this plugin, visit: https://github.com/sensu-plugins/sensu-plugins-network-checks

## Monitoring RabbitMQ

Use RabbitMQ alive check for remote monitoring from other Sensu stack

Example:
TODO: needs testing
Call out redaction/token substituion to hide password?
{{< highlight json >}}
{
  "checks": {
    "check_rabbitmq_alive": {
      "command": "check-rabbitmq-alive.rb -w remote.rabbitmq.host -v remote.rabbitmq.vhost -u remote.rabbitmq.username -p remote.rabbitmq.password -P remote.rabbitmq.listen.port",
      "subscribers": [
        "monitor_remote_rabbitmq"
      ],
      "interval": 60
    }
  }
}
{{< /highlight >}}

Add for more information on this plugin, visit: https://github.com/sensu-plugins/sensu-plugins-rabbitmq/blob/master/bin/check-rabbitmq-alive.rb

Other RabbitMQ maybes

Examples:

https://github.com/sensu-plugins/sensu-plugins-rabbitmq/blob/master/bin/metrics-rabbitmq-queue.rb

{{< highlight json >}}
{
  "checks": {
    "check_rabbitmq_queue": {
      "command": "TODO",
      "subscribers": [
        "rabbitmq"
      ],
      "interval": 60
    }
  }
}
{{< /highlight >}}

https://github.com/sensu-plugins/sensu-plugins-rabbitmq/blob/master/bin/check-rabbitmq-queue.rb

{{< highlight json >}}
{
  "checks": {
    "collect_rabbitmq_queue": {
      "command": "TODO",
      "subscribers": [
        "rabbitmq"
      ],
      "interval": 60,
      "type": "metric"
    }
  }
}
{{< /highlight >}}

## Monitoring Redis

Use Redis ping check for remote monitoring from another Sensu stack

https://github.com/sensu-plugins/sensu-plugins-redis/blob/master/bin/check-redis-ping.rb
maybe https://github.com/sensu-plugins/sensu-plugins-redis/blob/master/bin/metrics-redis-graphite.rb

Example:
TODO: test the checks
note about redis password assumption
{{< highlight json >}}
{
  "checks": {
    "redis_ping": {
      "command": "check-redis-ping.rb -h remote.redis.hostname -P redis.password",
      "subscribers": [
        "redis"
      ],
      "interval": 60
    }
  }
}
{{< /highlight >}}

Add for more information on this plugin, visit: https://github.com/sensu-plugins/sensu-plugins-rabbitmq/blob/master/bin/check-rabbitmq-alive.rb

{{< highlight json >}}
{
  "checks": {
    "collect_redis_metrics": {
      "command": "redis",
      "subscribers": [
        "redis"
      ],
      "interval": 60,
      "type": "metric"
    }
  }
}
{{< /highlight >}}

[1]: https://github.com/sensu-plugins/sensu-plugins-cpu-checks
[2]: https://github.com/sensu-plugins/sensu-plugins-memory-checks
[3]: https://github.com/sensu-plugins/sensu-plugins-disk-checks
[4]: https://github.com/sensu-plugins/sensu-plugins-network-checks
[5]: https://github.com/sensu-plugins
[6]: http://docs.sensu.io/sensu-core/1.4/api/health-and-info/#reference-documentation
[7]: https://github.com/sensu-plugins/sensu-plugins-http/blob/master/bin/check-http.rb
[8]: https://github.com/sensu-plugins/sensu-plugins-network-checks/blob/master/bin/check-ports.rb