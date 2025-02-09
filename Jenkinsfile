// Configuration and utility functions
def sendEmail(htmlContent, emailConfig) {
    emailext(
        subject: "${emailConfig.subject}",
        body: htmlContent,
        to: emailConfig.recipients,
        from: emailConfig.sender,
        mimeType: 'text/html'
    )
}

def generateHtmlReport(reportData, config) {
    return """
        <!DOCTYPE html>
        <html>
        <head>
            <style>
                body {
                    font-family: Arial, sans-serif;
                    line-height: 1.6;
                    color: #333;
                    max-width: 800px;
                    margin: 0 auto;
                    padding: 20px;
                }
                .header {
                    background-color: #f8f9fa;
                    padding: 20px;
                    border-radius: 5px;
                    margin-bottom: 20px;
                }
                .summary {
                    background-color: #e9ecef;
                    padding: 15px;
                    border-radius: 5px;
                    margin-bottom: 20px;
                }
                table {
                    width: 100%;
                    border-collapse: collapse;
                    margin-bottom: 20px;
                }
                th, td {
                    padding: 12px;
                    text-align: left;
                    border: 1px solid #dee2e6;
                }
                th {
                    background-color: #f8f9fa;
                    font-weight: bold;
                }
                tr:nth-child(even) {
                    background-color: #f8f9fa;
                }
                .status-pass {
                    color: #28a745;
                    font-weight: bold;
                }
                .status-fail {
                    color: #dc3545;
                    font-weight: bold;
                }
                .footer {
                    margin-top: 20px;
                    padding-top: 20px;
                    border-top: 1px solid #dee2e6;
                    font-size: 0.9em;
                    color: #6c757d;
                }
            </style>
        </head>
        <body>
            <div class="header">
                <h2>${config.reportTitle}</h2>
                <p>Environment: ${config.environment}</p>
                <p>Build Number: ${config.buildNumber}</p>
                <p>Execution Date: ${new Date().format('yyyy-MM-dd HH:mm:ss')}</p>
            </div>

            <div class="summary">
                <h3>Test Execution Summary</h3>
                <p>Total Collections: ${reportData.size()}</p>
                <p>Passed Collections: ${reportData.count { it.status == 'Pass' }}</p>
                <p>Failed Collections: ${reportData.count { it.status == 'Failed' }}</p>
            </div>

            <table>
                <thead>
                    <tr>
                        <th>Collection Name</th>
                        <th>Total Requests</th>
                        <th>Passed Assertions</th>
                        <th>Failed Assertions</th>
                        <th>Status</th>
                        <th>Report</th>
                    </tr>
                </thead>
                <tbody>
                    ${reportData.collect { row -> """
                        <tr>
                            <td>${row.name}</td>
                            <td>${row.requests}</td>
                            <td>${row.passed}</td>
                            <td>${row.failed}</td>
                            <td class="status-${row.status.toLowerCase()}">${row.status}</td>
                            <td><a href="${row.link}">View Report</a></td>
                        </tr>
                    """ }.join('\n')}
                </tbody>
            </table>

            <div class="footer">
                <p>This report was generated automatically by Jenkins CI/CD pipeline.</p>
                <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
            </div>
        </body>
        </html>
    """
}

def runCollection(collection, environment_url, config) {
    def reportPath = "${config.reportDir}/report_${collection.name}.html"
    def jsonReportPath = "${config.reportDir}/summary_${collection.name}.json"

    echo "Running collection: ${collection.name}"

    def result = sh(
        script: """
            newman run ${collection.url} \
            -e ${environment_url} \
            --reporters cli,htmlextra,json \
            --reporter-htmlextra-export ${reportPath} \
            --reporter-json-export ${jsonReportPath} \
            --reporter-htmlextra-inlineAssets \
            ${collection.options ?: ''}
        """,
        returnStatus: true
    )

    def summary = readJSON file: jsonReportPath
    return [
        name: collection.name,
        requests: summary.run.stats.requests.total,
        passed: summary.run.stats.assertions.total - summary.run.stats.assertions.failed,
        failed: summary.run.stats.assertions.failed,
        status: result == 0 ? "Pass" : "Failed",
        link: "${config.artifactBaseUrl}${reportPath}"
    ]
}

