# Prometheus BOSH Release

This is a [BOSH](http://bosh.io/) release for [Prometheus](https://prometheus.io/).

It includes the following [Prometheus Exporters](https://prometheus.io/docs/instrumenting/exporters/):
* [Blackbox](https://github.com/prometheus/blackbox_exporter)
* [Collectd](https://github.com/prometheus/collectd_exporter)
* [Consul](https://github.com/prometheus/consul_exporter)
* [Cloud Foundry Firehose](https://github.com/frodenas/firehose_exporter)
* [Github](https://github.com/infinityworksltd/github-exporter)
* [Graphite](https://github.com/prometheus/graphite_exporter)
* [HAProxy](https://github.com/prometheus/haproxy_exporter)
* [MySQL](https://github.com/prometheus/mysqld_exporter)
* [Node](https://github.com/prometheus/node_exporter)
* [PushGateway](https://github.com/prometheus/pushgateway)
* [RabbitMQ](https://github.com/kbudde/rabbitmq_exporter)
* [Redis](https://github.com/oliver006/redis_exporter)
* [Statsd](https://github.com/prometheus/statsd_exporter)

## Usage

### Upload the BOSH release

To use this BOSH release, first upload it to your BOSH:

```
bosh target BOSH_HOST
git clone https://github.com/cloudfoundry-community/prometheus-boshrelease.git
cd prometheus-boshrelease
bosh upload release releases/prometheus/prometheus-1.yml
```

### Create a BOSH deployment manifest

Now create a deployment file (using the files at the [examples](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/) directory as a starting point).

### Deploy using the BOSH deployment manifest

Using the previous created deployment manifest, now we can deploy it:

```
bosh deployment path/to/deployment.yml
bosh -n deploy
```

## Copyright

Copyright (c) 2016 Ferran Rodenas. See [LICENSE](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/LICENSE) for details.