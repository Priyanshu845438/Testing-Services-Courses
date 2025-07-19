
# Security Testing 🔐

## What is Security Testing?

Security testing evaluates applications to identify vulnerabilities, security flaws, and potential threats. It ensures that software systems protect data and maintain functionality as intended, safeguarding against malicious attacks and unauthorized access.

## Key Concepts

### Types of Security Testing
- **Authentication Testing**: User login and access controls
- **Authorization Testing**: Permission and role-based access
- **Data Protection**: Encryption and data handling
- **Input Validation**: SQL injection, XSS prevention
- **Session Management**: Token handling and session security
- **Error Handling**: Information disclosure prevention

### Security Testing Approaches
- **Static Application Security Testing (SAST)**: Code analysis
- **Dynamic Application Security Testing (DAST)**: Runtime testing
- **Interactive Application Security Testing (IAST)**: Combination approach
- **Penetration Testing**: Simulated attacks
- **Vulnerability Assessment**: Systematic security evaluation

## Tools & Frameworks

### OWASP Tools
```bash
# OWASP ZAP (Zed Attack Proxy)
# Automated security testing
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t https://your-app.com \
  -r zap_report.html

# OWASP Dependency Check
dependency-check --project "MyApp" --scan ./src --format ALL
```

### SAST Tools
```python
# Bandit - Python security linter
# Install: pip install bandit

# Example security issues Bandit can catch
import subprocess
import os

# High severity: Shell injection vulnerability
def bad_command_execution(user_input):
    # BAD: Directly using user input in shell command
    subprocess.call(f"echo {user_input}", shell=True)  # Bandit will flag this

# Medium severity: Use of exec
def dangerous_exec(code):
    exec(code)  # Bandit will flag this

# Low severity: Hardcoded password
PASSWORD = "hardcoded_secret"  # Bandit will flag this

# Run: bandit -r . -f json -o security_report.json
```

### DAST Tools
```javascript
// Using Playwright for security testing
const { chromium } = require('playwright');

async function testXSS() {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  
  // Test XSS vulnerability
  const xssPayload = '<script>alert("XSS")</script>';
  
  await page.goto('https://your-app.com/search');
  await page.fill('#search-input', xssPayload);
  await page.click('#search-button');
  
  // Check if XSS payload was executed
  const alertFired = await page.evaluate(() => {
    return window.alertFired || false;
  });
  
  console.log(`XSS vulnerability: ${alertFired ? 'FOUND' : 'NOT FOUND'}`);
  
  await browser.close();
}
```

### Penetration Testing Tools
```bash
# Nmap - Network scanning
nmap -sV -sC -O target-ip

# Nikto - Web vulnerability scanner
nikto -h https://your-app.com

# SQLmap - SQL injection testing
sqlmap -u "https://your-app.com/search?id=1" --batch --risk=3 --level=5
```

## Real-World Examples

### Authentication Security Testing
```python
import requests
import pytest

class TestAuthentication:
    BASE_URL = "https://api.example.com"
    
    def test_login_with_valid_credentials(self):
        """Test successful login"""
        response = requests.post(f"{self.BASE_URL}/auth/login", json={
            "username": "testuser",
            "password": "ValidPass123!"
        })
        
        assert response.status_code == 200
        assert "access_token" in response.json()
        assert "refresh_token" in response.json()
    
    def test_login_with_invalid_credentials(self):
        """Test login failure with wrong credentials"""
        response = requests.post(f"{self.BASE_URL}/auth/login", json={
            "username": "testuser",
            "password": "wrongpassword"
        })
        
        assert response.status_code == 401
        assert "error" in response.json()
        # Ensure no sensitive information is leaked
        assert "password" not in response.text.lower()
    
    def test_brute_force_protection(self):
        """Test account lockout after multiple failed attempts"""
        for i in range(6):  # Attempt 6 failed logins
            response = requests.post(f"{self.BASE_URL}/auth/login", json={
                "username": "testuser",
                "password": f"wrongpass{i}"
            })
        
        # Account should be locked after 5 attempts
        assert response.status_code == 423  # Locked
        assert "account locked" in response.json()["error"].lower()
    
    def test_password_complexity_requirements(self):
        """Test password strength validation"""
        weak_passwords = [
            "123456",
            "password",
            "abc123",
            "qwerty",
            "short"
        ]
        
        for weak_pass in weak_passwords:
            response = requests.post(f"{self.BASE_URL}/auth/register", json={
                "username": "newuser",
                "password": weak_pass,
                "email": "test@example.com"
            })
            
            assert response.status_code == 400
            assert "password" in response.json()["error"].lower()
```

