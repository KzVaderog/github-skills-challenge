# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!


## AIOps Monitoring Workflow

The AIOps monitoring workflow for the `payment-service` analyzes operational data, catches performance issues, and routes those findings through a simulated event pipeline. In this setup, AIOps means combining service metrics and logs, applying detection rules, and pushing the resulting anomalies downstream.

The pipeline is built on a few core components:

- **Data and rules:** `data/service_data.json` holds the metrics and logs. `src/anomaly_detector.py` evaluates CPU, memory, response time, and log severity against predefined rules.

- **Event routing:** Detected anomalies are published by `src/event_producer.py` to an in-memory topic (`src/event_topic.py`), where they are picked up by `src/event_consumer.py`.

- **Orchestration and testing:** Everything is tied together in `src/aiops_pipeline.py`, with supporting math in `src/calculations.py` and regression checks in the `tests/` directory.

## Data Analysis and Anomaly Detection

The provided dataset covers 10 minutes of payment-service activity, with one record per minute from `10:00` to `10:09`. Most of the window shows healthy behavior: successful payments logged at `INFO` level, response times around 120-150 ms, CPU under 50%, and memory around 55%.

However, things went wrong between `10:05` and `10:06`. The `AnomalyDetector` is configured to flag response times over 500 ms, CPU or memory over 80%, and `ERROR` or `WARNING` logs. It caught the incident:

- `10:05`: Response time reached 610 ms, accompanied by an `ERROR` log stating `Payment service timeout`.

- `10:06`: Response time reached 640 ms, CPU rose to 94%, memory reached 91%, and the log showed `Database connection timeout`.

The detector worked exactly as expected. It didn't miss the incident or falsely flag the healthy records, and each alert cleanly includes the original telemetry and the specific reason it triggered.

That said, one clear limitation of this approach is its reliance on static, hardcoded thresholds. It can't learn what "normal" looks like for this specific service over time, meaning a slow, creeping degradation that stays just under the 80% limit would easily slip under the radar.

## Event Pipeline and Applied Fixes
The event workflow is straightforward: the detector creates an ANOMALY event, the producer publishes it to a topic, and the consumer reads it back out. run_pipeline then packages these consumed events as the final output.

While testing the end-to-end flow, I ran into a few underlying bugs and applied the following fixes:

- **Log-level logic:** The detector was incorrectly labeling `WARNING` logs as errors and completely missing actual `ERROR` records. I split the logic to handle warnings and errors appropriately.

- **Topic mismatch:** The producer was sending events to `service-events`, but the consumer was listening to a different `anomaly-events` topic. I updated the pipeline to instantiate a single topic and pass it to both components.

- **Import errors:** Top-level imports were breaking when the test suite loaded the `src` package. Switching to package-relative imports, with a fallback for direct script execution, cleared this up.

After these corrections, the pipeline successfully processed all 10 records, generated the 2 expected events, and routed them flawlessly from producer to consumer.

## Reproducing the Workflow
To run this locally, ensure you are using Python 3.13 or a compatible version and execute the following from the repository root:


```bash
python3 -m pip install -r requirements.txt
pytest -q
python3 src/aiops_pipeline.py
```
You should see the test suite pass cleanly (10 passed). The pipeline execution will confirm 10 records processed, 2 anomalies detected, and 2 events consumed (specifically for the timeouts at 10:05 and 10:06). The anomaly timestamps should be `2026-09-20T10:05:00` and `2026-09-20T10:06:00`.

## Submission

- Repository: [KzVaderog/github-skills-challenge](https://github.com/KzVaderog/github-skills-challenge)
- Pull request: [#116](https://github.com/DebbieAUG/github-skills-challenge/pull/116)






---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)
