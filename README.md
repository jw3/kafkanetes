# "Kafkanetes"

Run [Apache Kafka](https://kafka.apache.org/) and [Apache ZooKeeper](https://zookeeper.apache.org/) on [OpenShift v3](https://www.openshift.com/).

Proof of concept; builds following architectures:

* 1 ZooKeeper pod <-> 1 Kafka pod
* 1 ZooKeeper pod <-> 2 Kafka pods
* 3 ZooKeeper pods <-> 1 Kafka pod
* 3 ZooKeeper pods <-> 2 Kafka pods

Jim Minter, 24/03/2016

Matthew Farrellee, 15 Feb 2017 (updated: new oc commands; skip persistent volumes; skip template load)

## Quick start

1. Clone repository
 ```bash
oc create -f https://raw.githubusercontent.com/mattf/kafkanetes/master/resources.yaml
```

1. Build the Kafkanetes image, containing CentOS, Java, Kafka and its distribution of Zookeeper
   ```bash
oc new-build https://github.com/mattf/kafkanetes.git
oc logs -f bc/kafkanetes
```

1. Deploy 1-pod Zookeeper
   ```bash
oc new-app kafkanetes-deploy-zk-1
```

1. Deploy 1-pod Kafka
   ```bash
oc new-app kafkanetes-deploy-kafka-1
```

## Follow the [Apache Kafka Documentation Quick Start](https://kafka.apache.org/documentation.html#quickstart)

1. Deploy a debugging container and connect to it
   ```bash
oc new-app kafkanetes-debug
oc rsh $(oc get pods -l deploymentconfig=kafkanetes-debug --template '{{range .items}}{{.metadata.name}}{{end}}')
```

1. Create a topic
   ```bash
bin/kafka-topics.sh --create --zookeeper kafkanetes-zk:2181 --replication-factor 1 --partitions 1 --topic test
```

1. List topics
   ```bash
bin/kafka-topics.sh --list --zookeeper kafkanetes-zk:2181
```

1. Send some messages
   ```bash
bin/kafka-console-producer.sh --broker-list kafkanetes-kafka:9092 --topic test <<EOF
foo
bar 
baz
EOF
```

1. Receive some messages
   ```bash
bin/kafka-console-consumer.sh --bootstrap-server kafkanetes-kafka:9092 --topic test --from-beginning
```

## Notes

* Known issue: with this setup, Kafka advertises itself using a non-qualified domain name, which means that it can only be accessed by clients in the same namespace.  Further customisation to changed the announced domain name/port and/or use NodePorts to enable external access should be fairly straightforward.

* The forthcoming Kubernetes ["Pet Set"](https://github.com/kubernetes/kubernetes/pull/18016) functionality should help normalise the templates for 3-pod ZooKeeper and 2-pod Kafka.
