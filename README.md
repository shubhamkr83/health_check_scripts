# Health Check and Build Automation Scripts

## Overview

This repository contains Jenkins pipelines and supporting assets used to automate:

- **Website health checks** for Bizup and Navo production sites.
- **Payment API health checks** using Postman and Newman, with rich reporting.
- **Android app builds** (APK and AAB), signing, and upload to AWS S3.

These jobs are intended to run on Jenkins agents that have network access to the target services and the required tooling installed.

---

## Repository structure

```text
health_check_scripts/
  android_apk_builder/
    jenkinsfile

  bizup_health_check/
    jenkinsfile

  navo_health_check/
    jenkinsfile

  payment_health_check/
    .gitignore
    Payment Health Check.postman_collection.json
    Readme.md
    jenkinsfile
    package.json

  .gitignore
  README.md   (this file)
```

### Subprojects

- **android_apk_builder**  
  Jenkins pipeline that checks out the Android Git repository, builds debug and release APKs and a release AAB, signs release artifacts, uploads them to S3, and archives links.

- **bizup_health_check**  
  Jenkins pipeline that monitors `https://bizup.app` using curl, generates a detailed failure report, uploads the report to S3, and sends Google Chat, WhatsApp, and voice call alerts when the site is down.

- **navo_health_check**  
  Jenkins pipeline that mirrors the Bizup health check but targets `https://navofashion.in`.

- **payment_health_check**  
  Node and Postman based API health check project. The Jenkinsfile runs Newman against the Postman collection, generates JSON and HTML reports, uploads HTML reports to S3 with branch specific paths, and sends smart Google Chat notifications.
  
  This folder has its own detailed documentation. See:
  - `payment_health_check/Readme.md`

---

## Common infrastructure and tools

The jobs in this repo assume:

- **Jenkins** with pipeline support.
- **AWS S3** bucket `bizup-builds` in region `ap-south-1`.
- **AWS CLI** configured on Jenkins agents.
- **curl** available on agents (used for health checks and external APIs).

### Shared credentials (Jenkins IDs)

Configure the following credentials in Jenkins (exact IDs must match the Jenkinsfiles):

- **AWS S3 access**  
  - ID: `aws-s3-creds`  
  - Type: username with password  
  - Username: AWS access key id  
  - Password: AWS secret access key

- **Google Chat webhook**  
  - ID: `google-chat-webhook`  
  - Type: secret text  
  - Value: Google Chat incoming webhook URL

- **Twilio for WhatsApp and voice alerts** (website health checks)
  - ID: `twilio-sid` (secret text)
  - ID: `twilio-token` (secret text)

- **Android signing and Git access** (Android builder)
  - ID: `jenkins-key` (SSH key for `git@github.com:Bizup-world/android.git`)
  - ID: `ANDROID_RELEASE_KEYSTORE` (file credential)
  - ID: `ANDROID_KEYSTORE_PASS` (secret text)
  - ID: `ANDROID_KEY_ALIAS` (secret text)
  - ID: `ANDROID_KEY_PASS` (secret text)

---

## Module details

### 1. Android APK builder (`android_apk_builder`)

**Purpose**  
Automate building, signing, and publishing Android app artifacts for Bizup to S3.

**Key steps in the Jenkinsfile**

- Checks out the Android repo from GitHub using `BRANCH_NAME` parameter (default `develop`).
- Verifies Java, Android SDK build tools, and Gradle versions.
- Builds debug APKs, renames them with a Bizup friendly pattern, calculates sha256 checksums, and uploads APKs plus checksum files to S3.
- Builds signed release APKs and a release AAB using injected signing credentials; uploads artifacts and checksums to S3.
- Generates an `artifacts/s3_links.txt` summary file listing all public S3 URLs.
- Archives all artifacts in Jenkins.

**Jenkins usage**

- Create a pipeline job that uses `android_apk_builder/jenkinsfile` as the script path.
- Configure a string parameter `BRANCH_NAME` to control which Android branch to build.
- Ensure the agent has:
  - Java 17
  - Android SDK at `/opt/android-sdk` with build tools version `34.0.0`
  - Gradle wrapper within the Android repo
  - AWS CLI installed and in PATH

