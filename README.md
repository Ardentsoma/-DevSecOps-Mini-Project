# 🛡️ Juice Shop Security Pipeline

## 📌 Project Overview

This project demonstrates the integration of security scanning tools within a CI/CD pipeline for the OWASP Juice Shop application. The pipeline performs:

- **Secret Detection** with [Gitleaks](https://github.com/gitleaks/gitleaks)
- **Static Code Analysis** with [Semgrep](https://semgrep.dev/)
- **Dependency Scanning** with [NJSSCAN](https://github.com/ajinabraham/njsscan)

Scan results are generated in JSON format and programmatically uploaded to [DefectDojo](https://www.defectdojo.org/) for centralized vulnerability management.

---

## ⚙️ Setup Instructions

### 🔧 Prerequisites

Ensure the following tools are installed and accessible in your system's PATH:

- [Jenkins](https://www.jenkins.io/) (with necessary plugins)
- [Python 3.x](https://www.python.org/downloads/)
- [Gitleaks](https://github.com/gitleaks/gitleaks)
- [Semgrep](https://semgrep.dev/docs/getting-started/)
- [NJSSCAN](https://github.com/ajinabraham/njsscan)
- [DefectDojo](https://github.com/DefectDojo/django-DefectDojo) instance (local or remote)

### 🛠️ Jenkins Pipeline Setup

1. **Clone the Repository**:

I already have a forked repository of the app but it can be cloned also
```bash
git clone https://github.com/juice-shop/juice-shop.git
```

2. **Jenkins Pipeline Configuration**
```groovy
pipeline {
    agent any

    environment {
        DEFECTDOJO_URL = 'https://demo.defectdojo.org/'
        DEFECTDOJO_API_KEY = credentials('defectdojo-token')  
        DEFECTDOJO_USER = 'admin'
        ENGAGEMENT_ID = '4'
    }

    stages {
        stage('Git checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Ardentsoma/juice-shop.git']])
            }
        }

        stage('Git leaks scanning') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                        bat "gitleaks detect --source . -v --report-format=json --report-path=gitleaks-report.json"
                    }
                }
                archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true
            }
        }

        stage('Node.js scanning') {
            steps {
                bat 'C:/Users/Soma/AppData/Local/Programs/Python/Python313/Scripts/nodejsscan.exe -d . -o nodejsscan-report.json -v'
                archiveArtifacts artifacts: 'nodejsscan-report.json', allowEmptyArchive: true
            }
        }

        stage('semgrep scanning') {
            steps {
                bat 'set PYTHONIOENCODING=utf-8'
                bat 'C:/Users/Soma/AppData/Local/Programs/Python/Python313/Scripts/semgrep.exe --config auto . --verbose --json > semgrep_results.json'
                archiveArtifacts artifacts: 'semgrep_results.json', allowEmptyArchive: true
            }
        }

        stage('Upload Scan results to DefectDojo') {
            steps {
                bat """
                echo Uploading Gitleaks report to DefectDojo...
                curl -X POST "%DEFECTDOJO_URL%/api/v2/import-scan/" ^
                  -H "Authorization: Token %DEFECTDOJO_API_KEY%" ^
                  -F "engagement=%ENGAGEMENT_ID%" ^
                  -F "scan_type=Gitleaks Scan" ^
                  -F "file=@gitleaks-report.json" ^
                  -F "active=true" ^
                  -F "verified=false"

                echo Uploading Semgrep report to DefectDojo...
                curl -X POST "%DEFECTDOJO_URL%/api/v2/import-scan/" ^
                  -H "Authorization: Token %DEFECTDOJO_API_KEY%" ^
                  -F "engagement=%ENGAGEMENT_ID%" ^
                  -F "scan_type=Semgrep JSON Report" ^
                  -F "file=@semgrep_results.json" ^
                  -F "active=true" ^
                  -F "verified=false"

                echo Uploading NJSSCAN report to DefectDojo...
                curl -X POST "%DEFECTDOJO_URL%/api/v2/import-scan/" ^
                  -H "Authorization: Token %DEFECTDOJO_API_KEY%" ^
                  -F "engagement=%ENGAGEMENT_ID%" ^
                  -F "scan_type=Node Security Platform Scan" ^
                  -F "file=@nodejsscan-report.json" ^
                  -F "active=true" ^
                  -F "verified=false"
                """
            }
        }
    }
}
```

   
