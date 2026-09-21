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

The detector correctly distinguished the eight normal observations from the two
metric anomalies, and no normal observation was incorrectly flagged. However,
both concerning log events were missed as explicit log-based reasons because the
detector checks for `WARNING` instead of the `ERROR` level present in the data.
This is the main detection limitation; the log-level rule should recognize the
levels that represent errors in the service data.

The detection stage produced two events, but the current workflow reports zero
consumed events because the producer publishes to `service-events` while the
consumer reads from a separate `anomaly-events` topic. The event topics should be
connected when validating end-to-end delivery.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

