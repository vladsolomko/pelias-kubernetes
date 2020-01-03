# Pelias Kubernetes Configuration

This repository contains Kubernetes configuration files to create a production ready instance of Pelias.

We use [Helm](https://helm.sh/) to manage everything.

This configuration is meant to be run on Kubernetes using real hardware or full sized virtual
machines in the cloud. It could work on a personal computer with
[minikube](https://github.com/kubernetes/minikube), but larger installations will likely benefit from additional RAM.

**Announcement:** This project previously took advantage of full planet data
for the [Placeholder](https://github.com/pelias/placeholder) and [Interpolation](https://github.com/pelias/interpolation/) services hosted by
[Nextzen](https://www.nextzen.org/). Unfortunately this hosting is no longer available. By default, this project will now download data for the [Portland Metro](https://github.com/pelias/docker/tree/master/projects/portland-metro) area.

## Setup

First, set up a Kubernetes cluster however works best for you. A popular choice is to use
[kops](https://github.com/kubernetes/kops) on AWS. The [Getting Started on AWS Guide](https://github.com/kubernetes/kops/blob/master/docs/aws.md) is a good starting point.

### Helm Installation

Helm must be installed before continuing. See [https://github.com/helm/helm#install](https://github.com/helm/helm#install) for instructions.

### Sizing the Kubernetes cluster

A working Pelias cluster contains at least some of the following services:
* Pelias API (requires about 50MB of RAM)
* Libpostal Service (requires about 2GB of RAM)
* Placeholder Service (Requires 256MB of RAM)
* Point in Polygon (PIP) Service (Requires up to 6GB of RAM)
* Interpolation Service (requires ~2GB of RAM)

See the [Pelias Services](https://github.com/pelias/documentation/blob/master/services.md) documentation to determine which services to run.

If using kops, it defaults to `t2.small` instances, which are far too small (they only have 2GB of ram).

You can edit the instance types using `kops edit ig nodes` before starting your cluster. `m5.large` is a good choice to start.

### Pelias Helm Chart installation

It's recommended to use a separate `.yaml` file to configure the Pelias chart. See [values.yaml](https://github.com/pelias/kubernetes/blob/master/values.yaml) for a list of what can be configured, then we recommend you create a _new, empty_ yaml file and modify only the values you need to change.

The pelias helm chart can be installed as follows:

```
helm install --name pelias --namespace pelias ./path/to/pelias/chart -f path/to/pelias-values.yaml
```

## Elasticsearch

Elasticsearch is used as the primary data store for Pelias.

Because Elasticsearch is a complex and performance critical piece of a Pelias installation, it is not included in this Helm chart.

Instead, we recommend Pelias users decide for themselves how to instal Elasticsearch and then configure their Pelias services in Kubernetes to connect to Elasticsearch.

Some methods for setting up Elasticsearch:

* [Pelias Elasticsearch Terraform scripts](https://github.com/pelias/terraform-elasticsearch). **Recommended on AWS** and tested with this project
* [Elasticsearch operator](https://github.com/upmc-enterprises/elasticsearch-operator) by UPMC Enterprises
* [Elasticsearch operator](https://github.com/zalando-incubator/es-operator) by Zalando
* [Elastic Cloud](https://www.elastic.co/cloud/) by Elastic, for those looking for a hosted solution
* [Amazon Elasticsearch Service](https://aws.amazon.com/elasticsearch-service/): hosted Elasticsearch on AWS (be sure sure to change the port from AWS's default of 443)

## Running a build

In addition to the primary Helm chart in this repo, which is used for running the Pelias services, the `build` directory contains an _experimental_ Helm chart for running a build on Kubernetes.

However, you probably shouldn't use it, as it hasn't been maintained in some time. Instead, use the [Pelias Docker setup](http://github.com/pelias/docker/) to run a build, and save the Elasticsearch index and other data in locations where your primary Pelias cluster can use it.

## Handy Kubernetes commands

We find ourselves using these from time to time.

### debugging 'init containers'

Sometimes an 'init container' fails to start, you can view the init logs:

```bash
# kubectl logs {{pod_name}} -c {{init_container_name}}
kubectl logs geonames-import-4vgq3 -c geonames-download
```

### opening a bash prompt in a running container

It can be useful to open a shell inside a running container for debugging:

```bash
# kubectl exec -it {{pod_name}} -- {{command}}
kubectl exec -it pelias-pip-3625698757-dtzmd -- /bin/bash
```

# Helm Chart Configuration
The following table lists common configurable parameters of the chart and
their default values. See values.yaml for all available options.

|       Parameter                        |           Description                       |                         Default                     |
|----------------------------------------|---------------------------------------------|-----------------------------------------------------|
| `elasticsearch.host`                   | Elasticsearch hostname                      | `elasticsearch-service`                                              |
| `elasticsearch.port`                   | Elasticsearch access port                   | `9200`                                              |
| `elasticsearch.protocol`               | Elasticsearch access protocol               | `http`                                              |
| `elasticsearch.auth`                   | Elasticsearch authentication `user:pass`    | `-`                                              |
| `pip.enabled`                          | Whether to enable pip service               | `true`                                              |
| `pip.host`                             | Pip service url                             | `http://pelias-pip-service:3102/`                   |
| `pip.pvc.create`                       | To use a custom PVC                         | `-`                                                 |
| `pip.pvc.name`                         | Name of the PVC                             | `-`                                                 |
| `pip.pvc.storageClass`                 | Storage Class to use for PVC	               | `-`                                                 |
| `pip.pvc.storage`                      | Amount of space to claim for PVC	           | `-`                                                 |
| `pip.pvc.annotations`                  | Storage Class annotation for PVC	           | `-`                                                 |
| `pip.pvc.accessModes`                  | Access mode to use for PVC      	           | `-`                                                 |
| `interpolation.enabled`                | Whether to enable interpolation service     | `false`                                             |
| `interpolation.host`                   | Pip service url                             | `http://pelias-interpolation-service:3000/`           |
| `interpolation.pvc.create`             | To use a custom PVC                         | `-`                                                 |
| `interpolation.pvc.name`               | Name of the PVC                             | `-`                                                 |
| `interpolation.pvc.storageClass`       | Storage Class to use for PVC	               | `-`                                                 |
| `interpolation.pvc.storage`            | Amount of space to claim for PVC	           | `-`                                                 |
| `interpolation.pvc.annotations`        | Storage Class annotation for PVC	           | `-`                                                 |
| `interpolation.pvc.accessModes`        | Access mode to use for PVC      	           | `-`                                                 |
