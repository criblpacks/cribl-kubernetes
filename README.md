# Cribl Kubernetes Pack

## About this Pack

This pack processes data from Kubernetes (K8S) environments sent via Cribl Edge. It is designed to:

* Provide basic processing of all logs sent via Cribl Edge.
  * All logs sent as JSON inside _raw along with important fields.
  * Logs from the Edge container itself are processed as JSON.
  * Logs from other containers that contain JSON have that JSON extracted and processed.
  * Other logs are encapsulated into a `message` field.
* Process K8S Events into JSON.
* Process and rename K8S Metrics.
* Flexible and customizeable filtering of all log types.
* Built-in support for Cribl HTTP Source and Cribl Lake Destination for all logs.

Note that this pack *requires* that all data be sent via Cribl Edge - see deployment section for details. 

---

## Deployment

This pack supports logs collected via Cribl Edge and sent to Cribl Lake. Note that you *must* create the Cribl Lake Datasets yourself - see below for the pre-configured Dataset Names or create your own and update the Destinations. 

### 1. Configure Cribl Edge

Deploy Cribl Edge into each K8S Cluster according to the [Cribl Edge Docs](https://docs.cribl.io/edge/usecase-kubernetes-observability/#deploy-cribl-edge-via-kubernetes). 

### 2. Configure the Pack

#### Source
The pack comes pre-configured with a Cribl HTTP input called `cribl-http-k8s`. Update it's configuration (Address/Port/TLS, etc) to accept Edge connections from your Fleet(s).

Note that while other Sources may work, the pack has only been tested against Cribl HTTP.

#### Destination
The pack comes pre-configured with three Cribl Lake Destinations:
* `k8s_logs` - uses dataset named *k8s_logs*.
* `k8s_events` - uses dataset named *k8s_events*.
* `k8s_metrics` - uses dataset named *k8s_metrics*.

You must create (or modify) the datasets before data will flow.

If you want to send to a Destination other than Cribl Lake, change the variable to the name of that Destination. You may need to modify the output format in, though. We recommend to *not* modify the Pack's pipelines but use a post-processing pipeline instead.

#### Filters
The Pack includes 4 filtering pipelines (one for each K8S pipeline) that can be used to drop/filter low value events. A comment at the top lists fields available for filtering.

*Note: Be sure to examine each filter pipeline and ensure the enabled filters meet your needs!*

* `cribl_k8s_logs_filter`: This is the most important filter due to the volume and often low quality of many K8S logs. The included examples use the `kube_container` field. `kube_namespace` is also a useful field to filter on. 
* `cribl_k8s_edge_logs_filter`: Filter logs generated from Cribl Edge.
* `cribl_k8s_events_filter`: Filter K8S Events. 
* `cribl_k8s_metrics_filter`: Filter K8S Metrics. The default is to filter `config_map`, `annotations`, and `labels`. 

#### Variables

This pack includes several variables that have reasonable defaults but should be verified:

* `edge_kube_container` - Container name for Crible Edge
* `cribl_k8s_output_logs_to_lake` - Dataset name for K8S logs output to Cribl Lake
* `cribl_k8s_output_metrics_to_lake` - Dataset name for K8S metrics output to Cribl Lake
* `cribl_k8s_output_events_to_lake` - Dataset name for K8S events output to Cribl Lake
* `k8s_logs_sourcetype` - Default sourcetype for K8S logs
* `k8s_logs_source` - Default source for K8S logs
* `k8s_events_sourcetype` - Default sourcetype for K8S events
* `k8s_events_source` - Default source for K8S events
* `k8s_edge_logs_sourcetype` - Default sourcetype for K8S logs received from the Crible Edge container
* `k8s_edge_logs_source` - Default sourcetype for K8S logs received from the Crible Edge container

---

## Upgrades

Upgrading certain Cribl Packs using the same Pack ID can have unintended consequences. See [Upgrading an Existing Pack](https://docs.cribl.io/stream/packs#upgrading) for details.

---

## Release Notes

### Version 1.0.0

* Initial release
* Supports K8S logs, events, and metrics delivered via Cribl Edge over Cribl HTTP.
* Supports the Cribl HTTP Source (included)
* Supports the Cribl Lake Destination (included)

---

## Contributing to the Pack

To contribute to the Pack, please connect with us on [Cribl Community Slack](https://cribl-community.slack.com/). You can suggest new features or offer to collaborate.

---

## Acknowledgement

Thanks to the Cribl community and platform users for field input on Kubernetes observability needs.

---

## Author

**Cribl Packs Team**

---

## License

This Pack uses the following license: [Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE).

---


