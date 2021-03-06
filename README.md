# Prometheus BOSH Release [![Build Status](https://ci.starkandwayne.com/api/v1/teams/main/pipelines/prometheus-boshrelease/jobs/testflight/badge)](https://ci.starkandwayne.com/teams/main/pipelines/prometheus-boshrelease?groups=prometheus-boshrelease)

This is a [BOSH](http://bosh.io/) release for [Prometheus](https://prometheus.io/), [Alertmanager](https://prometheus.io/docs/alerting/alertmanager/), and [Grafana](https://grafana.com/).

It includes the following [prometheus exporters](https://prometheus.io/docs/instrumenting/exporters/): [Blackbox](https://github.com/prometheus/blackbox_exporter), [BOSH](https://github.com/cloudfoundry-community/bosh_exporter), [BOSH TSDB](https://github.com/cloudfoundry-community/bosh_tsdb_exporter), [cAdvisor](https://github.com/google/cadvisor), [Cloud Foundry](https://github.com/cloudfoundry-community/cf_exporter), [Cloud Foundry Firehose](https://github.com/cloudfoundry-community/firehose_exporter), [Collectd](https://github.com/prometheus/collectd_exporter), [Consul](https://github.com/prometheus/consul_exporter), [Elasticsearch](https://github.com/justwatchcom/elasticsearch_exporter), [Github](https://github.com/infinityworksltd/github-exporter), [Graphite](https://github.com/prometheus/graphite_exporter), [HAProxy](https://github.com/prometheus/haproxy_exporter), [InfluxDB](https://github.com/prometheus/influxdb_exporter), [Kubernetes](https://github.com/kubernetes/kube-state-metrics), [Memcached](https://github.com/prometheus/memcached_exporter), [MongoDB](https://github.com/dcu/mongodb_exporter), [MySQL](https://github.com/prometheus/mysqld_exporter), [NATS](https://github.com/lovoo/nats_exporter), [PostgreSQL](https://github.com/wrouesnel/postgres_exporter), [PushGateway](https://github.com/prometheus/pushgateway), [RabbitMQ](https://github.com/kbudde/rabbitmq_exporter), [Redis](https://github.com/oliver006/redis_exporter), [Shield](https://github.com/cloudfoundry-community/shield_exporter), [Stackdriver](https://github.com/frodenas/stackdriver_exporter), [Statsd](https://github.com/prometheus/statsd_exporter).

It includes the following [grafana plugins](https://grafana.com/plugins): [clock](https://github.com/grafana/clock-panel), [diagram](https://github.com/jdbranham/grafana-diagram), [histogram](https://github.com/mtanda/grafana-histogram-panel), [piechart](https://github.com/grafana/piechart-panel), [status](https://github.com/Vonage/Grafana_Status_panel), [worldmap](https://github.com/grafana/worldmap-panel), [worldping](https://github.com/raintank/worldping-app).

Questions? Pop in our [slack channel](https://cloudfoundry.slack.com/messages/prometheus/)!

## Table of Contents

* [Usage](https://github.com/cloudfoundry-community/prometheus-boshrelease#usage)
  * [Requirements](https://github.com/cloudfoundry-community/prometheus-boshrelease#requirements)
  * [Clone the repository](https://github.com/cloudfoundry-community/prometheus-boshrelease#clone-the-repository)
  * [Basic deployment](https://github.com/cloudfoundry-community/prometheus-boshrelease#basic-deployment)
  * [Using BOSH Service Discovery](https://github.com/cloudfoundry-community/prometheus-boshrelease#using-bosh-service-discovery)
  * [Monitoring Cloud Foundry](https://github.com/cloudfoundry-community/prometheus-boshrelease#monitoring-cloud-foundry)
    * [Register Cloud Foundry routes](https://github.com/cloudfoundry-community/prometheus-boshrelease#register-cloud-foundry-routes)
    * [Use UAA for Grafana authentication](https://github.com/cloudfoundry-community/prometheus-boshrelease#use-uaa-for-grafana-authentication)
  * [Operations files](https://github.com/cloudfoundry-community/prometheus-boshrelease#operations-files)
  * [Deployment variables and the var-store](https://github.com/cloudfoundry-community/prometheus-boshrelease#deployment-variables-and-the-var-store)
* [Contributing](https://github.com/cloudfoundry-community/prometheus-boshrelease#contributing)
* [License](https://github.com/cloudfoundry-community/prometheus-boshrelease#license)

## Usage

### Requirements

In order to use this BOSH release you will need:

* [BOSH CLI v2](https://bosh.io/docs/cli-v2.html)
* An already deployed [BOSH environment](http://bosh.io/docs/init.html)
* A compatible [cloud-config](http://bosh.io/docs/terminology.html#cloud-config) with a `default` option for `network` and `vm_types` (you can use the example that comes from [cf-deployment](https://github.com/cloudfoundry/cf-deployment/blob/master/bosh-lite/cloud-config.yml))

Although not mandatory, it is recommended to deploy the [node exporter addon](https://github.com/cloudfoundry-community/node-exporter-boshrelease) in order to get system metrics.

###  Clone the repository

First, clone this repository into your workspace:

```
git clone https://github.com/cloudfoundry-community/prometheus-boshrelease
cd prometheus-boshrelease
export BOSH_ENVIRONMENT=<name>
```

### Basic deployment

To deploy a basic `prometheus` server with `alertmanager` and `grafana` (but no exporters) use the following command:

```
bosh -d prometheus deploy manifests/prometheus.yml \
  --vars-store tmp/deployment-vars.yml
```

Once deployed, look for the `nginx` instance IP address:

```
bosh -d prometheus instances
```

You can reach each component's web ui at:
* `alertmanager`: `http://<nginx-ip-address>:9093`
* `grafana`:  `http://<nginx-ip-address>:3000`
* `prometheus`:  `http://<nginx-ip-address>:9090`

Credentials for each components can be located at the `tmp/deployment-vars.yml` file.

### Using BOSH Service Discovery

If you want to use the [BOSH Service Discovery](https://github.com/cloudfoundry-community/bosh_exporter/blob/master/FAQ.md#how-can-i-use-the-service-discovery) in order to dynamically discover your exporters then add the [monitor-bosh.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-bosh.yml) op file by running the following command (filling the required variables with your own values):

```
bosh -d prometheus deploy manifests/prometheus.yml \
  --vars-store tmp/deployment-vars.yml \
  -o manifests/operators/monitor-bosh.yml \
  -v bosh_url= \
  -v bosh_username= \
  -v bosh_password= \
  --var-file bosh_ca_cert= \
  -v metrics_environment=
```

*NOTE: `metrics_environment` is an arbitrary name to identify your environment (`test`, `nyc-prod`, ...)*

If you have configured your [bosh-deployment](https://github.com/cloudfoundry/bosh-deployment) to use [UAA user management](http://bosh.io/docs/director-users-uaa.html) (via the [uaa.yml](https://github.com/cloudfoundry/bosh-deployment/blob/master/uaa.yml) ops file) we recommend adding the [add-bosh-exporter-uaa-clients.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/bosh/add-bosh-exporter-uaa-clients.yml) op file to your [bosh-deployment](https://github.com/cloudfoundry/bosh-deployment) and then adding the [enable-bosh-uaa.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/enable-bosh-uaa.yml) ops file to the prometheus deployment by running the following command (filling the required variables with your own values):

```
bosh -d prometheus deploy manifests/prometheus.yml \
  --vars-store tmp/deployment-vars.yml \
  -o manifests/operators/monitor-bosh.yml \
  -o manifests/operators/enable-bosh-uaa.yml \
  -v bosh_url= \
  --var-file bosh_ca_cert= \
  -v metrics_environment=
```

### Monitoring Cloud Foundry

If you want to monitor your [Cloud Foundry](https://www.cloudfoundry.org/) platform, first update your [cf-deployment](https://github.com/cloudfoundry/cf-deployment) adding the [add-prometheus-uaa-clients.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/cf/add-prometheus-uaa-clients.yml) op file. This will add the UAA clients required to gather information from the Cloud Foundry [API](https://apidocs.cloudfoundry.org/268/) and [Firehose](https://docs.cloudfoundry.org/loggregator/architecture.html#firehose).

Then add the [monitor-cf.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-cf.yml) op file by running the following command (filling the required variables with your own values):

```
bosh -d prometheus deploy manifests/prometheus.yml \
  --vars-store tmp/deployment-vars.yml \
  -o manifests/operators/monitor-bosh.yml \
  -v bosh_url= \
  -v bosh_username= \
  -v bosh_password= \
  --var-file bosh_ca_cert= \
  -v metrics_environment= \
  -o manifests/operators/monitor-cf.yml \
  -v metron_deployment_name= \
  -v system_domain= \
  -v uaa_clients_cf_exporter_secret= \
  -v uaa_clients_firehose_exporter_secret= \
  -v traffic_controller_external_port= \
  -v skip_ssl_verify=
```

*NOTE: `metron_deployment_name` property should match the `deployment` property of your `metron_agent` jobs ([cf-deployment](https://github.com/cloudfoundry/cf-deployment) uses your `system_domain` and [Pivotal Elastic Runtime](https://network.pivotal.io/products/elastic-runtime) hardcodes `cf`)*

#### Register Cloud Foundry routes

If you want to access `alertmanager`, `grafana`, and `prometheus` web ui's using your Cloud Foundry system domain instead of IP addresses, then you can register those [routes](https://docs.cloudfoundry.org/devguide/deploy-apps/routes-domains.html) inside your Cloud Foundry environment using the  [enable-cf-route-registrar.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/enable-cf-route-registrar.yml) op file by running the following command (filling the required variables with your own values):

```
bosh -d prometheus deploy manifests/prometheus.yml \
  --vars-store tmp/deployment-vars.yml \
  ...
  -o manifests/operators/enable-cf-route-registrar.yml \
  -v system_domain=
```

The op file will register the following routes:

* `https://alertmanager.<cf system domain>`
* `https://grafana.<cf system domain>`
* `https://prometheus.<cf system domain>`

#### Use UAA for Grafana authentication

If you want to allow users registered at your Cloud Foundry environment to access the Grafana dashboards (*Viewer* mode only), first update your [cf-deployment](https://github.com/cloudfoundry/cf-deployment) adding the [add-grafana-uaa-clients.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/cf/add-grafana-uaa-clients.yml) op file. This will add the UAA client required by the Grafana-UAA integration.

Then add the [enable-grafana-uaa.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/enable-grafana-uaa.yml) op file by running the following command (filling the required variables with your own values):

```
bosh -d prometheus deploy manifests/prometheus.yml \
  --vars-store tmp/deployment-vars.yml \
  ...
  -o manifests/operators/enable-grafana-uaa.yml \
  -v system_domain= \
  -v uaa_clients_grafana_secret= \
  --var-file uaa_ssl.ca= \
  --var-file uaa_ssl.certificate= \
  --var-file uaa_ssl.private_key=
```

### Operations files

Additional [operations files](http://bosh.io/docs/cli-ops-files.html) are located at the [manifests/operators](https://github.com/cloudfoundry-community/prometheus-boshrelease/tree/master/manifests/operators) directory. Those files includes a basic configuration, so extra ops files might be needed for additional configuration.

Please review the op files before deploying them to check the requeriments, dependencies and necessary variables.

| File | Description | exporter | dashboards | alerts |
| ---- | ----------- |:--------:|:----------:|:------:|
| [alertmanager-hipchat-receiver.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/alertmanager-hipchat-receiver.yml) | Configures a [HipChat](https://www.hipchat.com/) receiver for `alertmanager` | | | |
| [alertmanager-opsgenie-receiver.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/alertmanager-opsgenie-receiver.yml) | Configures a [OpsGenie](https://www.opsgenie.com/) receiver for `alertmanager` | | | |
| [alertmanager-pagerduty-receiver.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/alertmanager-pagerduty-receiver.yml) | Configures a [PagerDuty](https://www.pagerduty.com/) receiver for `alertmanager` | | | |
| [alertmanager-pushover-receiver.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/alertmanager-pushover-receiver.yml) | Configures a [Pushover](https://pushover.net/) receiver for `alertmanager` | | | |
| [alertmanager-slack-receiver.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/alertmanager-slack-receiver.yml) | Configures a [Slack](https://slack.com/) receiver for `alertmanager` | | | |
| [alertmanager-victorops-receiver.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/alertmanager-victorops-receiver.yml) | Configures a [VictorOps](https://victorops.com/) receiver for `alertmanager` | | | |
| [alertmanager-webhook-receiver.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/alertmanager-webhook-receiver.yml) | Configures a generic webhook receiver for `alertmanager` | | | |
| [alertmanager-web-external-url.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/alertmanager-web-external-url.yml) | Configures the URL under which `alertmanager` is externally reachable | | | |
| [enable-bosh-uaa.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/enable-bosh-uaa.yml) | Configures [monitor-bosh.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-bosh.yml) to use an UAA client (you must apply the [add-bosh-exporter-uaa-clients.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/bosh/add-bosh-exporter-uaa-clients.yml) op file to your [bosh-deployment](https://github.com/cloudfoundry/bosh-deployment)) | | | |
| [enable-cf-route-registrar.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/enable-cf-route-registrar.yml) | Registers `alertmanager`, `grafana`, and `prometheus` as [Cloud Foundry routes](https://docs.cloudfoundry.org/devguide/deploy-apps/routes-domains.html) (under your `system domain`) | | | |
| [enable-grafana-uaa.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/enable-grafana-uaa.yml) | Configures `grafana` user authentication to use [Cloud Foundry UAA](https://docs.cloudfoundry.org/concepts/architecture/uaa.html) (you must apply the [add-grafana-uaa-clients.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/cf/add-grafana-uaa-clients.yml) op file to your [cf-deployment](https://github.com/cloudfoundry/cf-deployment)) | | | |
| [monitor-bosh.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-bosh.yml) | Enables monitoring [BOSH](https://github.com/cloudfoundry-community/bosh_exporter) `jobs` and `processes` and enables [Service Discovery](https://github.com/cloudfoundry-community/bosh_exporter/blob/master/FAQ.md#how-can-i-use-the-service-discovery) | x | x | x |
| [monitor-cadvisor.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-cadvisor.yml) | Enables monitoring [cAdvisor](https://github.com/google/cadvisor) | x | | |
| [monitor-cf.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-cf.yml) | Enables monitoring [Cloud Foundry](https://www.cloudfoundry.org/) via the [Cloud Foundry](https://github.com/cloudfoundry-community/cf_exporter) and [Cloud Foundry Firehose](https://github.com/cloudfoundry-community/firehose_exporter) exporters (you must apply the [add-prometheus-uaa-clients.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/cf/add-prometheus-uaa-clients.yml) op file to your [cf-deployment](https://github.com/cloudfoundry/cf-deployment)) | x | x | x |
| [monitor-collectd.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-collectd.yml) | Enables monitoring [Collectd](https://github.com/prometheus/collectd_exporter) | x | | |
| [monitor-concourse.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-concourse.yml) | Enables monitoring [Concourse CI](http://concourse.ci/) (requires the [monitor-influxdb.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-influxdb.yml) op file) | | x | x |
| [monitor-concourse-influxdb.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-concourse-influxdb.yml) | Enables monitoring [Concourse CI](http://concourse.ci/). Requires [node exporter](https://github.com/cloudfoundry-community/node-exporter-boshrelease) on Concourse VMs (probably as a BOSH add-on) and InfluxDB to be deployed independently and configured as a data source in Grafana as well as Concourse configured to send events to InfluxDB | | x |  |
| [monitor-consul.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-consul.yml) | Enables monitoring [Consul](https://github.com/prometheus/consul_exporter) | x | x | x |
| [monitor-elasticsearch.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-elasticsearch.yml) | Enables monitoring [Elasticsearch](https://github.com/justwatchcom/elasticsearch_exporter) | x | x | x |
| [monitor-github.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-github.yml) | Enables monitoring [Github](https://github.com/infinityworksltd/github-exporter) | x | | |
| [monitor-graphite.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-graphite.yml) | Enables monitoring [Graphite](https://github.com/prometheus/graphite_exporter) | x | | |
| [monitor-haproxy.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-haproxy.yml) | Enables monitoring [HAProxy](https://github.com/prometheus/haproxy_exporter) | x | x | x |
| [monitor-http-probe.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-http-probe.yml) | Enables monitoring HTTP(s) endpoints via the [Blackbox](https://github.com/prometheus/blackbox_exporter) exporter | x | x | x |
| [monitor-influxdb.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-influxdb.yml) | Enables monitoring [InfluxDB](https://github.com/prometheus/influxdb_exporter) | x | | |
| [monitor-kubernetes.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-kubernetes.yml) | Enables monitoring [Kubernetes](https://github.com/kubernetes/kube-state-metrics) | x | x | x |
| [monitor-memcached.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-memcached.yml) | Enables monitoring [Memcached](https://github.com/prometheus/memcached_exporter) | x | | |
| [monitor-mongodb.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-mongodb.yml) | Enables monitoring [MongoDB](https://github.com/dcu/mongodb_exporter) | x | | |
| [monitor-mysql.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-mysql.yml) | Enables monitoring [MySQL](https://github.com/prometheus/mysqld_exporter) | x | x | x |
| [monitor-nats.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-nats.yml) | Enables monitoring [NATS](https://github.com/lovoo/nats_exporter) | x | | |
| [monitor-node.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-node.yml) | Enables monitoring system metrics via the [node exporter](https://github.com/cloudfoundry-community/node-exporter-boshrelease) | | x | |
| [monitor-p-rabbitmq.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-p-rabbitmq.yml) | Enables monitoring [RabbitMQ for PCF](https://network.pivotal.io/products/p-rabbitmq/) (requires the [monitor-cf.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-cf.yml) op file) | | x | x |
| [monitor-p-redis.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-p-redis.yml) | Enables monitoring [Redis for PCF](https://network.pivotal.io/products/p-redis/) (requires the [monitor-cf.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-cf.yml) op file) | | x | x |
| [monitor-postgres.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-postgres.yml) | Enables monitoring [PostgreSQL](https://github.com/wrouesnel/postgres_exporter) | x | x | x |
| [monitor-pushgateway.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-pushgateway.yml) | Deploys a [PushGateway](https://github.com/prometheus/pushgateway)  | x | | |
| [monitor-rabbitmq.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-rabbitmq.yml) | Enables monitoring [RabbitMQ](https://github.com/kbudde/rabbitmq_exporter) | x | x | x |
| [monitor-redis.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-redis.yml) | Enables monitoring [Redis](https://github.com/oliver006/redis_exporter) | x | x | x |
| [monitor-shield.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-shield.yml) | Enables monitoring [Shield](https://github.com/cloudfoundry-community/shield_exporter) | x | x | x |
| [monitor-stackdriver.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-stackdriver.yml) | Enables monitoring [Stackdriver](https://github.com/frodenas/stackdriver_exporter) | x | | |
| [monitor-statsd.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/monitor-statsd.yml) | Enables monitoring [Statsd](https://github.com/prometheus/statsd_exporter) | x | | |
| [use-sqlite3.yml](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/manifests/operators/use-sqlite3.yml) | Use sqlite3 instead of postgres |  | | |

### Deployment variables and the var-store

Some operators files requires additional information to provide environment-specific or sensitive configuration such as various credentials. To do this in the default configuration, we use the `--vars-store`. This flag takes the name of a `yml` file that it will read and write to. Where necessary credential values are not present, it will generate new values based on the type information stored at the different deployment files. Necessary variables that BOSH can't generate need to be supplied as well.
See each particular op files you're using for any additional necessary variables.

See also the [BOSH CLI documentation](http://bosh.io/docs/cli-int.html#value-sources) for more information about ways to supply such additional variables.

## Contributing

Refer to [CONTRIBUTING.md](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/CONTRIBUTING.md).

## License

Apache License 2.0, see [LICENSE](https://github.com/cloudfoundry-community/prometheus-boshrelease/blob/master/LICENSE).