### SQL Injection Testing
```python
class TestSQLInjection:
    
    def test_sql_injection_in_search(self):
        """Test SQL injection vulnerability in search"""
        sql_payloads = [
            "'; DROP TABLE users; --",
            "' OR '1'='1",
            "' UNION SELECT * FROM users --",
            "admin'--",
            "' OR 1=1#"
        ]
        
        for payload in sql_payloads:
            response = requests.get(f"{self.BASE_URL}/search", params={
                "query": payload
            })
            
            # Application should handle malicious input gracefully
            assert response.status_code in [200, 400, 422]
            
            # Check for SQL error messages (shouldn't be exposed)
            error_keywords = ["sql", "mysql", "postgres", "sqlite", "syntax error"]
            response_text = response.text.lower()
            
            for keyword in error_keywords:
                assert keyword not in response_text, f"SQL error exposed: {keyword}"
    
    def test_parameterized_queries(self):
        """Verify parameterized queries are used"""
        # Test with legitimate single quotes in data
        legitimate_data = "O'Connor's Restaurant"
        
        response = requests.get(f"{self.BASE_URL}/search", params={
            "query": legitimate_data
        })
        
        # Should handle single quotes properly without breaking
        assert response.status_code == 200
```

### XSS (Cross-Site Scripting) Testing
```python
class TestXSS:
    
    def test_reflected_xss(self):
        """Test reflected XSS vulnerabilities"""
        xss_payloads = [
            "<script>alert('XSS')</script>",
            "<img src=x onerror=alert('XSS')>",
            "javascript:alert('XSS')",
            "<svg onload=alert('XSS')>",
            "';alert('XSS');//"
        ]
        
        for payload in xss_payloads:
            response = requests.get(f"{self.BASE_URL}/search", params={
                "query": payload
            })
            
            # Payload should be properly escaped/sanitized
            assert payload not in response.text
            
            # Check for HTML encoding
            encoded_chars = ["&lt;", "&gt;", "&quot;", "&#x27;"]
            has_encoding = any(char in response.text for char in encoded_chars)
            
            if "<script>" in payload:
                assert has_encoding or payload not in response.text
    
    def test_stored_xss(self):
        """Test stored XSS in user comments"""
        xss_payload = "<script>fetch('/admin/users').then(r=>r.text()).then(console.log)</script>"
        
        # Submit comment with XSS payload
        response = requests.post(f"{self.BASE_URL}/comments", json={
            "content": xss_payload,
            "post_id": 1
        }, headers={"Authorization": "Bearer valid_token"})
        
        assert response.status_code in [200, 201]
        
        # Retrieve comments and verify XSS is prevented
        comments_response = requests.get(f"{self.BASE_URL}/posts/1/comments")
        
        # XSS payload should be neutralized
        assert "<script>" not in comments_response.text
        assert xss_payload not in comments_response.text
```

### Authorization Testing
```python
class TestAuthorization:
    
    def test_admin_only_endpoint(self):
        """Test admin-only endpoint access control"""
        # Regular user token
        user_token = "user_jwt_token"
        
        response = requests.get(f"{self.BASE_URL}/admin/users", headers={
            "Authorization": f"Bearer {user_token}"
        })
        
        assert response.status_code == 403  # Forbidden
        assert "insufficient privileges" in response.json()["error"].lower()
    
    def test_user_data_access_control(self):
        """Test users can only access their own data"""
        user1_token = "user1_jwt_token"
        user2_id = "user2_id"
        
        # User 1 trying to access User 2's profile
        response = requests.get(f"{self.BASE_URL}/users/{user2_id}/profile", headers={
            "Authorization": f"Bearer {user1_token}"
        })
        
        assert response.status_code == 403
    
    def test_horizontal_privilege_escalation(self):
        """Test prevention of horizontal privilege escalation"""
        user_token = "regular_user_token"
        
        # Try to modify admin settings
        response = requests.put(f"{self.BASE_URL}/admin/settings", json={
            "maintenance_mode": True
        }, headers={
            "Authorization": f"Bearer {user_token}"
        })
        
        assert response.status_code == 403
```

