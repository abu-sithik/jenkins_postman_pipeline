# Configuration Guide

This document details all configuration options and customization possibilities for the Postman Collections Runner.

## Pipeline Parameters

### Collections JSON Structure

```json
[
    {
        "name": "Collection_Name",
        "url": "collection_url",
        "options": "--timeout-request 10000 --delay-request 1000"
    }
]
```

#### Parameters:
- `name`: Display name for the collection
- `url`: Postman collection URL with API key
- `options` (optional): Additional Newman command line options

### Environment Configuration

```groovy
ENVIRONMENT_URLS = [
    QA: [
        id: 'YOUR_QA_ENV_ID',
        url: '',
        timeout: 30000,
        retries: 2
    ],
    Staging: [
        id: 'YOUR_STAGING_ENV_ID',
        url: '',
        timeout: 45000,
        retries: 1
    ],
    Production: [
        id: 'YOUR_PROD_ENV_ID',
        url: '',
        timeout: 60000,
        retries: 0
    ]
]
```

## Email Configuration

### Basic Settings

```groovy
emailConfig = [
    subject: "API Test Results - ${params.ENVIRONMENT}",
    recipients: params.EMAIL_RECIPIENTS,
    sender: 'jenkins@company.com',
    includeAttachments: true,
    maxAttachmentSize: '10MB'
]
```

### Custom Email Template

Create a custom email template by modifying `scripts/email-template.html`:

```html
<h2>${reportTitle}</h2>
<p>Environment: ${environment}</p>
<table>
    <!-- Custom table structure -->
</table>
```

## Report Customization

### HTML Report Options

```groovy
def reportConfig = [
    title: "API Test Results",
    showEnvironmentData: true,
    includeRequestLogs: true,
    timezone: "UTC",
    showMarkdownLinks: true
]
```

### Report Retention

```groovy
def reportRetentionConfig = [
    daysToKeep: 30,
    numToKeep: 50,
    artifactDaysToKeep: 15,
    artifactNumToKeep: 25
]
```

## Newman Configuration

### Default Newman Options

```groovy
def newmanOptions = [
    timeout: 60000,
    delayRequest: 1000,
    bail: true,
    suppressExitCode: false,
    color: true
]
```

### Environment-Specific Options

```groovy
def environmentOptions = [
    QA: [
        iterations: 1,
        ignoreRedirects: false
    ],
    Staging: [
        iterations: 1,
        ignoreRedirects: true
    ],
    Production: [
        iterations: 1,
        ignoreRedirects: true,
        timeout: 120000
    ]
]
```

## Security Configuration

### API Key Management

```groovy
// In Jenkins Credentials
POSTMAN_API_KEY = credentials('POSTMAN_API_KEY')

// Usage in collection URL
def getCollectionUrl(collectionId) {
    return "https://api.postman.com/collections/${collectionId}?apikey=${POSTMAN_API_KEY}"
}
```

### Environment Variables

Sensitive data should be stored as Jenkins credentials:

```groovy
environment {
    API_KEY = credentials('API_KEY')
    AUTH_TOKEN = credentials('AUTH_TOKEN')
    DATABASE_URL = credentials('DATABASE_URL')
}
```

## Advanced Configuration

### Parallel Execution

```groovy
def parallelConfig = [
    maxParallel: 3,
    failFast: true,
    timeoutMinutes: 30
]
```

### Retry Mechanism

```groovy
def retryConfig = [
    maxRetries: 3,
    retryDelay: 5000,
    retryConditions: [
        responseCode: [500, 502, 503, 504],
        networkError: true
    ]
]
```

### Notification Configuration

```groovy
def notificationConfig = [
    email: true,
    slack: false,
    teams: false,
    jira: false,
    channels: [
        slack: '#api-testing',
        teams: 'API Testing Team'
    ]
]
```