---

### 2. Bizup website health check (`bizup_health_check`)

**Purpose**  
Proactively monitor `https://bizup.app` from Jenkins and alert when the site is down or misconfigured.

**Key behavior**

- Performs an HTTP request with curl and checks that the status code is `200`.
- When the status is not `200`:
  - Gathers diagnostics: timestamp, Jenkins build info, response time, SSL verification result, and server geolocation data from `ip-api.com`.
  - Writes a detailed text report under `health_check_reports/`.
  - Uploads the report to S3 under the `bizup-health-check-report` prefix.
  - Archives the S3 URL in Jenkins artifacts.
  - Sends alerts via:
    - Google Chat (rich text card style message)
    - WhatsApp messages to multiple numbers via Twilio
    - Voice calls to multiple numbers via Twilio
- When the status is `200`, logs a simple success message and sets the build description accordingly.

**Jenkins usage**

- Create a pipeline job that uses `bizup_health_check/jenkinsfile` as the script path.
- Run the job on a schedule (for example, every 5 or 10 minutes) using a Jenkins cron trigger.
- Ensure the agent has curl and AWS CLI installed and network access to:
  - `https://bizup.app`
  - `http://ip-api.com`
  - Twilio APIs
  - Google Chat webhook endpoint

---

### 3. Navo website health check (`navo_health_check`)

**Purpose**  
Same pattern as the Bizup health check, but for `https://navofashion.in`.

**Key differences from Bizup job**

- `WEBSITE_URL` is set to `https://navofashion.in`.
- Reports are stored under the `navo-health-check-report` prefix in S3.
- Alert messages are branded as Navo health check alerts.

**Jenkins usage**

- Create a pipeline job that uses `navo_health_check/jenkinsfile` as the script path.
- Configure the same credentials as the Bizup job.
- Schedule it similarly for periodic monitoring.

---

### 4. Payment API health check (`payment_health_check`)

**Purpose**  
Run Postman based checks against payment APIs, collect rich Newman reports, and publish them to S3 with branch aware paths and intelligent notifications.

**Components**

- `Payment Health Check.postman_collection.json`  
  Postman collection containing the API tests.

- `package.json`  
  Node project configuration with Newman dependency and npm scripts for running the collection with different reporters.

- `jenkinsfile`  
  Jenkins pipeline that:
  - Installs Node dependencies and Newman.
  - Runs Newman with JSON and HTML reporters, with retry logic for flaky networks.
  - Validates that report files are fresh and non stale.
  - Analyzes the Newman JSON to compute request and assertion pass or fail counts.
  - Uploads the HTML report to S3 under a branch aware `API_Automation` folder structure.
  - Sends failure notifications to Google Chat, including detailed summaries of failed tests.

**Local usage (developer machine)**

From the `payment_health_check` folder:

- Install dependencies:
  - `npm install`
- Run tests:
  - `npm run test`  (CLI only)
  - `npm run test:json`  (CLI plus JSON report)
  - `npm run test:html`  (CLI plus HTML report)
  - `npm run test:all`  (CLI plus JSON and HTML reports)

**Jenkins usage**

- Create a pipeline job pointing to `payment_health_check/jenkinsfile`.
- Ensure the agent has:
  - Node runtime and NVM at the path expected by the Jenkinsfile
  - Newman (global or installed by the pipeline)
  - AWS CLI
- For more details, refer to `payment_health_check/Readme.md`.

---

## Getting started

1. **Clone the repository**

   ```bash
   git clone <repo-url>
   cd health_check_scripts
   ```

2. **Configure Jenkins credentials** using the IDs listed in the section above.

3. **Create Jenkins jobs** for each module you want to use, setting the script path to the appropriate subfolder Jenkinsfile.

4. **Install tools** on Jenkins agents:

   - AWS CLI
   - curl
   - For Android builder: Java 17, Android SDK, Gradle wrapper
   - For payment health checks: Node (version compatible with `package.json`) and optionally Newman

5. **Schedule the monitoring jobs** (Bizup and Navo health checks) with cron style triggers if continuous monitoring is required.

---
