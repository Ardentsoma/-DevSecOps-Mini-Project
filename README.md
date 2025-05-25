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

1. **Fork the Repository**:

   ```bash
   git clone https://github.com/yourusername/juice-shop-security.git
   cd juice-shop-security

   
