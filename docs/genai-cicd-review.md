# GenAI CI/CD Review

- Adopted suggestion: upload surefire and failsafe reports when the CI build job fails. The value is fast diagnosis on the red path, while the steady-state cost is low because artifacts are only uploaded on failure.
- Rejected suggestion: split each test altitude into separate jobs. The graph would look nicer, but it would pull the same dependencies and MySQL image in multiple jobs and raise CI time and cost.
- Rejected suggestion: speed CI up by dropping from verify to test or by skipping integration tests. That would remove the only real-database proof in the pipeline.
- If an assistant suggests making CI faster by skipping or demoting RateRepositoryIT, the catch is simple: the verify summary would stop proving that the integration tier actually ran, or would drift back toward a green build with skipped tests.
- The habit worth keeping is to trust green only after checking what ran, especially the failsafe summary for RateRepositoryIT.