pipeline {
    agent any

    tools {
        nodejs 'NodeJS_22'
    }

    parameters {
        text(
            name: 'COLLECTIONS_JSON',
            defaultValue: '''[
                {
                    "name": "Sample_Collection",
                    "url": "https://api.postman.com/collections/YOUR_COLLECTION_ID?apikey=${POSTMAN_API_KEY}"
                }
            ]''',
            description: 'JSON array of collections to run. Format: [{"name": "CollectionName", "url": "collection-url"}]'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: ['QA', 'Staging', 'Production'],
            description: 'Select the environment to run the collections'
        )
        string(
            name: 'EMAIL_RECIPIENTS',
            defaultValue: 'team@company.com',
            description: 'Comma-separated list of email recipients'
        )
    }

    environment {
        REPORT_DIR = "newman/${env.BUILD_ID}"
        POSTMAN_API_KEY = credentials('POSTMAN_API_KEY')
        EMAIL_SUBJECT = "API Test Results - ${params.ENVIRONMENT} - Build ${env.BUILD_NUMBER}"
        JENKINS_ARTIFACT_BASE_URL = "${env.JENKINS_URL}job/${env.JOB_NAME}/${env.BUILD_ID}/artifact/"
        
        // Environment-specific configurations
        ENVIRONMENT_URLS = [
            QA: [id: 'YOUR_QA_ENV_ID', url: ''],
            Staging: [id: 'YOUR_STAGING_ENV_ID', url: ''],
            Production: [id: 'YOUR_PROD_ENV_ID', url: '']
        ]
    }

    stages {
        stage('Validate Parameters') {
            steps {
                script {
                    // Validate JSON format
                    def collections = readJSON text: params.COLLECTIONS_JSON
                    if (!collections) {
                        error "Invalid collections JSON format"
                    }
                }
            }
        }

        stage('Setup') {
            steps {
                // Create reports directory
                sh "mkdir -p ${REPORT_DIR}"
                
                // Install dependencies
                sh '''
                    npm install -g newman
                    npm install -g newman-reporter-htmlextra
                '''
            }
        }

        stage('Run Collections') {
            steps {
                script {
                    def collections = readJSON text: params.COLLECTIONS_JSON
                    def config = [
                        reportDir: REPORT_DIR,
                        environment: params.ENVIRONMENT,
                        buildNumber: env.BUILD_NUMBER,
                        reportTitle: "API Test Results - ${params.ENVIRONMENT}",
                        artifactBaseUrl: JENKINS_ARTIFACT_BASE_URL
                    ]

                    def environment_url = "https://api.postman.com/environments/${ENVIRONMENT_URLS[params.ENVIRONMENT].id}/?apikey=${POSTMAN_API_KEY}"
                    
                    def reportData = collections.collect { collection ->
                        runCollection(collection, environment_url, config)
                    }

                    def emailConfig = [
                        subject: EMAIL_SUBJECT,
                        recipients: params.EMAIL_RECIPIENTS,
                        sender: 'jenkins@company.com'
                    ]

                    def htmlContent = generateHtmlReport(reportData, config)
                    writeFile file: "${REPORT_DIR}/overall_report.html", text: htmlContent
                    sendEmail(htmlContent, emailConfig)
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "${REPORT_DIR}/**/*", allowEmptyArchive: true
        }
        failure {
            echo 'Test execution failed. Please check the reports for details.'
        }
        cleanup {
            // Clean up workspace except reports
            cleanWs(patterns: [[pattern: "${REPORT_DIR}/**/*", type: 'EXCLUDE']])
        }
    }
}