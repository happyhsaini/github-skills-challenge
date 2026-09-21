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


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

