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

## Anomaly Detection Review

The provided `AnomalyDetector` uses thresholds of 500 ms for response time and 80% for both CPU and memory. It also treats `ERROR` and `WARNING` log levels as concerning. The resulting report is:

| Timestamp | Metric and log evidence | Detection reasons |
| --- | --- | --- |
| `2026-09-20T10:05:00` | 610 ms response time; 75% CPU; 70% memory; `ERROR`: `Payment service timeout` | High response time; error log detected |
| `2026-09-20T10:06:00` | 640 ms response time; 94% CPU; 91% memory; `ERROR`: `Database connection timeout` | High response time; high CPU utilization; high memory utilization; error log detected |

The detector processed all 10 records and identified the expected two-record incident window. No expected anomaly was missed in this dataset, and no normal `INFO` event was incorrectly flagged. The detection result includes the timestamp, service, reasons, and complete source record, which makes each decision traceable. One limitation is that the detector uses fixed per-record thresholds and does not learn a service baseline or detect trends; a gradual degradation that remains below a threshold could therefore be missed.

## Event Streaming Workflow Review

The event flow was verified using the provided components:

1. The `AnomalyDetector` creates an `ANOMALY` event when a record breaches a metric threshold or contains a concerning log level.
2. The `EventProducer` receives that event and publishes it to the `EventTopic` named `service-events`.
3. The `EventTopic` stores the published event in its in-memory message stream.
4. The `EventConsumer` reads the event from the same topic.
5. `run_pipeline` collects the consumed events and returns them as the downstream AIOps result, preserving the event reasons and source telemetry.

Execution result: `10` records were processed, `2` anomalies were detected, and `2` events were consumed. The detected and consumed event lists were identical. The events were for `10:05` (payment timeout) and `10:06` (database connection timeout), confirming that anomaly events traveled through the complete producer, topic, consumer, and downstream pipeline.

## Troubleshooting and Corrections

The workflow issues were investigated against the original implementation and corrected within the existing architecture:

- **AnomalyDetector:** The detector checked for `WARNING` but labeled the result as an error, so the dataset's `ERROR` records were not recognized as log anomalies. The correction handles `ERROR` as an error reason and `WARNING` as a warning reason. Re-running detection flags both timeout records at `10:05` and `10:06`.
- **Event topic routing:** The pipeline created `service-events` for the producer but `anomaly-events` for the consumer. Because these were separate in-memory topics, published events could not be consumed. The correction creates one `service-events` topic and passes the same instance to both producer and consumer. Re-running the pipeline produced 2 events and consumed all 2.
- **Python imports:** The original modules used only top-level imports, which failed when the pipeline was imported as the `src` package by tests. The correction uses package-relative imports with a direct-script fallback. Both `pytest` and `python3 src/aiops_pipeline.py` now execute successfully.

## End-to-End Execution Result

The corrected path was executed as:

`Operational Data -> Anomaly Detection -> Event -> Producer -> service-events Topic -> Consumer -> AIOps Pipeline`

The final output processed 10 records, detected 2 anomalous observations, generated 2 `ANOMALY` events, published them to `service-events`, consumed both events, and returned them from `run_pipeline`. The final reported issues were the payment service timeout at `10:05` and the database connection timeout at `10:06`, including their metric and log reasons.

## Reproduce the Demonstration

1. Open this repository in its GitHub Codespace or clone it locally with Python 3.13 or compatible.
2. Install the dependencies:

	```bash
	python3 -m pip install -r requirements.txt
	```

3. Run the regression tests:

	```bash
	pytest -q
	```

4. Execute the complete AIOps workflow from the repository root:

	```bash
	python3 src/aiops_pipeline.py
	```

5. Confirm the output reports `10` records processed, `2` anomalies detected, and `2` events consumed. The two reported timestamps should be `2026-09-20T10:05:00` and `2026-09-20T10:06:00`.


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