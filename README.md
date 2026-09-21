# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

## AIOps Assessment Scenario

This assessment monitors a `payment-service`. Its operational telemetry includes
response time, CPU usage, memory usage, log level, and log messages. The sample
records are stored in `data/service_data.json`.

The operational problem is detecting service incidents, such as slow responses,
high resource usage, and error logs, before they become harder to diagnose. The
pipeline identifies suspicious telemetry and turns it into events that can be
processed separately from the original service data.

AIOps is used here to combine operational metrics and logs, detect anomalies, and
move those findings through an event-driven workflow. The main components are:

- `data/service_data.json`: operational service telemetry.
- `src/calculations.py`: metric-related calculations used by the assessment.
- `src/anomaly_detector.py`: evaluates telemetry and creates anomaly events.
- `src/event_producer.py`: publishes detected events.
- `src/event_topic.py`: provides the in-memory event-stream topic.
- `src/event_consumer.py`: consumes published anomaly events.
- `src/aiops_pipeline.py`: coordinates loading data, detection, publishing, and
	final event consumption.
- `tests/`: verifies calculations and the AIOps event workflow.

## Operational Data Analysis

The synthetic data contains 10 observations for `payment-service`, recorded one
minute apart from `2026-09-20T10:00:00` through `2026-09-20T10:09:00`. The
timestamp identifies when each telemetry observation occurred and allows the
records to be viewed as a short time series.

- **Metrics:** `response_time_ms` measures request latency, while `cpu_percent`
	and `memory_percent` measure resource utilization.
- **Logs:** `log_level` and `message` describe the service log event. `service`
	identifies the emitting service and is useful context for both metrics and
	logs.
- **Normal behaviour:** The records at 10:00-10:04 and 10:07-10:09 have
	`INFO` logs saying the payment request was processed successfully. Response
	times are 120-150 ms, CPU is 42-50%, and memory is 51-57%.
- **Unusual behaviour:** At 10:05, response time rises to 610 ms and the log
	reports a payment service timeout at `ERROR` level. At 10:06, response time
	rises to 640 ms, CPU reaches 94%, memory reaches 91%, and the log reports a
	database connection timeout at `ERROR` level. These two adjacent records are
	the incident window; the later records return to the normal range.

Good luck!


## Anomaly Detection Results

Using the provided `AnomalyDetector` with its default thresholds (response time
over 500 ms, CPU over 80%, or memory over 80%), the pipeline processed all 10
records and produced 2 anomaly events:

- `2026-09-20T10:05:00`: flagged for **high response time** (`610 ms`). The
	related log is `ERROR: Payment service timeout`; CPU was 75% and memory was
	70%.
- `2026-09-20T10:06:00`: flagged for **high response time** (`640 ms`), **high
	CPU utilization** (`94%`), and **high memory utilization** (`91%`). The
	related log is `ERROR: Database connection timeout`.

The detector correctly distinguishes the eight normal observations from the two
anomalous observations, and no normal observation is incorrectly flagged. The
log-level rule was corrected to recognize the `ERROR` level present in the data,
so both concerning log events are now reported with the explicit reason
`Error log detected`.

The detection stage produces two events, including the relevant metric and log
reasons. The event-flow wiring is described below.

## Event Streaming Workflow

The event flow uses the provided components as follows:

- **Event/message:** an anomaly dictionary containing the timestamp, service,
	anomaly type, reasons, and original source record.
- **Producer:** `EventProducer` accepts each detected anomaly and publishes it.
- **Topic:** `EventTopic("anomaly-events")` stores the published messages in
	memory.
- **Consumer:** `EventConsumer` reads messages from that same topic for the
	downstream AIOps processing step.

After connecting the producer and consumer to the same topic, the workflow
processed 10 records, detected 2 anomalies, published 2 events, and consumed 2
events. Both downstream events were printed with their service, timestamp, type,
and detection reasons, confirming complete event delivery.

## End-to-End Pipeline Execution

The corrected workflow was executed successfully through every stage:

`Operational Data -> Anomaly Detection -> Event -> Producer -> Topic -> Consumer -> AIOps`

The final run processed 10 operational records, detected 2 anomalous
observations, generated 2 `ANOMALY` events, published both events to the shared
`anomaly-events` topic, and consumed both events. The downstream output identified
the `payment-service` issues at `10:05` and `10:06`, including high response time,
the `ERROR` log signal, and the high CPU and memory utilization at `10:06`.
This confirms that the final output represents the detected operational issue.

## Issues Identified and Corrected

- The producer and consumer were connected to separate topic instances, so
	detected events could not reach the consumer. Both now use the shared
	`anomaly-events` topic.
- The detector checked for `WARNING` even though the operational data uses
	`ERROR` for concerning logs. The detector now recognizes `ERROR` and records
	`Error log detected` as a reason.

## Limitations and Possible Improvements

The detector uses fixed thresholds and a small set of rules, so it may miss
gradual degradation or service-specific patterns. A production implementation
could learn a baseline from historical telemetry and use a durable event broker
with richer log classification and alert correlation.

## Reproduce the Demonstration

From the repository root, run:

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python -m pytest -q
python src/aiops_pipeline.py
```

The final command should report 10 records processed, 2 anomalies detected, and
2 events consumed, followed by the two `payment-service` anomaly details. On
Windows PowerShell, activate the environment with
`venv\\Scripts\\Activate.ps1` instead of `source venv/bin/activate`.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

