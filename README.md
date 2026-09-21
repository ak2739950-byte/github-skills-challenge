# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

## Operational data analysis

The repository’s synthetic service telemetry is a compact time series with a clear baseline and a short anomaly window. The data is stored in [data/service_data.json](data/service_data.json).

### 1. Metrics fields

The numeric operational metrics are:

- response_time_ms: request latency in milliseconds; normal values are around 120–150 ms and spike to 610–640 ms during the anomaly period.
- cpu_percent: CPU utilization percentage; normal range is roughly 42–57%, and it rises to 75–94% during the abnormal events.
- memory_percent: memory utilization percentage; normal range is roughly 51–57%, and it rises to 70–91% during the abnormal events.

These fields are the machine-measured indicators that show service health over time.

### 2. Log information fields

The log-oriented fields are:

- service: identifies the application component generating the event.
- log_level: severity such as INFO or ERROR.
- message: human-readable event text describing what happened.

The dataset shows that logs are paired with metrics for each timestamp, allowing the system to correlate a technical symptom (high latency, high CPU, high memory) with a textual description (for example, “Payment service timeout” and “Database connection timeout”).

### 3. How timestamps are used

The timestamp field is ISO 8601-formatted and appears to represent one-minute observations from 2026-09-20T10:00:00 through 2026-09-20T10:09:00.

This gives a simple chronological sequence for the service telemetry:

- 10:00–10:04: stable normal operation
- 10:05–10:06: abnormal response with elevated resource usage and error logs
- 10:07–10:09: recovery back to normal operation

The timestamps are used to order events and identify the window in which the service degraded.

### 4. Normal behaviour observations

The normal observations are the records with low response times, moderate resource usage, and INFO-level success messages. Specifically:

- 10:00, 10:01, 10:02, 10:03, 10:04
- 10:07, 10:08, 10:09

These records have:

- response_time_ms between roughly 120 and 150 ms
- cpu_percent between roughly 42% and 57%
- memory_percent between roughly 51% and 57%
- log_level = INFO
- message = “Payment request processed successfully”

This pattern looks like steady, healthy operation with no latency or resource saturation issues.

### 5. Unusual behaviour observations

The unusual observations are the records around 10:05 and 10:06:

- 2026-09-20T10:05:00 — response_time_ms = 610, cpu_percent = 75, memory_percent = 70, log_level = ERROR, message = “Payment service timeout”
- 2026-09-20T10:06:00 — response_time_ms = 640, cpu_percent = 94, memory_percent = 91, log_level = ERROR, message = “Database connection timeout”

These observations clearly stand out from the baseline because they combine:

- unusually high latency
- elevated CPU usage and memory pressure
- ERROR-level logging
- explicit timeout/error messages

This is consistent with a short service degradation or outage window in which the application is struggling under load or blocked by a dependency issue.

### Overall conclusion

The dataset shows a healthy service baseline followed by a brief but severe anomaly period. The operation looks normal when response_time_ms stays near 120–150 ms and CPU/memory stay around 40–60%; the service becomes abnormal when latency jumps above 500 ms and resource usage rises sharply, accompanied by ERROR logs and timeout messages.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

## Task 1: Set Up and Understand the Project

This exercise is being completed in the provided GitHub Codespace workspace for the exercise repository. The original assessment structure is preserved, and the application components have not been rewritten.

### AIOps scenario

The service being monitored is a payment application service. It records request latency, CPU utilization, memory utilization, log severity, and a message describing the result of each request.

The operational problem is to identify a short period of service degradation. The expected symptoms are increased response time, elevated CPU and memory usage, and error or timeout messages. AIOps is used to correlate these telemetry signals, detect abnormal records, create anomaly events, and pass those events through the processing workflow.

### Repository components

| Area | Location | Purpose |
| --- | --- | --- |
| Operational data | [data/service_data.json](data/service_data.json) | Ten one-minute payment-service telemetry records. |
| Metrics and logs | `service_data.json` records | Metrics include `response_time_ms`, `cpu_percent`, and `memory_percent`; log fields include `service`, `log_level`, and `message`. |
| Anomaly detection | [src/anomaly_detector.py](src/anomaly_detector.py) | Applies thresholds and returns an anomaly event with its timestamp, service, source record, and reasons. |
| Event production | [src/event_producer.py](src/event_producer.py) | Publishes detected events to an event topic. |
| Event topic | [src/event_topic.py](src/event_topic.py) | Stores events in an in-memory list that simulates a streaming topic. |
| Event consumption | [src/event_consumer.py](src/event_consumer.py) | Reads published events from the topic. |
| Final AIOps processing | [src/aiops_pipeline.py](src/aiops_pipeline.py) | Loads the data, runs detection, publishes events, consumes them, and returns the workflow result. |
| Supporting calculations | [src/calculations.py](src/calculations.py) | Contains independent example calculation helpers used by the calculation tests. |

The workflow is:

