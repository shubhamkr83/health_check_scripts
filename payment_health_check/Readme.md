# 🧪 API Automation with Postman & Jenkins

> **Automated API testing pipeline with enhanced reporting, branch-specific S3 organization, and intelligent notifications**

[![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-blue?style=flat-square&logo=jenkins)](https://jenkins.io/)
[![Postman](https://img.shields.io/badge/Postman-API%20Testing-orange?style=flat-square&logo=postman)](https://postman.com/)
[![AWS S3](https://img.shields.io/badge/AWS-S3%20Storage-green?style=flat-square&logo=amazon-aws)](https://aws.amazon.com/s3/)
[![Node.js](https://img.shields.io/badge/Node.js-Runtime-brightgreen?style=flat-square&logo=node.js)](https://nodejs.org/)

---

## 📋 Table of Contents

- [🎯 Overview](#-overview)
- [📁 Project Structure](#-project-structure)
- [🚀 Features](#-features)
- [🔧 Jenkins Setup](#-jenkins-setup)
- [📊 Reports](#-reports)
- [🌿 Branch-Specific S3 Organization](#-branch-specific-s3-organization)
- [🔔 Smart Notifications](#-smart-notifications)
- [🚀 Quick Start](#-quick-start)
- [💡 Benefits](#-benefits)

---

## 🎯 Overview

This project provides a comprehensive API testing automation solution that combines Postman collections with Jenkins CI/CD pipelines. It generates Newman default JSON reports, organizes results by Git branches in AWS S3, and sends intelligent notifications to Google Chat.

## 📁 Project Structure

```
📦 API Automation Project
├── 📂 postman_reports/
│   ├── 📄 [timestamp]_result.json
│   └── 📄 [timestamp]_result.html
├── 📄 Navo API Automation.postman_collection.json
├── 📄 Jenkinsfile
├── 📄 package.json
├── (no extra reporter configs)
├── 📖 BRANCH_S3_SETUP.md
└── 📖 README.md
```

## 🚀 Features

<table>
<tr>
<td>

### 🧪 **Testing**
- ✅ Automated Postman test execution via Newman
- 🔄 Continuous integration with Jenkins
- 📊 Comprehensive test result analysis

</td>
<td>

### 📈 **Reporting**
- 📊 Newman JSON & HTML reports
- 🔍 Pipeline parses JSON for pass/fail summary
- 🌐 HTML reports for visual inspection

</td>
</tr>
<tr>
<td>

### ☁️ **Cloud Storage**
- 🗂️ Automated S3 upload
- 🌿 Branch-specific organization
- 🔗 Direct report URL generation

</td>
<td>

### 🔔 **Notifications**
- 💬 Google Chat integration
- 🎯 Branch-aware alerts
- 📝 Detailed failure summaries

</td>
</tr>
</table>

---

## 🔧 Jenkins Setup

### 🔌 Required Plugins

| Plugin | Purpose |
|--------|---------|
| **Pipeline** | Core pipeline functionality |
| **Post Build Script** | Post-build actions |
| **Mask Passwords** | Secure credential handling |
| **Credentials Binding** | Credential management |

### 🔐 Required Credentials

#### 1. AWS S3 Credentials
```yaml
Type: Username with password
ID: aws-s3-creds
Username: AWS_ACCESS_KEY_ID
Password: AWS_SECRET_ACCESS_KEY
```

#### 2. Google Chat Webhook
```yaml
Type: Secret text
ID: google-chat-webhook
Value: [Your Google Chat Webhook URL]
```

### 🛠️ Required Tools on Jenkins Agent

- **Node.js** (via NVM)
- **Newman** (`npm install -g newman`)
- **AWS CLI**

---

## 📊 Reports

- The pipeline generates timestamped JSON and HTML files under `postman_reports/`.
- The **HTML report is uploaded to S3** at a branch-specific path for easy viewing.
- The **JSON report is used locally** for pipeline analysis and notifications.
- HTML reports provide rich visual inspection of test results with detailed formatting.

### 🧪 Local Testing Commands

```bash
# Install dependencies
npm install

# Run tests with CLI reporter only
npm run test

# Run tests with JSON report
npm run test:json

# Run tests with HTML report
npm run test:html

# Run tests with both JSON and HTML reports
npm run test:all
```

> JSON produced by Newman is consumed by the Jenkins pipeline for summaries.

---

## 🌿 Branch-Specific S3 Organization

### 📂 S3 Folder Structure

```
🗂️ bizup-builds/API_Automation/
├── 🌟 main/
│   └── 📊 reports/
│       └── 🧪 postman/
│           ├── 📄 result-1.html
│           ├── 📄 result-2.html
│           └── ...
├── 🚧 dev/
│   └── 📊 reports/
│       └── 🧪 postman/
│           ├── 📄 result-1.html
│           ├── 📄 result-2.html
│           └── ...
└── 🔧 [other-branches]/
    └── 📊 reports/
        └── 🧪 postman/
            └── ...
```

### 🗺️ Branch Mapping

| Git Branch | S3 Folder | Environment | Description |
|------------|-----------|-------------|-------------|
| `main` | `API_Automation/main/` | 🌟 **Production** | Production reports |
| `master` | `API_Automation/main/` | 🌟 **Production** | Production reports (legacy) |
| `dev` | `API_Automation/dev/` | 🚧 **Development** | Development reports |
| `develop` | `API_Automation/dev/` | 🚧 **Development** | Development reports (alt) |
| `feature/xyz` | `API_Automation/feature-xyz/` | 🔧 **Feature** | Feature branch reports |

### 🔗 Report URL Patterns

#### 🌟 Main Branch Reports
```
https://bizup-builds.s3.ap-south-1.amazonaws.com/API_Automation/main/reports/postman/result-[BUILD_NUMBER].html
```

#### 🚧 Dev Branch Reports
```
https://bizup-builds.s3.ap-south-1.amazonaws.com/API_Automation/dev/reports/postman/result-[BUILD_NUMBER].html
```

#### 🔧 Feature Branch Reports
```
https://bizup-builds.s3.ap-south-1.amazonaws.com/API_Automation/[branch-name]/reports/postman/result-[BUILD_NUMBER].html
```

> 📖 **For detailed branch organization info, see [BRANCH_S3_SETUP.md](./BRANCH_S3_SETUP.md)**

---

## 🔔 Smart Notifications

### 📱 Google Chat Integration

Our notification system provides intelligent, branch-aware alerts:

#### 🎯 Notification Features
- **Branch identification** in alert titles
- **Direct S3 report links** for quick access
- **Detailed failure summaries** with context
- **Environment-specific formatting**

#### 📋 Example Notification (Dev Branch)
```
❌ API Automation Tests Alert (DEV Branch)

Failed Tests: 
🔴 Login Test:
   • Expected status code 200 but got 401

🔗 Build URL: Click Here
📊 Report URL: Click Here (Dev S3 folder)
🌿 Branch: dev
📂 S3 Folder: API_Automation/dev
```

### 🏷️ Report Labels

- JSON is parsed to compute request and assertion pass/fail counts.

---

## 🚀 Quick Start

### 1️⃣ **Setup Repository**
```bash
git clone [your-repo-url]
cd api-automation-project
```

### 2️⃣ **Install Dependencies**
```bash
npm install
```

### 3️⃣ **Test Locally**
```bash
npm run test
```

### 4️⃣ **Configure Jenkins**
- Install required plugins
- Add AWS S3 and Google Chat credentials
- Create pipeline job with this repository

### 5️⃣ **Run Pipeline**
- Push to `dev` or `main` branch
- Monitor Jenkins build progress
- Check S3 for generated reports

### 6️⃣ **Access Reports**
- **Main branch**: `API_Automation/main/reports/postman/`
- **Dev branch**: `API_Automation/dev/reports/postman/`

---

## 💡 Benefits

<div align="center">

### 🎯 **Why This Solution?**

</div>

<table>
<tr>
<td width="50%">

#### 🏢 **Organization**
- **Clear Separation**: Production vs development reports
- **Historical Tracking**: Organized by branch and build
- **Team Collaboration**: Role-based report access
- **Audit Trail**: Complete testing history

</td>
<td width="50%">

#### ⚡ **Efficiency**
- **Automated Reporting**: No manual intervention needed
- **Quick Access**: Direct S3 URLs in notifications
- **Automated Summaries**: Parse Newman JSON for insights
- **Scalable Structure**: Easy to extend for new branches

</td>
</tr>
</table>

---

<div align="center">

### 🌟 **Professional API Testing Made Simple**

*JSON reports • Branch organization • Smart notifications • Cloud storage*

---

**📧 Need help?** Check our detailed setup guides:
- [📖 BRANCH_S3_SETUP.md](./BRANCH_S3_SETUP.md)

</div>

---

<div align="center">
<sub>Built with ❤️ for reliable API testing automation</sub>
</div>