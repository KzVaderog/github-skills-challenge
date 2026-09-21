# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!


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