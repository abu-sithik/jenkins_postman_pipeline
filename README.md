# Postman Collections Runner for Jenkins

A robust Jenkins pipeline solution for automating Postman collection execution as part of your CI/CD process. This tool enables automated API testing with detailed reporting and email notifications.

## Features

- 🚀 Run multiple Postman collections in sequence
- 📊 Generate detailed HTML reports for each collection
- 📧 Send email notifications with test results
- 🔄 Support for multiple environments (QA, Staging, Production)
- 📝 Comprehensive test execution logs
- 🏗️ Easy integration with existing CI/CD pipelines

## Prerequisites

- Jenkins server (2.0 or higher)
- NodeJS installed on Jenkins
- Newman and newman-reporter-htmlextra (automatically installed by the pipeline)
- Postman collections and environments
- Postman API key

## Setup Instructions

### 1. Jenkins Configuration

1. Install required Jenkins plugins:
   - NodeJS Plugin
   - Email Extension Plugin
   - Pipeline Plugin

2. Configure NodeJS in Jenkins:
   - Go to Manage Jenkins > Tools
   - Add NodeJS installation (name it 'NodeJS_22')

3. Configure Jenkins Credentials:
   - Add your Postman API key as a secret text credential named 'POSTMAN_API_KEY'

### 2. Pipeline Setup

1. Create a new Jenkins Pipeline job
2. Copy the contents of `Jenkinsfile` to your pipeline configuration
3. Configure the following environment variables in the Jenkinsfile:
   - Update `ENVIRONMENT_URLS` with your environment IDs
   - Modify email configurations as needed

### 3. Collection Configuration

Create a JSON configuration for your collections:

```json
[
    {
        "name": "Authentication_Tests",
        "url": "https://api.postman.com/collections/YOUR_COLLECTION_ID?apikey=${POSTMAN_API_KEY}"
    },
    {
        "name": "User_API_Tests",
        "url": "https://api.postman.com/collections/YOUR_COLLECTION_ID?apikey=${POSTMAN_API_KEY}"
    }
]
```

## Usage

1. Open your Jenkins pipeline
2. Click "Build with Parameters"
3. Enter/modify:
   - Collections JSON
   - Environment selection
   - Email recipients
4. Click "Build"

## Reports

The pipeline generates two types of reports:
1. Individual HTML reports for each collection
2. Overall summary report with results from all collections

Reports are archived as Jenkins artifacts and can be accessed from the build page.

## Configuration Options

### Environment Variables

| Variable | Description |
|----------|-------------|
| REPORT_DIR | Directory for storing reports |
| POSTMAN_API_KEY | Your Postman API key |
| EMAIL_SUBJECT | Email notification subject |
| JENKINS_ARTIFACT_BASE_URL | Base URL for Jenkins artifacts |

### Pipeline Parameters

| Parameter | Description |
|-----------|-------------|
| COLLECTIONS_JSON | JSON array of collections to run |
| ENVIRONMENT | Environment selection (QA/Staging/Production) |
| EMAIL_RECIPIENTS | Comma-separated list of email recipients |

## Directory Structure

```
.
├── Jenkinsfile                  # Main pipeline script
├── README.md                    # This file
├── docs/                        # Additional documentation
│   ├── setup.md                # Detailed setup instructions
│   └── configuration.md        # Configuration guide
├── scripts/                     # Helper scripts
│   └── email-template.html     # Email template
└── examples/                    # Example configurations
    └── collections.json        # Sample collections config
```
