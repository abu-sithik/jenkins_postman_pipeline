# Detailed Setup Instructions

This document provides comprehensive setup instructions for the Postman Collections Runner Jenkins pipeline.

## Jenkins Server Setup

### 1. Required Plugins

Install the following Jenkins plugins:
- NodeJS Plugin
- Email Extension Plugin
- Pipeline Plugin
- Credentials Plugin

```
Manage Jenkins > Manage Plugins > Available > Search and install each plugin
```

### 2. NodeJS Configuration

1. Navigate to `Manage Jenkins > Tools`
2. Click "Add NodeJS"
3. Configure as follows:
   - Name: `NodeJS_22`
   - Install automatically: Check
   - Version: Select latest LTS version
4. Click Save

### 3. Credentials Setup

1. Navigate to `Manage Jenkins > Manage Credentials`
2. Click on "Jenkins" under Stores scoped to Jenkins
3. Click "Global credentials"
4. Click "Add Credentials"
5. Configure:
   - Kind: Secret text
   - Scope: Global
   - Secret: Your Postman API key
   - ID: POSTMAN_API_KEY
   - Description: Postman API Key for Collection Runner

### 4. Email Configuration

1. Navigate to `Manage Jenkins > Configure System`
2. Find "Extended E-mail Notification"
3. Configure:
   - SMTP server
   - SMTP port
   - Credentials if required
   - Default recipients
   - Default subject
   - Default content

## Pipeline Configuration

### 1. Create Pipeline Job

1. Click "New Item" on Jenkins dashboard
2. Enter name for your pipeline
3. Select "Pipeline"
4. Click OK

### 2. Configure Pipeline

1. In pipeline configuration:
   - Select "Pipeline script" or "Pipeline script from SCM"
   - If using SCM:
     - Select Git
     - Enter repository URL
     - Specify branch
     - Script Path: Jenkinsfile

### 3. Environment Configuration

Update the `ENVIRONMENT_URLS` variable in Jenkinsfile:

```groovy
ENVIRONMENT_URLS = [
    QA: [id: 'YOUR_QA_ENV_ID', url: ''],
    Staging: [id: 'YOUR_STAGING_ENV_ID', url: ''],
    Production: [id: 'YOUR_PROD_ENV_ID', url: '']
]
```

## Postman Setup

### 1. Collection Preparation

1. Create/organize your Postman collections
2. Ensure all environment variables are properly set
3. Add appropriate tests to requests
4. Get collection IDs from Postman

### 2. Environment Setup

1. Create environments in Postman for each target (QA, Staging, Production)
2. Set appropriate variables
3. Get environment IDs from Postman

## First Run

1. Open pipeline in Jenkins
2. Click "Build with Parameters"
3. Enter test configuration:
   ```json
   [
       {
           "name": "Test_Collection",
           "url": "https://api.postman.com/collections/YOUR_COLLECTION_ID?apikey=${POSTMAN_API_KEY}"
       }
   ]
   ```
4. Select environment
5. Enter email recipients
6. Click Build

## Troubleshooting

### Common Issues

1. **Newman not installing:**
   - Check NodeJS installation
   - Verify network connectivity
   - Check npm registry access

2. **Email notifications not working:**
   - Verify SMTP settings
   - Check email credentials
   - Verify recipient addresses

3. **Reports not generating:**
   - Check write permissions on REPORT_DIR
   - Verify newman-reporter-htmlextra installation
   - Check disk space

### Getting Help

1. Check Jenkins logs for error messages
2. Verify Postman API key validity
3. Ensure collection and environment IDs are correct
4. Contact support through GitHub issues