Operational Data -> Anomaly Detection -> Anomaly Event -> Producer -> Topic -> Consumer -> AIOps Output

The telemetry shows normal operation from 10:00 through 10:04 and again from 10:07 through 10:09. The anomaly window is 10:05-10:06, when latency, resource utilization, and error messages all indicate service degradation.

## Task 2: Analyse Logs and Metrics

The operational data in [data/service_data.json](data/service_data.json) contains ten records for `payment-service`, one per minute from `2026-09-20T10:00:00` through `2026-09-20T10:09:00`.

### Fields and observations

- **Metrics:** `response_time_ms`, `cpu_percent`, and `memory_percent` measure latency and resource utilization.
- **Log information:** `service` identifies the component, `log_level` identifies severity, and `message` describes the event.
- **Timestamps:** ISO 8601 timestamps order the observations and make the degradation window visible chronologically.
- **Normal:** 10:00-10:04 and 10:07-10:09 show 120-150 ms response times, 42-57% CPU, 51-57% memory, `INFO` logs, and successful payment messages.
- **Unusual:** 10:05 has 610 ms latency, 75% CPU, 70% memory, and an `ERROR` payment timeout. 10:06 has 640 ms latency, 94% CPU, 91% memory, and an `ERROR` database connection timeout.

## Task 3: Identify Anomalies

The provided `AnomalyDetector` uses thresholds of response time greater than 500 ms, CPU greater than 80%, and memory greater than 80%. It also reports `ERROR` logs as a concerning reason.

| Timestamp | Service | Metrics | Log | Reasons flagged |
| --- | --- | --- | --- | --- |
| 2026-09-20T10:05:00 | `payment-service` | 610 ms, CPU 75%, memory 70% | `ERROR`: Payment service timeout | High response time; Error log detected |
| 2026-09-20T10:06:00 | `payment-service` | 640 ms, CPU 94%, memory 91% | `ERROR`: Database connection timeout | High response time; High CPU utilization; High memory utilization; Error log detected |

The detector found both expected anomalies and did not flag any of the eight normal records. Each event retains the complete input record in its `source` field. One limitation is that the thresholds are fixed rather than learned from a service baseline; a rolling baseline and configurable severity rules would improve the approach.


## Task 4: AIOps Event Flow

The component roles are:

- **Event/message:** The dictionary returned by `AnomalyDetector`, containing type, timestamp, service, reasons, and source data.
- **Producer:** `EventProducer.publish()` sends a detected event to the topic.
- **Topic:** `EventTopic` is the in-memory channel that stores published messages.
- **Consumer:** `EventConsumer.consume()` reads messages from the topic.
- **AIOps output:** `run_pipeline()` returns the consumed events for downstream processing and reporting.

Operational Data -> Anomaly Detection -> Event -> Producer -> Topic -> Consumer -> AIOps Output

## Task 5: Investigate and Correct the Workflow

## Task 5: Investigate and Correct the Workflow

### Issue 1: Error logs were not reported

- **Affected component:** [src/anomaly_detector.py](src/anomaly_detector.py)
- **Cause:** The detector checked for `WARNING`, but the supplied anomaly records use `ERROR`.
- **Correction applied:** Changed the log-level condition to detect `ERROR`.
- **Verification result:** Both timeout records include `Error log detected`.

### Issue 2: Events were not reaching the consumer

- **Affected component:** [src/aiops_pipeline.py](src/aiops_pipeline.py)
- **Cause:** The producer and consumer used separate `EventTopic` objects, so the consumer read an empty topic.
- **Correction applied:** Shared one `anomaly-events` topic with both components.
- **Verification result:** Two events are published and two events are consumed.

### Issue 3: Package imports were not reproducible

- **Affected components:** [src/aiops_pipeline.py](src/aiops_pipeline.py), [src/event_producer.py](src/event_producer.py), and [src/event_consumer.py](src/event_consumer.py)
- **Cause:** Absolute internal imports worked only when `src` was manually added to `PYTHONPATH`.
- **Correction applied:** Added package-relative imports with a direct-script fallback.
- **Verification result:** The test suite and module pipeline command both run successfully.

## Task 6: Execute the End-to-End Pipeline

```text
Records processed: 10
Anomalies detected: 2
Events consumed: 2
```

The consumed events were for 10:05 and 10:06, verifying the complete event-processing path.

## Task 7: Documentation and Reproduction

From the repository root:

1. Install dependencies with `pip install -r requirements.txt`.

2. Operational Data

Describe the metrics, logs, and timestamps present in the provided operational data.

3. Observations from Logs and Metrics

Describe the normal and unusual observations found during your analysis

 4. Anomaly Detection Findings

Record the anomalies detected by the provided anomaly-detection component, including relevant metric/log information.Also mention whether any expected anomaly was missed or any normal event was incorrectly flagged.

5. Event-Processing Flow

Operational Data → Anomaly Detection → Event → Producer → Topic → Consumer → AIOps Output