## Resume Showcase Tips

### Strong Security Testing Experience
```
✅ Excellent Examples:
• "Implemented comprehensive security testing framework identifying 40+ vulnerabilities before production"
• "Conducted OWASP Top 10 security assessments reducing security risks by 85%"
• "Automated SAST/DAST security testing in CI/CD pipeline preventing 15+ security incidents"
• "Led penetration testing initiatives improving application security posture significantly"

✅ Technical Skills to Highlight:
• OWASP Top 10 knowledge
• Tools: OWASP ZAP, Burp Suite, Nessus, Veracode
• Vulnerability types: XSS, CSRF, SQL Injection, Authentication
• Security frameworks: SAST, DAST, IAST
• Compliance: PCI-DSS, GDPR, HIPAA security testing
```

### Quantifiable Achievements
- Number of vulnerabilities identified/fixed
- Security posture improvement percentages
- Compliance audit success rates
- Security incident prevention metrics
- Cost savings from early vulnerability detection

## Learning Path 📚

### Beginner (0-4 months)
1. **Security Fundamentals**
   ```
   Core Concepts:
   - CIA Triad (Confidentiality, Integrity, Availability)
   - OWASP Top 10 vulnerabilities
   - Basic authentication vs authorization
   - Input validation principles
   ```

2. **Manual Security Testing**
   - Browser developer tools for security testing
   - Basic SQL injection testing
   - XSS identification and testing
   - Authentication bypass attempts

3. **Security Tools Introduction**
   - OWASP ZAP basic usage
   - Burp Suite Community Edition
   - Browser security extensions

### Intermediate (4-8 months)
1. **Automated Security Testing**
   ```python
   # Security test automation with Python
   import requests
   from security_test_framework import SecurityTester
   
   class WebAppSecurityTests:
       def __init__(self):
           self.tester = SecurityTester()
       
       def run_owasp_top_10_tests(self):
           # Automated OWASP Top 10 testing
           pass
   ```

2. **Advanced Vulnerability Testing**
   - CSRF testing
   - Session management flaws
   - Business logic vulnerabilities
   - API security testing

3. **Security Tool Mastery**
   - Advanced Burp Suite techniques
   - Custom security test scripts
   - Integration with CI/CD pipelines

### Advanced (8+ months)
1. **Penetration Testing**
   - Network penetration testing
   - Web application pen testing
   - Mobile application security
   - Cloud security assessment

2. **Security Architecture**
   - Threat modeling
   - Security requirements analysis
   - Security design reviews
   - Risk assessment methodologies

3. **Compliance and Governance**
   - Security compliance testing
   - Audit preparation and execution
   - Security metrics and reporting
   - Incident response testing

## Best Practices

### Security Test Design
```python
# Good: Comprehensive security test
class SecurityTestSuite:
    def setup_method(self):
        self.app_url = "https://secure-app.example.com"
        self.test_data = self.load_security_payloads()
    
    def test_comprehensive_input_validation(self):
        """Test multiple input validation scenarios"""
        test_cases = [
            {"type": "sql_injection", "payloads": self.test_data["sql"]},
            {"type": "xss", "payloads": self.test_data["xss"]},
            {"type": "command_injection", "payloads": self.test_data["cmd"]},
        ]
        
        for test_case in test_cases:
            self.run_input_validation_tests(test_case)
    
    def verify_security_headers(self):
        """Verify essential security headers"""
        response = requests.get(self.app_url)
        
        required_headers = [
            "Content-Security-Policy",
            "X-Frame-Options",
            "X-Content-Type-Options",
            "Strict-Transport-Security"
        ]
        
        for header in required_headers:
            assert header in response.headers
```

