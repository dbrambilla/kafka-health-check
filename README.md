# Kafka Health Check

Health checker for Kafka that operates by

* inserting a message in a dedicated health check topic and waiting for it to
become available on the consumer side, and
* checking if the broker is in the in-sync replica set for all partitions it replicates.

## Status
[![Build Status](https://travis-ci.org/andreas-schroeder/kafka-health-check.svg?branch=master)](https://travis-ci.org/andreas-schroeder/kafka-health-check)

## Usage

```
kafka-health-check usage:
  -broker-id uint
    	id of the Kafka broker to health check
  -broker-port uint
    	Kafka broker port (default 9092)
  -check-interval duration
    	how frequently to perform health checks (default 10s)
  -no-topic-creation
    	disable automatic topic creation and deletion.
  -server-port uint
    	port to open for http health status queries (default 8000)
  -topic string
    	name of the topic to use - use one per broker, defaults to broker-<id>-health-check
  -zookeeper string
    	ZooKeeper connect string (e.g. node1:2181,node2:2181,.../chroot)
```

## Supported Kafka Versions

Tested with the following Kafka versions:

* 0.9.0.1

## Building

Run `make` to build after running `make deps` to restore the dependencies using [govendor](https://github.com/kardianos/govendor).

### Prerequisites

* Make to run the [Makefile](Makefile)
* [Go 1.6](https://golang.org/dl/) since it's written in Go


## Notable Details on Health Check Behavior

* When first started, the checker tries to find the Kafka broker to check in the cluster metadata. Then, it tries to
  find the health check topic, and creates it if missing by communicating directly with ZooKeeper(configuration:
  10 seconds message lifetime, one single partition assigned to the broker to check).
  This behavior can be disabled by using `-no-topic-creation`.

* When shutting down, the checker deletes to health check topic partition by communicating directly with ZooKeeper.
  This behavior can be disabled by using `-no-topic-creation`.

* The check will try to create the health check topic only on its first connection after startup. If the topic
  disappears later while the check is running, it will not try to re-create its health check topic.

* The check opens a port accepting http requests on a given port (default 8000). The return codes and response bodies
are:
  * `200` with `sync` for a healthy broker that is fully in sync with all leaders.
  * `200` with `imok` for a healthy broker.
  * `500` with `nook` for an unhealthy one.
