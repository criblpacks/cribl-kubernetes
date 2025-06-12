# Cribl Kubernetes Pack

## About this Pack

This pack handles logs from Kubernetes environments, including logs from control plane components, nodes, and containers. It is designed to:

* Normalize and enrich Kubernetes logs with contextual metadata (e.g., cluster, namespace, pod, container name).
* Support different input formats: JSON, CRI-O-style logs, and standard container output.
* Enable log reduction by trimming verbose fields, standardizing timestamps, and optionally dropping less critical messages such as debug-level logs.
* Allow for flexible routing of logs to appropriate pipelines or destinations based on source, cluster, or workload.
* Add filters to the pipelines as desired to drop events based on field content or log severity.

Route your Kubernetes logs to the pack using Cribl Stream routes or Quick Connect, and select appropriate output formats for downstream consumers like Splunk, Elasticsearch, or S3.

---

## Deployment

This pack supports logs delivered in JSON or plaintext formats, typically collected from sources such as Fluent Bit, Vector, or file-based collectors mounted on Kubernetes nodes.

### 1. Configure the Pack

This pack includes several functions that normalize Kubernetes logs, extract structured fields, and enrich events with metadata.

* Use reduction functions to minimize the volume of less useful logs (e.g., container stdout noise, control plane debug logs).
* Only enable one output format at a time. Multiple active outputs may interfere with formatting.
* Configure the global variable `k8s_default_cluster` for cases where cluster name is not available in logs.

### 2. Configure the Collector

Configure your Kubernetes log collector (e.g., Fluent Bit or REST API log forwarder) to send data to Cribl Stream:

1. Ensure logs are tagged with necessary context (e.g., `pod_name`, `namespace`, `container_name`, `log`).
2. For file-based log delivery (e.g., `/var/log/containers/*.log`), ensure proper multiline event breaking.
3. If JSON-formatted logs are used, use a matching Event Breaker to extract individual log lines from arrays.
4. Sample Fluent Bit configuration:

   * Input: tail logs from `/var/log/containers/*.log`
   * Output: HTTP or TCP to Cribl Stream
   * Enable Kubernetes filter plugin to add labels and pod metadata

### 3. Tie the Pack to Kubernetes Logs

#### Send to Routes

1. In your Routes, select the `[Pack] cribl-kubernetes (Cribl Kubernetes Pack)` pipeline.

#### Inline

1. For Pipeline, select `[Pack] cribl-kubernetes (Cribl Kubernetes Pack)` pipeline.
2. For Destination, select your desired destination (e.g., Splunk, S3, Cribl Edge).

---

## Upgrades

Upgrading certain Cribl Packs using the same Pack ID can have unintended consequences. See [Upgrading an Existing Pack](https://docs.cribl.io/stream/packs#upgrading) for details.

---

## Release Notes

### Version 1.0.0

* Initial release
* Added support for structured and unstructured Kubernetes log formats
* Added enrichment functions for cluster, pod, namespace, and container metadata
* Created reduction logic to filter and trim common noisy fields
* Sample routes and global variables included

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

## Event Breaker

Under **Knowledge > Event Breakers**, add a new rule for JSON-formatted Kubernetes logs:

```json
{
  "minRawLength": 128,
  "id": "Kubernetes JSON Event Breaker",
  "rules": [
    {
      "condition": "_raw.includes(\"log\":) && _raw.includes(\"kubernetes\")",
      "type": "json",
      "timestampAnchorRegex": "/\\\"time\\\":\\\"/",
      "timestamp": {
        "type": "format",
        "length": 100,
        "format": "%Y-%m-%dT%H:%M:%S.%LZ"
      },
      "timestampTimezone": "utc",
      "timestampEarliest": "-1weeks",
      "timestampLatest": "+1week",
      "maxEventBytes": 10485760,
      "disabled": false,
      "parserEnabled": false,
      "shouldUseDataRaw": false,
      "jsonExtractAll": true,
      "eventBreakerRegex": "/[\\n\\r]+(?!\\s)/",
      "name": "Kubernetes JSON Logs"
    }
  ],
  "description": "Breaker for Kubernetes container logs in JSON format",
  "tags": "rest,json,kubernetes"
}
```