### Vulnerability Management
- Prioritize vulnerabilities by risk level
- Implement fix verification testing
- Maintain vulnerability databases
- Regular security regression testing

### Reporting and Communication
```python
def generate_security_report(findings):
    """Generate comprehensive security report"""
    report = {
        "summary": {
            "total_vulnerabilities": len(findings),
            "critical": len([f for f in findings if f.severity == "critical"]),
            "high": len([f for f in findings if f.severity == "high"]),
            "medium": len([f for f in findings if f.severity == "medium"]),
            "low": len([f for f in findings if f.severity == "low"])
        },
        "findings": findings,
        "recommendations": generate_recommendations(findings)
    }
    
    return report
```

## Common Security Testing Challenges

### False Positives Management
```python
def validate_vulnerability(finding):
    """Verify if security finding is a real vulnerability"""
    validation_tests = {
        "sql_injection": validate_sql_injection,
        "xss": validate_xss,
        "csrf": validate_csrf
    }
    
    validator = validation_tests.get(finding.type)
    return validator(finding) if validator else False
```

### Environment-Specific Testing
- Different security configurations per environment
- Production-like security testing in staging
- Continuous security monitoring
- Security test data management

### Advanced Security Testing Framework
```python
# Comprehensive security testing suite
class AdvancedSecurityTestSuite:
    
    def __init__(self, base_url):
        self.base_url = base_url
        self.session = requests.Session()
        self.vulnerabilities = []
    
    def test_csrf_protection(self):
        """Test Cross-Site Request Forgery protection"""
        # Get CSRF token
        login_page = self.session.get(f"{self.base_url}/login")
        csrf_token = self.extract_csrf_token(login_page.text)
        
        # Test valid CSRF token
        valid_response = self.session.post(f"{self.base_url}/transfer", data={
            'amount': '100',
            'to_account': '12345',
            'csrf_token': csrf_token
        })
        
        # Test missing CSRF token
        invalid_response = self.session.post(f"{self.base_url}/transfer", data={
            'amount': '100',
            'to_account': '12345'
        })
        
        # Should reject request without valid CSRF token
        assert invalid_response.status_code in [403, 400]
        
        # Test invalid CSRF token
        forged_response = self.session.post(f"{self.base_url}/transfer", data={
            'amount': '100',
            'to_account': '12345',
            'csrf_token': 'invalid_token'
        })
        
        assert forged_response.status_code in [403, 400]
    
    def test_session_security(self):
        """Test session management security"""
        # Test session fixation
        session_before_login = self.session.cookies.get('sessionid')
        
        login_response = self.session.post(f"{self.base_url}/login", data={
            'username': 'testuser',
            'password': 'password123'
        })
        
        session_after_login = self.session.cookies.get('sessionid')
        
        # Session ID should change after login
        assert session_before_login != session_after_login
        
        # Test session timeout
        time.sleep(1)  # In real test, this would be longer
        protected_response = self.session.get(f"{self.base_url}/profile")
        
        # Should handle session timeout appropriately
        if protected_response.status_code == 401:
            assert "session expired" in protected_response.text.lower()
    
    def test_file_upload_security(self):
        """Test file upload security"""
        malicious_files = [
            ('shell.php', '<?php system($_GET["cmd"]); ?>', 'application/x-php'),
            ('script.js', 'alert("XSS")', 'application/javascript'),
            ('virus.exe', b'\x4d\x5a\x90\x00', 'application/x-executable')
        ]
        
        for filename, content, content_type in malicious_files:
            files = {'file': (filename, content, content_type)}
            
            response = self.session.post(
                f"{self.base_url}/upload",
                files=files
            )
            
            # Should reject malicious file types
            assert response.status_code in [400, 415, 422]
            
            # Check if file was actually rejected
            if response.status_code == 200:
                # Verify file is not accessible
                file_url = f"{self.base_url}/uploads/{filename}"
                access_response = requests.get(file_url)
                assert access_response.status_code == 404
    
    def test_directory_traversal(self):
        """Test directory traversal vulnerabilities"""
        traversal_payloads = [
            "../../../etc/passwd",
            "..\\..\\..\\windows\\system32\\drivers\\etc\\hosts",
            "....//....//....//etc/passwd",
            "%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd"
        ]
        
        for payload in traversal_payloads:
            response = self.session.get(f"{self.base_url}/files", params={
                'filename': payload
            })
            
            # Should not expose system files
            system_indicators = ['root:', 'bin:', 'daemon:', '[boot loader]']
            response_text = response.text.lower()
            
            for indicator in system_indicators:
                assert indicator not in response_text
    
    def test_information_disclosure(self):
        """Test for information disclosure vulnerabilities"""
        # Test error page information leakage
        error_response = self.session.get(f"{self.base_url}/nonexistent")
        
        sensitive_info = [
            'stacktrace',
            'database connection',
            'internal server error',
            'file path',
            'line number'
        ]
        
        for info in sensitive_info:
            assert info not in error_response.text.lower()
        
        # Test HTTP headers for information disclosure
        headers_to_check = [
            'server',
            'x-powered-by',
            'x-aspnet-version'
        ]
        
        for header in headers_to_check:
            if header in error_response.headers:
                # Should not reveal detailed version information
                header_value = error_response.headers[header].lower()
                version_patterns = [r'\d+\.\d+\.\d+', r'version \d+']
                
                for pattern in version_patterns:
                    import re
                    if re.search(pattern, header_value):
                        self.vulnerabilities.append(f"Version disclosure in {header} header")
```

