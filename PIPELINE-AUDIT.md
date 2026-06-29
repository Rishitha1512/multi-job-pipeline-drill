# Pipeline Audit

## Lint
**Purpose:** Checks the code for style and linting errors.

**Current Issue:** Runs independently with no timeout. The workflow also currently fails because the ESLint configuration is missing.

**Fix:** Add a timeout and make it the first job so every other job depends on it.

---

## Unit Tests
**Purpose:** Runs unit tests.

**Current Issue:** Runs in parallel with every other job, even if lint fails.

**Fix:** Add `needs: lint` so tests only run after lint succeeds. Add a timeout.

---

## Build
**Purpose:** Builds the application and creates the `dist/` folder.

**Current Issue:** Does not upload the build artifact for later jobs.

**Fix:** Add `needs: lint`, upload the `dist/` folder using `actions/upload-artifact@v4`, and add a timeout.

---

## Integration Tests
**Purpose:** Runs integration tests using the built application.

**Current Issue:** Runs before the build completes and does not download the build artifact.

**Fix:** Add `needs: build`, download the artifact using `actions/download-artifact@v4`, and add a timeout.

---

## Deploy Staging
**Purpose:** Deploys the application to the staging environment.

**Current Issue:** Runs on every branch without waiting for validation jobs.

**Fix:** Make it depend on both unit tests and integration tests, only run on the `main` branch, and add a timeout.

---

## Deploy Production
**Purpose:** Deploys the application to production.

**Current Issue:** Runs on every push without waiting for staging.

**Fix:** Make it depend on staging, only run on the `main` branch, and add a timeout.

---

## Notify
**Purpose:** Sends a notification after the pipeline completes.

**Current Issue:** Does not run if previous jobs fail.

**Fix:** Add `if: always()` so it always runs.