# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

## AIOps Assessment Scenario

For this assessment, I worked with a small monitoring workflow for a `payment-service`. The data includes normal payment requests and a short incident where requests slow down, resource usage increases, and timeout errors appear in the logs.

The goal is to spot those problems and pass the findings through the supplied event workflow. In this example, AIOps means combining the service metrics and logs, applying the existing detection rules, and sending the resulting anomaly events to the next part of the pipeline.

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

`data/service_data.json` contains 10 records for `payment-service`, covering `2026-09-20T10:00:00` to `2026-09-20T10:09:00`.

The metric fields are `response_time_ms`, `cpu_percent`, and `memory_percent`. The log fields are `log_level` and `message`. Each record also has a `timestamp`, and the records are one minute apart, so it is easy to line up the metric changes with the log events.

Most of the records look normal. From `10:00` to `10:04` and again from `10:07` to `10:09`, the service reports successful payments at `INFO` level. Response time stays between 120 and 150 ms, CPU stays between 42% and 50%, and memory stays between 51% and 57%.

The two unusual records are `10:05` and `10:06`. At `10:05`, response time jumps to 610 ms and the log says `Payment service timeout` at `ERROR` level. At `10:06`, response time is 640 ms, CPU is 94%, memory is 91%, and the log reports a database connection timeout.

## Anomaly Detection Review

The existing `AnomalyDetector` uses a 500 ms response-time limit and 80% limits for CPU and memory. It also flags `ERROR` and `WARNING` logs. Running it against the data produced these results:

| Timestamp | Metric and log evidence | Detection reasons |
| --- | --- | --- |
| `2026-09-20T10:05:00` | 610 ms response time; 75% CPU; 70% memory; `ERROR`: `Payment service timeout` | High response time; error log detected |
| `2026-09-20T10:06:00` | 640 ms response time; 94% CPU; 91% memory; `ERROR`: `Database connection timeout` | High response time; high CPU utilization; high memory utilization; error log detected |

All 10 records were processed and the expected two-record incident window was detected. I did not see a missed anomaly or a normal `INFO` record incorrectly flagged. Each result includes the timestamp, service, reasons, and original source record, so the reason for each alert is clear.

One limitation is that the rules use fixed thresholds. They do not learn what is normal for this service or look for gradual changes, so a slow deterioration that stays below a threshold could be missed.

## Event Streaming Workflow Review

I traced the anomaly through the supplied components:

1. The `AnomalyDetector` creates an `ANOMALY` event when a record breaches a metric threshold or contains a concerning log level.
2. The `EventProducer` receives that event and publishes it to the `EventTopic` named `service-events`.
3. The `EventTopic` stores the published event in its in-memory message stream.
4. The `EventConsumer` reads the event from the same topic.
5. `run_pipeline` collects the consumed events and returns them as the downstream AIOps result, preserving the event reasons and source telemetry.

The run processed `10` records, detected `2` anomalies, and consumed `2` events. The detected and consumed lists were identical. The events corresponded to the payment timeout at `10:05` and the database connection timeout at `10:06`.

## Troubleshooting and Corrections

I found three issues while checking the original workflow:

- **Log-level check:** The detector was checking for `WARNING` but labeling it as an error, and it missed the dataset's `ERROR` records. I corrected the handling so `ERROR` and `WARNING` are treated separately. The rerun correctly flags both timeout records.
- **Topic routing:** The producer was using `service-events` while the consumer was connected to a different `anomaly-events` topic. I changed the pipeline to create one topic and pass that same topic to both components. The rerun produced 2 events and consumed both of them.
- **Imports:** The original top-level imports failed when the modules were loaded as the `src` package by the tests. Package-relative imports with a fallback for direct script execution fixed this. Both `pytest` and `python3 src/aiops_pipeline.py` now run successfully.

## End-to-End Execution Result

The final path was:

`Operational Data -> Anomaly Detection -> Event -> Producer -> service-events Topic -> Consumer -> AIOps Pipeline`

The final output processed 10 records, generated 2 `ANOMALY` events, published them to `service-events`, consumed both events, and returned them from `run_pipeline`. The reported issues were the payment service timeout at `10:05` and the database connection timeout at `10:06`, along with the metric and log reasons.

## Reproduce the Demonstration

From the repository root, use Python 3.13 or a compatible version:

```bash
python3 -m pip install -r requirements.txt
pytest -q
python3 src/aiops_pipeline.py
```

The tests should pass (`10 passed`). The pipeline should report 10 records processed, 2 anomalies detected, and 2 events consumed. The anomaly timestamps should be `2026-09-20T10:05:00` and `2026-09-20T10:06:00`.


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