### Automated Security Scanning
```python
# Automated security scanning integration
class AutomatedSecurityScanner:
    
    def __init__(self, target_url):
        self.target_url = target_url
        self.results = {}
    
    def run_nikto_scan(self):
        """Run Nikto web vulnerability scanner"""
        import subprocess
        
        nikto_cmd = [
            'nikto',
            '-h', self.target_url,
            '-Format', 'json',
            '-output', 'nikto_results.json'
        ]
        
        try:
            result = subprocess.run(nikto_cmd, capture_output=True, text=True, timeout=300)
            
            if result.returncode == 0:
                with open('nikto_results.json', 'r') as f:
                    self.results['nikto'] = json.load(f)
            
            return result.returncode == 0
            
        except subprocess.TimeoutExpired:
            return False
    
    def run_ssl_scan(self):
        """Run SSL/TLS security assessment"""
        import ssl
        import socket
        from urllib.parse import urlparse
        
        parsed_url = urlparse(self.target_url)
        hostname = parsed_url.hostname
        port = parsed_url.port or (443 if parsed_url.scheme == 'https' else 80)
        
        try:
            context = ssl.create_default_context()
            
            with socket.create_connection((hostname, port), timeout=10) as sock:
                with context.wrap_socket(sock, server_hostname=hostname) as ssock:
                    cert = ssock.getpeercert()
                    cipher = ssock.cipher()
                    version = ssock.version()
                    
                    self.results['ssl'] = {
                        'certificate': cert,
                        'cipher_suite': cipher,
                        'tls_version': version,
                        'cert_expiry': cert.get('notAfter'),
                        'issuer': cert.get('issuer')
                    }
                    
                    # Check for weak configurations
                    weak_configs = []
                    
                    if version in ['SSLv2', 'SSLv3', 'TLSv1', 'TLSv1.1']:
                        weak_configs.append(f"Weak TLS version: {version}")
                    
                    if cipher and 'RC4' in cipher[0]:
                        weak_configs.append("Weak cipher: RC4")
                    
                    self.results['ssl']['vulnerabilities'] = weak_configs
                    
            return True
            
        except Exception as e:
            self.results['ssl'] = {'error': str(e)}
            return False
    
    def run_dependency_check(self):
        """Check for vulnerable dependencies"""
        import subprocess
        
        # For Python projects
        safety_cmd = ['safety', 'check', '--json']
        
        try:
            result = subprocess.run(safety_cmd, capture_output=True, text=True)
            
            if result.stdout:
                self.results['dependencies'] = json.loads(result.stdout)
            
            return len(self.results.get('dependencies', [])) == 0
            
        except (subprocess.SubprocessError, json.JSONDecodeError):
            return False
    
    def generate_security_report(self):
        """Generate comprehensive security report"""
        report = {
            'target': self.target_url,
            'scan_date': datetime.now().isoformat(),
            'summary': {
                'total_vulnerabilities': 0,
                'critical': 0,
                'high': 0,
                'medium': 0,
                'low': 0
            },
            'findings': self.results
        }
        
        # Calculate vulnerability counts
        for scan_type, results in self.results.items():
            if isinstance(results, list):
                report['summary']['total_vulnerabilities'] += len(results)
        
        return report
```

