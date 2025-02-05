# Building-a-Free-Python-DevSecOps-Pipeline-with-GitHub-Actions-and-OWASP-ZAP

DevSecOps isn’t just a trend. It is a critical practice in modern software development. By automating security checks within your CI/CD pipeline, you can catch vulnerabilities early and save significant time. In this guide, we’ll create a fully automated DevSecOps pipeline using GitHub Actions and open-source tools like Safety, Bandit, Gitleaks, and OWASP ZAP.

The project we’ll use for demonstration is PyGoat, an intentionally vulnerable Python web application created by OWASP. It’s an excellent tool for learning how to identify and fix security flaws in real-world scenarios.

This educational pipeline provides a foundation for understanding how security testing can be integrated into CI/CD workflows. While this example does not block the pipeline for vulnerabilities, you can enhance it by adding thresholds to halt the workflow if a critical vulnerability level is reached.

The project we’ll use for demonstration is PyGoat, an intentionally vulnerable Python web application created by OWASP. PyGoat is an excellent tool for identifying and mitigating security flaws in real-world applications.

GitHub - adeyosemanputra/pygoat: intentionally vuln web Application Security in django
intentionally vuln web Application Security in django - adeyosemanputra/pygoat
github.com


Pipeline Pros:

Detect Vulnerabilities Early: Tackle security flaws before they become critical.
Optimise Testing: Save valuable time with automated checks integrated into your workflow.
Embrace Open-Source Solutions: Achieve robust security without spending a penny.
Tools Overview

To build this pipeline, we’ll use the following open-source tools:

GitHub Actions: Automates our CI/CD pipeline.
Safety: Scans Python dependencies for vulnerabilities.
Bandit: Performs static analysis to catch security issues in Python code.
Gitleaks: Detects hardcoded secrets in your repository.
OWASP ZAP: A dynamic application security testing (DAST) tool for scanning running applications.
Pipeline Code

https://github.com/OladeleO/pygoat/blob/free-devsecops-pipeline/.github/workflows/main.yml

Step-by-Step Breakdown

1. SCA with Safety

Safety identifies vulnerabilities in your Python dependencies.

Command:
safety check --file=requirements.txt --json > safety-report.json  
Output: A JSON report listing dependencies with known vulnerabilities.
2. SAST with Bandit

Bandit scans Python code for common security issues like insecure use of eval().

Command:
bandit -r . -f html -o bandit-report.html  
Output: An HTML report highlighting issues found in your codebase.
3. Secrets Detection with Gitleaks

Gitleaks scans for hardcoded secrets like API keys or credentials.

Configuration:
uses: gitleaks/gitleaks-action@v2  
with:  
  config: .gitleaks.toml  
4. DAST with OWASP ZAP

OWASP ZAP scans the running PyGoat application for security vulnerabilities like exposed headers or weak authentication controls.

How It Works:
OWASP ZAP interacts with the PyGoat container to identify security flaws.

Output: An HTML report with detailed findings.
  dast_scan:
    runs-on: ubuntu-latest
    services:
      app:
        image: pygoat/pygoat:latest
        ports:
          - 8000:8000
Challenges and Fixes

Challenges are common when building a DevSecOps pipeline. Here’s how to overcome them:

Service Not Reachable: A frequent issue with DAST tools like OWASP ZAP is when the target application service isn’t accessible. To resolve this, use curl -L to verify that the application is running and accessible at the expected endpoint. Additionally, ensure the application is properly started before the DAST scan begins.
Gitleaks Blocking the Pipeline: Secrets detection tools like Gitleaks may halt the pipeline when sensitive information is found. To avoid interruptions, add continue-on-error: true to this step, allowing the pipeline to complete while still flagging issues for review
OWASP ZAP Issue Creation: By default, OWASP ZAP may attempt to create issues in your repository based on findings. If this isn’t desirable, you can disable automatic issue creation by setting allow_issue_writing: false in the ZAP configuration.
How GitHub Actions’ Services Feature Helps

The services feature in GitHub Actions makes running Dynamic Application Security Testing (DAST) easier by allowing dependent services (like application containers) to run alongside your workflow. This ensures that tools like OWASP ZAP can interact seamlessly with the target application.

For example, you can use the services feature to start your target application, such as PyGoat, within the same workflow. This eliminates the need for external setup and ensures the application is running and accessible at the expected endpoint when the DAST scan begins.

Key Benefits:

Reduced Connectivity Issues: This feature ensures the application is up and running, minimising errors like “service not reachable.”
Improved Reliability: Creates a self-contained environment, making your pipeline more robust and repeatable.
Simplified Configuration: Eliminates the need for manual service management by integrating container dependencies directly into the workflow.
Results

After running the pipeline, you’ll generate the following reports:

Safety Report: A JSON file listing vulnerable dependencies.

Bandit Report: An HTML file with static analysis findings.

GitLeaks Report: The job summary is generated by GitLeaks’ internal implementation, which uses the GITHUB_STEP_SUMMARY environment variable.

ZAP Scan Report: An HTML file summarising dynamic application vulnerabilities.

Here’s what a successful pipeline run looks like:


This pipeline is intentionally kept flexible and educational, providing a foundation for further enhancement. For instance, we can add stricter rules to block deployments when critical vulnerabilities are found or expand it with tools like Trivy for container scanning and KubeSec for Kubernetes security checks.
