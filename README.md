# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

## AIOps Assessment Scenario

This repository monitors a `payment-service` using synthetic operational telemetry. The data represents normal payment requests alongside a short incident involving slow responses, high CPU and memory utilization, and error logs caused by payment and database timeouts.

The operational problem is to identify these service anomalies quickly and move them through an event-driven workflow so they can be consumed for further action. AIOps in this assessment combines service metrics and logs, applies simple anomaly-detection rules, and publishes the resulting events for downstream processing.

## Repository Components

- `data/service_data.json` contains the operational service data, including metrics and log records.
- `src/anomaly_detector.py` evaluates response time, CPU, memory, and log levels against anomaly rules.
- `src/event_topic.py` provides the in-memory event topic used to pass anomaly events.
- `src/event_producer.py` publishes detected anomalies to the topic.
- `src/event_consumer.py` consumes events from the topic.
- `src/aiops_pipeline.py` loads the data, coordinates detection and event flow, and reports the final AIOps results.
- `src/calculations.py` contains standalone calculation examples used by the supporting tests.
- `tests/` contains regression tests for the calculations and AIOps pipeline behavior.



## Operational Data Analysis

The dataset in `data/service_data.json` contains 10 observations for `payment-service` from `2026-09-20T10:00:00` through `2026-09-20T10:09:00`.

- **Metrics:** `response_time_ms` measures request latency, while `cpu_percent` and `memory_percent` measure resource utilization.
- **Log information:** `log_level` identifies the severity (`INFO` or `ERROR`) and `message` describes the event, such as a successful payment, a payment timeout, or a database connection timeout.
- **Timestamps:** `timestamp` uses ISO 8601-style date-time values. The records are ordered chronologically at one-minute intervals, which makes it possible to relate metric changes and log events to a specific point in the service timeline.
- **Normal behavior:** The observations from `10:00` through `10:04` and from `10:07` through `10:09` show successful payment messages at `INFO` level. Response times range from 120 to 150 ms, CPU from 42% to 50%, and memory from 51% to 57%.
- **Unusual behavior:** At `10:05`, response time rises to 610 ms and the service logs an `ERROR` for a payment timeout. At `10:06`, response time reaches 640 ms, CPU reaches 94%, memory reaches 91%, and the log reports a database connection timeout at `ERROR` level. These two records are the incident window and are the observations the anomaly detector should flag.


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

The Main Prtoblems were:
The main problems were:

In aiops_pipeline.py, the detector published to one topic and the consumer listened to a different one. That meant anomalies were never consumed.
In anomaly_detector.py, the code checked for "WARNING" but reported "Error log detected", which was the wrong condition.
The project also had an import-path problem for test execution, which I fixed with pytest.ini.



What was done to fix these:

Fixed the event flow so the producer and consumer use the same topic.
Corrected log-level detection for ERROR records.
Kept package imports working when run both as a script and as a package.
Added regression tests to protect the pipeline behavior and error detection.