### Security Testing in CI/CD
```yaml
# GitHub Actions security testing workflow
name: Security Testing

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.9'
    
    - name: Install security tools
      run: |
        pip install bandit safety semgrep
        sudo apt-get update
        sudo apt-get install -y nikto
    
    - name: Run Bandit security scan
      run: |
        bandit -r . -f json -o bandit-report.json
        bandit -r . -f txt
    
    - name: Run Safety dependency check
      run: |
        safety check --json --output safety-report.json
        safety check
    
    - name: Run Semgrep security scan
      run: |
        semgrep --config=auto --json --output=semgrep-report.json .
        semgrep --config=auto .
    
    - name: Upload security reports
      uses: actions/upload-artifact@v3
      with:
        name: security-reports
        path: |
          bandit-report.json
          safety-report.json
          semgrep-report.json
    
    - name: Security scan summary
      run: |
        echo "Security scan completed. Check artifacts for detailed reports."
        if [ -s bandit-report.json ]; then
          echo "Bandit found potential security issues"
        fi
        if [ -s safety-report.json ]; then
          echo "Safety found vulnerable dependencies"
        fi
```

### Penetration Testing Methodologies
```python
# Penetration testing framework
class PenetrationTestSuite:
    
    def __init__(self, target):
        self.target = target
        self.findings = []
    
    def reconnaissance_phase(self):
        """Information gathering phase"""
        import nmap
        
        nm = nmap.PortScanner()
        
        # Port scanning
        scan_results = nm.scan(self.target, '1-1000')
        
        open_ports = []
        for host in scan_results['scan'].keys():
            for port in scan_results['scan'][host]['tcp'].keys():
                if scan_results['scan'][host]['tcp'][port]['state'] == 'open':
                    open_ports.append({
                        'port': port,
                        'service': scan_results['scan'][host]['tcp'][port]['name'],
                        'version': scan_results['scan'][host]['tcp'][port].get('version', 'unknown')
                    })
        
        return {
            'open_ports': open_ports,
            'host_info': scan_results['scan']
        }
    
    def vulnerability_assessment(self):
        """Vulnerability assessment phase"""
        vulnerabilities = []
        
        # Web application scanning
        web_vulns = self.scan_web_vulnerabilities()
        vulnerabilities.extend(web_vulns)
        
        # Network service scanning
        network_vulns = self.scan_network_services()
        vulnerabilities.extend(network_vulns)
        
        return vulnerabilities
    
    def exploitation_phase(self):
        """Controlled exploitation phase"""
        # This should only be done in authorized environments
        exploitation_results = []
        
        # Test for common vulnerabilities
        if self.test_sql_injection():
            exploitation_results.append({
                'vulnerability': 'SQL Injection',
                'severity': 'Critical',
                'exploitable': True
            })
        
        if self.test_xss():
            exploitation_results.append({
                'vulnerability': 'Cross-Site Scripting',
                'severity': 'High',
                'exploitable': True
            })
        
        return exploitation_results
    
    def generate_pentest_report(self):
        """Generate penetration testing report"""
        report = {
            'executive_summary': self.create_executive_summary(),
            'technical_findings': self.findings,
            'risk_assessment': self.assess_risks(),
            'recommendations': self.generate_recommendations(),
            'methodology': 'OWASP Testing Guide v4.0',
            'scope': f'Web application: {self.target}',
            'test_date': datetime.now().strftime('%Y-%m-%d')
        }
        
        return report
```

---

**Next Step**: Move to cloud environments! Continue to [Cloud Testing](./06-cloud-testing.md) to learn AWS, GCP, disaster recovery, and multi-region testing strategies.
