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

Good luck!


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

