# Security Scanning in CI/CD

### 1. SAST (Static Application Security Testing)
**Tool:** GitHub CodeQL

**Purpose:** Analyzes source code for security vulnerabilities without executing it

- **What it scans:**
  - Security vulnerabilities in Python code
  - Common coding mistakes that could lead to exploits
  - SQL injection, XSS, command injection patterns
  - Insecure cryptography usage
  - Path traversal vulnerabilities
  - Extended security queries for deeper analysis

- **When it runs:**
  - On every push to `main` branch
  - On every pull request to `main` branch

- **Configuration:** Uses `security-extended` query suite for comprehensive coverage

### 2. SCA (Software Composition Analysis)
**Tool:** OWASP Dependency-Check

**Purpose:** Identifies known vulnerabilities in project dependencies

- **What it scans:**
  - Third-party Python packages (from `requirements.txt`)
  - Known CVEs (Common Vulnerabilities and Exposures)
  - Retired/deprecated packages with security issues
  - Transitive dependencies

- **When it runs:**
  - On every push to `main` or `master` branch
  - On every pull request to `main` or `master` branch

- **Failure threshold:** CVSS score ≥ 7.0 (High and Critical vulnerabilities)

### 3. DAST (Dynamic Application Security Testing)
**Tool:** OWASP ZAP (Zed Attack Proxy)

**Purpose:** Tests the running application for runtime vulnerabilities

- **What it scans:**
  - HTTP security headers
  - Cross-Site Scripting (XSS)
  - SQL Injection
  - Cross-Site Request Forgery (CSRF)
  - Insecure cookies
  - Security misconfigurations
  - Information disclosure

- **When it runs:**
  - On every push to `main` or `master` branch
  - On every pull request to `main` or `master` branch

- **Scan types:**
  - **Baseline Scan:** Quick passive scan for common issues
  - **Full Scan:** Comprehensive active scan with crawling and attack simulation

## Workflow Files

Our security scanning is implemented in the following GitHub Actions workflows:

- [1-sast-only.yml](.github/workflows/1-sast-only.yml) - CodeQL static analysis only
- [2-sca-only.yml](.github/workflows/2-sca-only.yml) - OWASP Dependency-Check only
- [3-dast-only.yml](.github/workflows/3-dast-only.yml) - OWASP ZAP dynamic testing only
- [4-complete-security.yml](.github/workflows/4-complete-security.yml) - All three scans in sequence

### Complete Security Pipeline

The complete pipeline ([4-complete-security.yml](.github/workflows/4-complete-security.yml)) runs all scans in the following order:

1. **SAST** - CodeQL analyzes source code
2. **SCA** - Dependency-Check scans packages (runs after SAST)
3. **Build** - Verifies application builds correctly (runs after SAST & SCA pass)
4. **DAST** - ZAP scans running application (runs after successful build)
5. **Summary** - Generates consolidated security report

This staged approach ensures that code-level issues are caught before runtime testing.

## Interpreting Scan Results

### Understanding Severity Levels

#### CVSS Scores (SCA)
- **Critical (9.0-10.0):** Immediate action required
- **High (7.0-8.9):** Fix as soon as possible (workflow fails on these)
- **Medium (4.0-6.9):** Plan to fix in near future
- **Low (0.1-3.9):** Fix when convenient

#### CodeQL Alerts (SAST)
- **Error:** High-severity issues that should be fixed immediately
- **Warning:** Medium-severity issues requiring review
- **Note:** Low-severity suggestions for improvement

#### ZAP Alerts (DAST)
- **High:** Definite vulnerability needing immediate fix
- **Medium:** Likely security issue requiring investigation
- **Low:** Potential issue or best practice violation
- **Informational:** General security recommendations