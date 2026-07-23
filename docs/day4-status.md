# Day 4 Status

## Test Altitudes

- Unit: [fx-exchange/fx-app-spring/src/test/java/com/fx/api/ConversionServiceTest.java](fx-exchange/fx-app-spring/src/test/java/com/fx/api/ConversionServiceTest.java) isolates rounding, fee calculation, and unknown-pair handling with a mocked repository.
- Web slice: [fx-exchange/fx-app-spring/src/test/java/com/fx/api/web/RateControllerTest.java](fx-exchange/fx-app-spring/src/test/java/com/fx/api/web/RateControllerTest.java) checks list JSON, 404 on missing pair, 201 on valid conversion, and 400 validation failure.
- Integration: [fx-exchange/fx-app-spring/src/test/java/com/fx/api/repo/RateRepositoryIT.java](fx-exchange/fx-app-spring/src/test/java/com/fx/api/repo/RateRepositoryIT.java) boots MySQL with Testcontainers and relies on Liquibase to create and seed fxdb.
- The intended fast/full split is mvn test for unit plus slice, and mvn verify for unit plus slice plus RateRepositoryIT.

## CI Guards

- [fx-exchange/.github/workflows/ci.yml](fx-exchange/.github/workflows/ci.yml) runs on pull requests only, so PRs are gated before merge and main is not rebuilt twice.
- The build job runs mvn -B verify in fx-app-spring, which covers unit, slice, and Testcontainers integration tests.
- The lint job runs mvn -B validate in parallel to catch basic Maven/project issues cheaply.
- The dependency-check job runs OWASP dependency-check in tolerated-failure mode so the report is visible without blocking every PR.
- On build failure, surefire and failsafe reports are uploaded as artifacts. This was the one review suggestion adopted because it improves debugging value without splitting the suite or weakening coverage.

## CI Does Not Guard

- CI does not perform a full user-level end-to-end check against a live compose stack. The manual honesty check remains docker compose up and then GET /api/rates returning 10 rows.
- CI does not configure branch protection or environment approvals by itself. Those gates must still be enabled in the GitHub repository settings.
- CI does not replace reading the verify summary. A green build can still hide skipped integration tests if Docker is unavailable.

## Supply Chain Notes

- Direct dependencies are small, but the Maven tree expands into many transitive artifacts. Two representative chains to understand are spring-boot-starter-validation -> hibernate-validator -> org.glassfish:jakarta.el, and spring-boot-starter-web -> spring-webmvc.
- GitHub Advisory search on 2026-07-23 returned no results for org.glassfish jakarta.el and no results for liquibase-core.
- GitHub Advisory search for mysql-connector-j returned multiple advisories, including GHSA-m6vm-37g8-gqvh for com.mysql:mysql-connector-j, which is why alerts and update automation matter even when the current build is green.
- Dependabot configuration lives in [fx-exchange/.github/dependabot.yml](fx-exchange/.github/dependabot.yml). Team rule: review Dependabot PRs within one working day, and merge security updates the same day when CI is green.

## Definition Of Done

1. From fx-app-spring, mvn verify is green, nothing is wrongly skipped in the summary, and docker compose up serves /api/rates with 10 rows.
2. New behaviour has tests at the right altitude: unit for calculation, slice for endpoint and validation, IT for SQL or persistence changes.
3. A teammate reviewed the PR, checked out the branch, ran mvn verify, and approved.
4. Code reaches main only through a pull request, never a direct push.
5. Main is still green after merge, and a fresh clone can build and verify.