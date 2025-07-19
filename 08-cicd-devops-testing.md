
# CI/CD & DevOps Testing 🚀

## What is CI/CD & DevOps Testing?

CI/CD & DevOps testing integrates automated testing throughout the software development lifecycle, ensuring rapid, reliable delivery while maintaining quality through continuous integration, continuous deployment, and infrastructure automation. It encompasses testing the entire delivery pipeline, from code commit to production deployment, including infrastructure, configurations, and deployment processes.

DevOps testing bridges the gap between development and operations teams, enabling faster feedback loops, early defect detection, and automated quality gates that prevent faulty code from reaching production environments.

## Key Concepts

### CI/CD Pipeline Testing Components
- **Continuous Integration (CI)**: Automated build, test, and code quality validation
- **Continuous Deployment (CD)**: Automated deployment across environments
- **Infrastructure as Code (IaC)**: Version-controlled infrastructure testing
- **Configuration Management**: Environment setup and validation testing
- **Pipeline Security**: Security scanning and vulnerability assessment
- **Rollback Testing**: Deployment failure recovery validation

### DevOps Testing Categories
- **Pipeline Testing**: CI/CD workflow validation and optimization
- **Environment Testing**: Infrastructure and configuration validation
- **Container Testing**: Docker, Kubernetes, and orchestration testing
- **Deployment Testing**: Blue-green, canary, and rolling deployment validation
- **Monitoring Testing**: Observability and alerting system validation
- **Chaos Engineering**: Resilience and failure recovery testing

### Testing Automation Levels
- **Unit Testing in Pipeline**: Fast feedback for individual components
- **Integration Testing**: Service and component interaction validation
- **End-to-End Testing**: Complete user workflow validation
- **Performance Testing**: Load and stress testing in pipeline
- **Security Testing**: SAST, DAST, and compliance validation

## Tools & Frameworks

### GitHub Actions Comprehensive Example
```yaml
# .github/workflows/comprehensive-pipeline.yml
name: Comprehensive CI/CD Pipeline

on:
  push:
    branches: [main, develop, 'feature/*']
  pull_request:
    branches: [main, develop]
  schedule:
    - cron: '0 2 * * *'  # Daily security scans

env:
  NODE_VERSION: '18'
  PYTHON_VERSION: '3.11'
  DOCKER_REGISTRY: 'ghcr.io'
  KUBECTL_VERSION: '1.28.0'

jobs:
  # ============ CODE QUALITY & SECURITY ============
  code-quality:
    runs-on: ubuntu-latest
    outputs:
      quality-gate: ${{ steps.quality-check.outputs.passed }}
    
    steps:
    - name: Checkout Repository
      uses: actions/checkout@v4
      with:
        fetch-depth: 0  # Full history for better analysis

    - name: Setup Python Environment
      uses: actions/setup-python@v4
      with:
        python-version: ${{ env.PYTHON_VERSION }}
        cache: 'pip'

    - name: Install Quality Tools
      run: |
        pip install --upgrade pip
        pip install black isort flake8 mypy bandit safety pylint
        pip install -r requirements.txt

    - name: Code Formatting Check
      run: |
        echo "::group::Black Formatting Check"
        black --check --diff src/ tests/
        echo "::endgroup::"
        
        echo "::group::Import Sorting Check"
        isort --check-only --diff src/ tests/
        echo "::endgroup::"

    - name: Linting and Type Checking
      run: |
        echo "::group::Flake8 Linting"
        flake8 src/ tests/ --statistics --tee --output-file=flake8-report.txt
        echo "::endgroup::"
        
        echo "::group::MyPy Type Checking"
        mypy src/ --html-report mypy-report
        echo "::endgroup::"
        
        echo "::group::Pylint Analysis"
        pylint src/ --output-format=text --reports=yes > pylint-report.txt || true
        echo "::endgroup::"

    - name: Security Scanning
      run: |
        echo "::group::Bandit Security Scan"
        bandit -r src/ -f json -o bandit-report.json
        bandit -r src/ -f txt
        echo "::endgroup::"
        
        echo "::group::Safety Dependency Check"
        safety check --json --output safety-report.json
        safety check --short-report
        echo "::endgroup::"

    - name: Upload Quality Reports
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: code-quality-reports
        path: |
          flake8-report.txt
          pylint-report.txt
          mypy-report/
          bandit-report.json
          safety-report.json

    - name: Quality Gate Decision
      id: quality-check
      run: |
        # Implement quality gate logic
        FLAKE8_ERRORS=$(grep -c "E[0-9]" flake8-report.txt || echo "0")
        BANDIT_ISSUES=$(jq '.results | length' bandit-report.json || echo "0")
        
        if [ "$FLAKE8_ERRORS" -gt 10 ] || [ "$BANDIT_ISSUES" -gt 0 ]; then
          echo "Quality gate failed - too many issues"
          echo "passed=false" >> $GITHUB_OUTPUT
          exit 1
        else
          echo "Quality gate passed"
          echo "passed=true" >> $GITHUB_OUTPUT
        fi

  # ============ COMPREHENSIVE TESTING ============
  test-suite:
    runs-on: ubuntu-latest
    needs: code-quality
    if: needs.code-quality.outputs.quality-gate == 'true'
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    strategy:
      matrix:
        test-type: [unit, integration, api, e2e]
      fail-fast: false

    steps:
    - name: Checkout Repository
      uses: actions/checkout@v4

    - name: Setup Test Environment
      uses: actions/setup-python@v4
      with:
        python-version: ${{ env.PYTHON_VERSION }}
        cache: 'pip'

    - name: Install Dependencies
      run: |
        pip install --upgrade pip
        pip install -r requirements.txt
        pip install -r requirements-test.txt
        pip install pytest pytest-cov pytest-xdist pytest-html pytest-json-report

    - name: Setup Test Database
      run: |
        export DATABASE_URL="postgresql://testuser:testpass@localhost:5432/testdb"
        python scripts/setup_test_db.py

    - name: Run Test Suite - ${{ matrix.test-type }}
      env:
        DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
        REDIS_URL: redis://localhost:6379
        TEST_TYPE: ${{ matrix.test-type }}
      run: |
        case "${{ matrix.test-type }}" in
          unit)
            pytest tests/unit/ -v --cov=src --cov-report=xml --cov-report=html \
              --html=unit-test-report.html --json-report --json-report-file=unit-results.json \
              -n auto --maxfail=5
            ;;
          integration)
            pytest tests/integration/ -v --tb=short \
              --html=integration-test-report.html --json-report --json-report-file=integration-results.json \
              --maxfail=3
            ;;
          api)
            pytest tests/api/ -v --tb=short \
              --html=api-test-report.html --json-report --json-report-file=api-results.json
            ;;
          e2e)
            pytest tests/e2e/ -v --tb=short \
              --html=e2e-test-report.html --json-report --json-report-file=e2e-results.json \
              --maxfail=1
            ;;
        esac

    - name: Upload Test Results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-results-${{ matrix.test-type }}
        path: |
          *-test-report.html
          *-results.json
          coverage.xml
          htmlcov/

    - name: Publish Test Results
      uses: dorny/test-reporter@v1
      if: always()
      with:
        name: ${{ matrix.test-type }}-tests
        path: '*-results.json'
        reporter: pytest-json

  # ============ CONTAINER BUILD & TESTING ============
  container-build:
    runs-on: ubuntu-latest
    needs: [code-quality, test-suite]
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}
    
    steps:
    - name: Checkout Repository
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.DOCKER_REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract Metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.DOCKER_REGISTRY }}/${{ github.repository }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}

    - name: Build and Export for Testing
      uses: docker/build-push-action@v5
      with:
        context: .
        target: test
        load: true
        tags: app:test
        cache-from: type=gha
        cache-to: type=gha,mode=max

    - name: Test Container Security
      run: |
        echo "::group::Container Structure Test"
        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
          -v $(pwd)/container-structure-test.yaml:/tests.yaml \
          gcr.io/gcp-runtimes/container-structure-test:latest \
          test --image app:test --config /tests.yaml
        echo "::endgroup::"
        
        echo "::group::Trivy Security Scan"
        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
          -v $(pwd):/workspace aquasec/trivy:latest \
          image --format json --output trivy-report.json app:test
        echo "::endgroup::"

    - name: Test Container Functionality
      run: |
        echo "::group::Container Functionality Test"
        # Start container
        docker run -d --name test-app -p 8080:8080 \
          -e DATABASE_URL=sqlite:///test.db app:test
        
        # Wait for startup
        timeout 60s bash -c 'while ! curl -sf http://localhost:8080/health; do sleep 2; done'
        
        # Run functionality tests
        curl -f http://localhost:8080/health
        curl -f http://localhost:8080/api/v1/status
        
        # Check container logs
        docker logs test-app
        
        # Cleanup
        docker stop test-app
        docker rm test-app
        echo "::endgroup::"

    - name: Build and Push Production Image
      id: build
      uses: docker/build-push-action@v5
      with:
        context: .
        target: production
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

    - name: Upload Security Reports
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: container-security-reports
        path: trivy-report.json

  # ============ INFRASTRUCTURE TESTING ============
  infrastructure-test:
    runs-on: ubuntu-latest
    needs: container-build
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    steps:
    - name: Checkout Repository
      uses: actions/checkout@v4

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3
      with:
        terraform_version: 1.6.0

    - name: Terraform Validation
      working-directory: infrastructure/
      run: |
        terraform fmt -check -recursive
        terraform init -backend=false
        terraform validate

    - name: Infrastructure Security Scan
      run: |
        echo "::group::Checkov Infrastructure Scan"
        pip install checkov
        checkov -d infrastructure/ --framework terraform \
          --output json --output-file checkov-report.json
        echo "::endgroup::"
        
        echo "::group::TFSec Security Scan"
        docker run --rm -v $(pwd):/src aquasec/tfsec:latest \
          /src/infrastructure --format json > tfsec-report.json
        echo "::endgroup::"

    - name: Plan Infrastructure Changes
      working-directory: infrastructure/
      env:
        TF_VAR_environment: staging
      run: |
        terraform init
        terraform plan -out=tfplan -var-file=staging.tfvars

    - name: Upload Infrastructure Reports
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: infrastructure-reports
        path: |
          checkov-report.json
          tfsec-report.json
          infrastructure/tfplan

  # ============ DEPLOYMENT TO STAGING ============
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [container-build, infrastructure-test]
    if: github.ref == 'refs/heads/main'
    environment: staging
    
    steps:
    - name: Checkout Repository
      uses: actions/checkout@v4

    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Setup Kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: ${{ env.KUBECTL_VERSION }}

    - name: Configure Kubernetes Context
      run: |
        aws eks update-kubeconfig --region us-east-1 --name staging-cluster

    - name: Deploy Infrastructure
      working-directory: infrastructure/
      run: |
        terraform init
        terraform apply -auto-approve -var-file=staging.tfvars

    - name: Deploy Application
      run: |
        # Update Kubernetes manifests with new image
        IMAGE_TAG="${{ needs.container-build.outputs.image-tag }}"
        sed -i "s|IMAGE_TAG_PLACEHOLDER|${IMAGE_TAG}|g" k8s/staging/deployment.yaml
        
        # Apply Kubernetes manifests
        kubectl apply -f k8s/staging/

    - name: Wait for Deployment
      run: |
        kubectl rollout status deployment/app -n staging --timeout=600s
        kubectl wait --for=condition=ready pod -l app=myapp -n staging --timeout=300s

    - name: Verify Deployment Health
      run: |
        # Get service endpoint
        ENDPOINT=$(kubectl get svc myapp-service -n staging -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
        
        # Wait for endpoint to be ready
        timeout 300s bash -c "while ! curl -sf http://${ENDPOINT}/health; do sleep 10; done"
        
        # Run health checks
        curl -f http://${ENDPOINT}/health
        curl -f http://${ENDPOINT}/api/v1/status

  # ============ STAGING TESTING ============
  staging-tests:
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment: staging
    
    strategy:
      matrix:
        test-suite: [smoke, api, performance, security]
      fail-fast: false

    steps:
    - name: Checkout Repository
      uses: actions/checkout@v4

    - name: Setup Test Environment
      uses: actions/setup-python@v4
      with:
        python-version: ${{ env.PYTHON_VERSION }}

    - name: Install Test Dependencies
      run: |
        pip install --upgrade pip
        pip install -r requirements-test.txt
        pip install locust pytest-html

    - name: Get Staging Endpoint
      run: |
        # Configure AWS and kubectl
        aws configure set region us-east-1
        aws eks update-kubeconfig --region us-east-1 --name staging-cluster
        
        # Get service endpoint
        ENDPOINT=$(kubectl get svc myapp-service -n staging -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
        echo "STAGING_ENDPOINT=http://${ENDPOINT}" >> $GITHUB_ENV

    - name: Run Test Suite - ${{ matrix.test-suite }}
      env:
        BASE_URL: ${{ env.STAGING_ENDPOINT }}
        TEST_ENVIRONMENT: staging
      run: |
        case "${{ matrix.test-suite }}" in
          smoke)
            pytest tests/smoke/ -v --html=smoke-report.html \
              --self-contained-html --tb=short
            ;;
          api)
            pytest tests/staging/api/ -v --html=api-staging-report.html \
              --self-contained-html
            ;;
          performance)
            locust -f tests/performance/locustfile.py --headless \
              --users 50 --spawn-rate 5 --run-time 300s \
              --host $BASE_URL --html performance-report.html
            ;;
          security)
            docker run --rm -v $(pwd):/zap/wrk/:rw \
              owasp/zap2docker-stable zap-baseline.py \
              -t $BASE_URL -J security-report.json
            ;;
        esac

    - name: Upload Test Results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: staging-test-results-${{ matrix.test-suite }}
        path: |
          *-report.html
          *-report.json

  # ============ PRODUCTION DEPLOYMENT ============
  deploy-production:
    runs-on: ubuntu-latest
    needs: [staging-tests]
    if: github.ref == 'refs/heads/main'
    environment: 
      name: production
      url: https://myapp.com
    
    steps:
    - name: Checkout Repository
      uses: actions/checkout@v4

    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID_PROD }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY_PROD }}
        aws-region: us-east-1

    - name: Setup Kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: ${{ env.KUBECTL_VERSION }}

    - name: Configure Production Kubernetes
      run: |
        aws eks update-kubeconfig --region us-east-1 --name production-cluster

    - name: Blue-Green Deployment
      run: |
        # Get current version
        CURRENT_VERSION=$(kubectl get deployment myapp -n production -o jsonpath='{.metadata.labels.version}')
        NEW_VERSION="${{ github.sha }}"
        
        # Update deployment with new image
        IMAGE_TAG="${{ needs.container-build.outputs.image-tag }}"
        sed -i "s|IMAGE_TAG_PLACEHOLDER|${IMAGE_TAG}|g" k8s/production/deployment.yaml
        sed -i "s|VERSION_PLACEHOLDER|${NEW_VERSION}|g" k8s/production/deployment.yaml
        
        # Deploy new version
        kubectl apply -f k8s/production/deployment.yaml
        
        # Wait for new deployment
        kubectl rollout status deployment/myapp -n production --timeout=600s
        
        # Health check
        kubectl wait --for=condition=ready pod -l app=myapp,version=${NEW_VERSION} -n production --timeout=300s

    - name: Production Health Verification
      run: |
        # Get production endpoint
        ENDPOINT=$(kubectl get svc myapp-service -n production -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
        
        # Comprehensive health checks
        for i in {1..10}; do
          if curl -sf https://${ENDPOINT}/health; then
            echo "Health check $i: PASS"
          else
            echo "Health check $i: FAIL"
            exit 1
          fi
          sleep 5
        done

    - name: Update Traffic Routing
      run: |
        # Update service selector to new version
        NEW_VERSION="${{ github.sha }}"
        kubectl patch service myapp-service -n production \
          -p '{"spec":{"selector":{"version":"'${NEW_VERSION}'"}}}'

    - name: Post-Deployment Monitoring
      run: |
        echo "Deployment completed successfully"
        echo "New version: ${{ github.sha }}"
        echo "Image: ${{ needs.container-build.outputs.image-tag }}"

  # ============ ROLLBACK CAPABILITY ============
  rollback:
    runs-on: ubuntu-latest
    if: failure()
    needs: [deploy-production]
    environment: production
    
    steps:
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID_PROD }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY_PROD }}
        aws-region: us-east-1

    - name: Setup Kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: ${{ env.KUBECTL_VERSION }}

    - name: Configure Kubernetes
      run: |
        aws eks update-kubeconfig --region us-east-1 --name production-cluster

    - name: Rollback Deployment
      run: |
        echo "Rolling back production deployment"
        kubectl rollout undo deployment/myapp -n production
        kubectl rollout status deployment/myapp -n production --timeout=300s

    - name: Verify Rollback
      run: |
        # Get production endpoint
        ENDPOINT=$(kubectl get svc myapp-service -n production -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
        
        # Health check after rollback
        timeout 300s bash -c "while ! curl -sf https://${ENDPOINT}/health; do sleep 10; done"
        
        echo "Rollback completed successfully"

  # ============ NOTIFICATION & REPORTING ============
  notify:
    runs-on: ubuntu-latest
    needs: [deploy-production, staging-tests]
    if: always()
    
    steps:
    - name: Notify Slack
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        channel: '#deployments'
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        fields: repo,message,commit,author,action,eventName,ref,workflow
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

    - name: Create Deployment Summary
      run: |
        echo "## Deployment Summary" >> $GITHUB_STEP_SUMMARY
        echo "- **Environment**: Production" >> $GITHUB_STEP_SUMMARY
        echo "- **Commit**: ${{ github.sha }}" >> $GITHUB_STEP_SUMMARY
        echo "- **Image**: ${{ needs.container-build.outputs.image-tag }}" >> $GITHUB_STEP_SUMMARY
        echo "- **Status**: ${{ job.status }}" >> $GITHUB_STEP_SUMMARY
```

### Jenkins Advanced Pipeline
```groovy
// Jenkinsfile - Advanced CI/CD Pipeline
@Library('shared-pipeline-library') _

pipeline {
    agent none
    
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['staging', 'production'],
            description: 'Target deployment environment'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Skip test execution (emergency deployments only)'
        )
        booleanParam(
            name: 'FORCE_DEPLOY',
            defaultValue: false,
            description: 'Force deployment despite quality gate failures'
        )
    }
    
    environment {
        DOCKER_REGISTRY = 'your-registry.com'
        DOCKER_IMAGE = 'myapp'
        DOCKER_TAG = "${BUILD_NUMBER}-${GIT_COMMIT.take(8)}"
        KUBECONFIG = credentials('kubernetes-config')
        SONAR_TOKEN = credentials('sonar-token')
        SLACK_CHANNEL = '#ci-cd'
    }
    
    stages {
        stage('Preparation') {
            agent any
            steps {
                script {
                    // Load pipeline configuration
                    def config = readYaml file: '.jenkins/pipeline-config.yaml'
                    env.PIPELINE_CONFIG = writeJSON returnText: true, json: config
                    
                    // Set build description
                    currentBuild.description = "Branch: ${env.BRANCH_NAME}, Environment: ${params.ENVIRONMENT}"
                }
                
                // Clean workspace
                cleanWs()
                
                // Checkout code
                checkout scm
                
                // Cache dependencies
                cache(maxCacheSize: 250, caches: [
                    arbitraryFileCache(path: 'node_modules', fingerprints: [packageJsonFingerprint()]),
                    arbitraryFileCache(path: '.pip-cache', fingerprints: [requirementsTxtFingerprint()])
                ]) {
                    sh 'echo "Dependencies cached"'
                }
            }
            post {
                always {
                    // Archive pipeline configuration
                    archiveArtifacts artifacts: '.jenkins/pipeline-config.yaml', allowEmptyArchive: true
                }
            }
        }
        
        stage('Code Quality Analysis') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    args '-u root:root'
                }
            }
            
            when {
                not { params.SKIP_TESTS }
            }
            
            parallel {
                stage('Static Analysis') {
                    steps {
                        sh '''
                            pip install --upgrade pip
                            pip install flake8 mypy black isort bandit safety
                            pip install -r requirements.txt
                        '''
                        
                        script {
                            def qualityGate = [:]
                            
                            // Code formatting
                            try {
                                sh 'black --check --diff src/ tests/'
                                qualityGate.formatting = 'PASS'
                            } catch (Exception e) {
                                qualityGate.formatting = 'FAIL'
                                unstable("Code formatting issues found")
                            }
                            
                            // Linting
                            try {
                                sh 'flake8 src/ tests/ --max-complexity=10 --max-line-length=100'
                                qualityGate.linting = 'PASS'
                            } catch (Exception e) {
                                qualityGate.linting = 'FAIL'
                                unstable("Linting issues found")
                            }
                            
                            // Type checking
                            try {
                                sh 'mypy src/ --strict'
                                qualityGate.typing = 'PASS'
                            } catch (Exception e) {
                                qualityGate.typing = 'FAIL'
                                unstable("Type checking issues found")
                            }
                            
                            // Store quality gate results
                            writeJSON file: 'quality-gate.json', json: qualityGate
                        }
                    }
                }
                
                stage('Security Scan') {
                    steps {
                        sh '''
                            pip install bandit safety
                            pip install -r requirements.txt
                        '''
                        
                        script {
                            def securityResults = [:]
                            
                            // Bandit security scan
                            try {
                                sh 'bandit -r src/ -f json -o bandit-report.json'
                                sh 'bandit -r src/ -f txt'
                                
                                def banditReport = readJSON file: 'bandit-report.json'
                                securityResults.bandit = [
                                    issues: banditReport.results.size(),
                                    status: banditReport.results.size() == 0 ? 'PASS' : 'FAIL'
                                ]
                            } catch (Exception e) {
                                securityResults.bandit = [issues: -1, status: 'ERROR']
                            }
                            
                            // Safety dependency check
                            try {
                                sh 'safety check --json --output safety-report.json'
                                sh 'safety check'
                                
                                def safetyReport = readJSON file: 'safety-report.json'
                                securityResults.safety = [
                                    vulnerabilities: safetyReport.size(),
                                    status: safetyReport.size() == 0 ? 'PASS' : 'FAIL'
                                ]
                            } catch (Exception e) {
                                securityResults.safety = [vulnerabilities: -1, status: 'ERROR']
                            }
                            
                            writeJSON file: 'security-results.json', json: securityResults
                        }
                    }
                }
                
                stage('Dependency Analysis') {
                    steps {
                        sh '''
                            pip install pip-audit
                            pip install -r requirements.txt
                        '''
                        
                        script {
                            // Audit dependencies for vulnerabilities
                            try {
                                sh 'pip-audit --format=json --output=dependency-audit.json'
                                def auditResults = readJSON file: 'dependency-audit.json'
                                
                                if (auditResults.vulnerabilities.size() > 0) {
                                    unstable("Vulnerable dependencies found")
                                }
                            } catch (Exception e) {
                                unstable("Dependency audit failed")
                            }
                        }
                    }
                }
            }
            
            post {
                always {
                    // Archive quality reports
                    archiveArtifacts artifacts: '''
                        quality-gate.json,
                        security-results.json,
                        dependency-audit.json,
                        bandit-report.json,
                        safety-report.json
                    ''', allowEmptyArchive: true
                    
                    // Publish security scan results
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: '.',
                        reportFiles: 'bandit-report.json',
                        reportName: 'Security Scan Report'
                    ])
                }
            }
        }
        
        stage('Comprehensive Testing') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            
            when {
                not { params.SKIP_TESTS }
            }
            
            environment {
                DATABASE_URL = 'postgresql://testuser:testpass@postgres:5432/testdb'
                REDIS_URL = 'redis://redis:6379'
            }
            
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh '''
                            pip install --upgrade pip
                            pip install -r requirements.txt
                            pip install -r requirements-test.txt
                            pip install pytest pytest-cov pytest-xdist pytest-html
                        '''
                        
                        sh '''
                            pytest tests/unit/ -v \
                                --cov=src \
                                --cov-report=xml \
                                --cov-report=html \
                                --cov-report=term-missing \
                                --html=unit-test-report.html \
                                --self-contained-html \
                                --junitxml=unit-test-results.xml \
                                -n auto \
                                --maxfail=10
                        '''
                    }
                    
                    post {
                        always {
                            publishTestResults testResultsPattern: 'unit-test-results.xml'
                            publishCoverage adapters: [coberturaAdapter('coverage.xml')], sourceFileResolver: sourceFiles('STORE_LAST_BUILD')
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'htmlcov',
                                reportFiles: 'index.html',
                                reportName: 'Coverage Report'
                            ])
                        }
                    }
                }
                
                stage('Integration Tests') {
                    steps {
                        script {
                            // Start test services
                            sh '''
                                docker-compose -f docker-compose.test.yml up -d
                                sleep 30
                            '''
                            
                            try {
                                sh '''
                                    pip install --upgrade pip
                                    pip install -r requirements.txt
                                    pip install -r requirements-test.txt
                                    pip install pytest pytest-html
                                '''
                                
                                sh '''
                                    pytest tests/integration/ -v \
                                        --html=integration-test-report.html \
                                        --self-contained-html \
                                        --junitxml=integration-test-results.xml \
                                        --tb=short \
                                        --maxfail=5
                                '''
                            } finally {
                                // Cleanup test services
                                sh 'docker-compose -f docker-compose.test.yml down'
                            }
                        }
                    }
                    
                    post {
                        always {
                            publishTestResults testResultsPattern: 'integration-test-results.xml'
                        }
                    }
                }
                
                stage('API Tests') {
                    steps {
                        sh '''
                            pip install --upgrade pip
                            pip install requests pytest pytest-html
                        '''
                        
                        sh '''
                            pytest tests/api/ -v \
                                --html=api-test-report.html \
                                --self-contained-html \
                                --junitxml=api-test-results.xml
                        '''
                    }
                    
                    post {
                        always {
                            publishTestResults testResultsPattern: 'api-test-results.xml'
                        }
                    }
                }
            }
            
            post {
                always {
                    archiveArtifacts artifacts: '''
                        *-test-report.html,
                        *-test-results.xml,
                        coverage.xml,
                        htmlcov/**
                    ''', allowEmptyArchive: true
                }
            }
        }
        
        stage('Build & Package') {
            agent any
            
            steps {
                script {
                    // Build Docker image
                    def image = docker.build("${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}")
                    
                    // Test image
                    sh """
                        docker run --rm -d --name test-container -p 8080:8080 \
                            ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        sleep 15
                        curl -f http://localhost:8080/health || exit 1
                        docker stop test-container
                    """
                    
                    // Security scan image
                    sh """
                        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                            aquasec/trivy:latest image \
                            --format json \
                            --output trivy-report.json \
                            ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                    
                    // Push image
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-registry-credentials') {
                        image.push()
                        image.push('latest')
                    }
                    
                    // Store image info
                    writeJSON file: 'image-info.json', json: [
                        registry: DOCKER_REGISTRY,
                        image: DOCKER_IMAGE,
                        tag: DOCKER_TAG,
                        fullName: "${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                    ]
                }
            }
            
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.json,image-info.json', allowEmptyArchive: true
                }
            }
        }
        
        stage('Deploy to Staging') {
            agent any
            
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                    expression { params.ENVIRONMENT == 'staging' }
                }
            }
            
            steps {
                script {
                    // Deploy using Helm
                    sh """
                        helm upgrade --install myapp-staging ./helm/myapp \
                            --namespace staging \
                            --create-namespace \
                            --set image.repository=${DOCKER_REGISTRY}/${DOCKER_IMAGE} \
                            --set image.tag=${DOCKER_TAG} \
                            --set environment=staging \
                            --values helm/myapp/values-staging.yaml \
                            --wait \
                            --timeout=600s
                    """
                    
                    // Verify deployment
                    sh """
                        kubectl rollout status deployment/myapp-staging -n staging --timeout=300s
                        kubectl wait --for=condition=ready pod -l app=myapp -n staging --timeout=300s
                    """
                }
            }
        }
        
        stage('Staging Tests') {
            agent any
            
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                    expression { params.ENVIRONMENT == 'staging' }
                }
            }
            
            parallel {
                stage('Smoke Tests') {
                    steps {
                        script {
                            def stagingUrl = sh(
                                script: "kubectl get ingress myapp-staging -n staging -o jsonpath='{.spec.rules[0].host}'",
                                returnStdout: true
                            ).trim()
                            
                            sh """
                                pip install requests pytest pytest-html
                                export STAGING_URL=https://${stagingUrl}
                                pytest tests/smoke/ -v \
                                    --html=smoke-test-report.html \
                                    --self-contained-html
                            """
                        }
                    }
                }
                
                stage('Performance Tests') {
                    steps {
                        script {
                            def stagingUrl = sh(
                                script: "kubectl get ingress myapp-staging -n staging -o jsonpath='{.spec.rules[0].host}'",
                                returnStdout: true
                            ).trim()
                            
                            sh """
                                pip install locust
                                locust -f tests/performance/locustfile.py --headless \
                                    --users 20 --spawn-rate 2 --run-time 180s \
                                    --host https://${stagingUrl} \
                                    --html performance-report.html
                            """
                        }
                    }
                }
                
                stage('Security Tests') {
                    steps {
                        script {
                            def stagingUrl = sh(
                                script: "kubectl get ingress myapp-staging -n staging -o jsonpath='{.spec.rules[0].host}'",
                                returnStdout: true
                            ).trim()
                            
                            sh """
                                docker run --rm -v \$(pwd):/zap/wrk/:rw \
                                    owasp/zap2docker-stable zap-baseline.py \
                                    -t https://${stagingUrl} \
                                    -J zap-report.json \
                                    -r zap-report.html
                            """
                        }
                    }
                }
            }
            
            post {
                always {
                    archiveArtifacts artifacts: '''
                        smoke-test-report.html,
                        performance-report.html,
                        zap-report.html,
                        zap-report.json
                    ''', allowEmptyArchive: true
                }
            }
        }
        
        stage('Production Deployment') {
            agent any
            
            when {
                allOf {
                    branch 'main'
                    anyOf {
                        expression { params.ENVIRONMENT == 'production' }
                        expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
                    }
                }
            }
            
            steps {
                script {
                    // Manual approval for production
                    timeout(time: 15, unit: 'MINUTES') {
                        input message: 'Deploy to Production?', 
                              ok: 'Deploy',
                              parameters: [
                                  choice(name: 'DEPLOYMENT_STRATEGY', 
                                         choices: ['blue-green', 'rolling', 'canary'], 
                                         description: 'Deployment strategy')
                              ]
                    }
                    
                    // Blue-Green Deployment
                    if (env.DEPLOYMENT_STRATEGY == 'blue-green') {
                        sh """
                            # Deploy to blue environment
                            helm upgrade --install myapp-blue ./helm/myapp \
                                --namespace production \
                                --set image.repository=${DOCKER_REGISTRY}/${DOCKER_IMAGE} \
                                --set image.tag=${DOCKER_TAG} \
                                --set environment=production \
                                --set deployment.strategy=blue \
                                --values helm/myapp/values-production.yaml \
                                --wait \
                                --timeout=600s
                        """
                        
                        // Health check
                        sh """
                            kubectl rollout status deployment/myapp-blue -n production --timeout=300s
                            kubectl wait --for=condition=ready pod -l app=myapp,version=blue -n production --timeout=300s
                        """
                        
                        // Switch traffic
                        sh """
                            kubectl patch service myapp-service -n production \
                                -p '{"spec":{"selector":{"version":"blue"}}}'
                        """
                        
                        // Cleanup old version
                        sh """
                            kubectl delete deployment myapp-green -n production --ignore-not-found=true
                        """
                    }
                    
                    // Rolling Deployment
                    else if (env.DEPLOYMENT_STRATEGY == 'rolling') {
                        sh """
                            helm upgrade --install myapp-production ./helm/myapp \
                                --namespace production \
                                --set image.repository=${DOCKER_REGISTRY}/${DOCKER_IMAGE} \
                                --set image.tag=${DOCKER_TAG} \
                                --set environment=production \
                                --set deployment.strategy=rolling \
                                --values helm/myapp/values-production.yaml \
                                --wait \
                                --timeout=600s
                        """
                    }
                    
                    // Canary Deployment
                    else if (env.DEPLOYMENT_STRATEGY == 'canary') {
                        sh """
                            # Deploy canary version (10% traffic)
                            helm upgrade --install myapp-canary ./helm/myapp \
                                --namespace production \
                                --set image.repository=${DOCKER_REGISTRY}/${DOCKER_IMAGE} \
                                --set image.tag=${DOCKER_TAG} \
                                --set environment=production \
                                --set deployment.strategy=canary \
                                --set deployment.canaryWeight=10 \
                                --values helm/myapp/values-production.yaml \
                                --wait \
                                --timeout=600s
                        """
                        
                        // Monitor canary for 10 minutes
                        sleep(time: 600, unit: 'SECONDS')
                        
                        // Promote canary to 100%
                        sh """
                            helm upgrade myapp-canary ./helm/myapp \
                                --namespace production \
                                --set deployment.canaryWeight=100 \
                                --reuse-values \
                                --wait
                        """
                    }
                }
            }
        }
        
        stage('Production Verification') {
            agent any
            
            when {
                branch 'main'
            }
            
            steps {
                script {
                    def productionUrl = sh(
                        script: "kubectl get ingress myapp-production -n production -o jsonpath='{.spec.rules[0].host}'",
                        returnStdout: true
                    ).trim()
                    
                    // Health checks
                    sh """
                        for i in {1..10}; do
                            echo "Health check attempt \$i"
                            curl -f https://${productionUrl}/health
                            curl -f https://${productionUrl}/api/v1/status
                            sleep 30
                        done
                    """
                    
                    // Monitoring alerts check
                    sh """
                        python scripts/check_monitoring_alerts.py --environment production
                    """
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Generate deployment report
                def deploymentReport = [
                    buildNumber: BUILD_NUMBER,
                    gitCommit: GIT_COMMIT,
                    branch: BRANCH_NAME,
                    environment: params.ENVIRONMENT,
                    dockerImage: "${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}",
                    deploymentTime: new Date().toString(),
                    buildStatus: currentBuild.result ?: 'SUCCESS'
                ]
                
                writeJSON file: 'deployment-report.json', json: deploymentReport
                archiveArtifacts artifacts: 'deployment-report.json'
                
                // Send notifications
                if (currentBuild.result == 'FAILURE') {
                    slackSend(
                        channel: SLACK_CHANNEL,
                        color: 'danger',
                        message: """
                            🚨 *Deployment Failed*
                            Job: ${env.JOB_NAME}
                            Build: ${env.BUILD_NUMBER}
                            Branch: ${env.BRANCH_NAME}
                            Commit: ${env.GIT_COMMIT.take(8)}
                            Environment: ${params.ENVIRONMENT}
                        """
                    )
                } else if (currentBuild.result == 'SUCCESS') {
                    slackSend(
                        channel: SLACK_CHANNEL,
                        color: 'good',
                        message: """
                            ✅ *Deployment Successful*
                            Job: ${env.JOB_NAME}
                            Build: ${env.BUILD_NUMBER}
                            Branch: ${env.BRANCH_NAME}
                            Commit: ${env.GIT_COMMIT.take(8)}
                            Environment: ${params.ENVIRONMENT}
                            Image: ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        """
                    )
                }
            }
        }
        
        failure {
            script {
                // Automatic rollback for production failures
                if (params.ENVIRONMENT == 'production' && env.BRANCH_NAME == 'main') {
                    try {
                        sh """
                            echo "Initiating automatic rollback"
                            helm rollback myapp-production -n production
                            kubectl rollout status deployment/myapp-production -n production --timeout=300s
                        """
                        
                        slackSend(
                            channel: SLACK_CHANNEL,
                            color: 'warning',
                            message: "🔄 Automatic rollback completed for production deployment"
                        )
                    } catch (Exception e) {
                        slackSend(
                            channel: SLACK_CHANNEL,
                            color: 'danger',
                            message: "❌ Automatic rollback failed. Manual intervention required!"
                        )
                    }
                }
            }
        }
        
        cleanup {
            // Clean workspace
            cleanWs()
        }
    }
}
```

### Infrastructure Testing with Terraform
```python
# infrastructure_test.py - Comprehensive Infrastructure Testing
import boto3
import json
import subprocess
import pytest
import time
from typing import Dict, List, Any
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class InfrastructureTestSuite:
    """Comprehensive infrastructure testing suite"""
    
    def __init__(self, terraform_dir: str = "./terraform"):
        self.terraform_dir = terraform_dir
        self.aws_session = boto3.Session()
        self.terraform_outputs = {}
        
    def setup_class(self):
        """Setup infrastructure for testing"""
        logger.info("Setting up infrastructure for testing")
        
        # Initialize Terraform
        self.run_terraform_command(['init'])
        
        # Plan infrastructure
        self.run_terraform_command(['plan', '-out=tfplan'])
        
        # Apply infrastructure
        self.run_terraform_command(['apply', 'tfplan'])
        
        # Get outputs
        self.terraform_outputs = self.get_terraform_outputs()
        
    def teardown_class(self):
        """Cleanup infrastructure after testing"""
        logger.info("Cleaning up infrastructure")
        self.run_terraform_command(['destroy', '-auto-approve'])
    
    def run_terraform_command(self, args: List[str]) -> subprocess.CompletedProcess:
        """Run terraform command with error handling"""
        cmd = ['terraform'] + args
        
        result = subprocess.run(
            cmd,
            cwd=self.terraform_dir,
            capture_output=True,
            text=True,
            check=True
        )
        
        if result.stderr:
            logger.warning(f"Terraform stderr: {result.stderr}")
            
        return result
    
    def get_terraform_outputs(self) -> Dict[str, Any]:
        """Get all terraform outputs"""
        result = self.run_terraform_command(['output', '-json'])
        outputs = json.loads(result.stdout)
        
        # Extract values from terraform output format
        return {k: v['value'] for k, v in outputs.items()}
    
    def test_vpc_configuration(self):
        """Test VPC and networking configuration"""
        vpc_id = self.terraform_outputs.get('vpc_id')
        assert vpc_id, "VPC ID should be available in outputs"
        
        ec2 = self.aws_session.client('ec2')
        
        # Test VPC exists and is available
        vpc_response = ec2.describe_vpcs(VpcIds=[vpc_id])
        vpc = vpc_response['Vpcs'][0]
        
        assert vpc['State'] == 'available', "VPC should be in available state"
        assert vpc['CidrBlock'] == '10.0.0.0/16', "VPC should have correct CIDR block"
        assert vpc['EnableDnsHostnames'] == True, "DNS hostnames should be enabled"
        assert vpc['EnableDnsSupport'] == True, "DNS support should be enabled"
        
        # Test VPC tags
        vpc_tags = {tag['Key']: tag['Value'] for tag in vpc.get('Tags', [])}
        assert 'Environment' in vpc_tags, "VPC should have Environment tag"
        assert 'Name' in vpc_tags, "VPC should have Name tag"
        
        logger.info(f"✅ VPC {vpc_id} configuration test passed")
    
    def test_subnet_configuration(self):
        """Test subnet configuration and distribution"""
        subnet_ids = self.terraform_outputs.get('private_subnet_ids', [])
        public_subnet_ids = self.terraform_outputs.get('public_subnet_ids', [])
        
        assert len(subnet_ids) >= 2, "Should have at least 2 private subnets"
        assert len(public_subnet_ids) >= 2, "Should have at least 2 public subnets"
        
        ec2 = self.aws_session.client('ec2')
        
        # Test private subnets
        private_response = ec2.describe_subnets(SubnetIds=subnet_ids)
        private_subnets = private_response['Subnets']
        
        # Verify subnets are in different AZs
        private_azs = {subnet['AvailabilityZone'] for subnet in private_subnets}
        assert len(private_azs) >= 2, "Private subnets should span multiple AZs"
        
        # Test subnet CIDR allocation
        expected_private_cidrs = ['10.0.1.0/24', '10.0.2.0/24', '10.0.3.0/24']
        actual_private_cidrs = [subnet['CidrBlock'] for subnet in private_subnets]
        
        for expected_cidr in expected_private_cidrs[:len(actual_private_cidrs)]:
            assert expected_cidr in actual_private_cidrs, f"Expected CIDR {expected_cidr} not found"
        
        # Test public subnets
        public_response = ec2.describe_subnets(SubnetIds=public_subnet_ids)
        public_subnets = public_response['Subnets']
        
        # Verify public subnets have internet gateway route
        for subnet in public_subnets:
            route_tables = ec2.describe_route_tables(
                Filters=[
                    {'Name': 'association.subnet-id', 'Values': [subnet['SubnetId']]}
                ]
            )
            
            has_igw_route = False
            for rt in route_tables['RouteTables']:
                for route in rt['Routes']:
                    if route.get('GatewayId', '').startswith('igw-'):
                        has_igw_route = True
                        break
            
            assert has_igw_route, f"Public subnet {subnet['SubnetId']} should have IGW route"
        
        logger.info("✅ Subnet configuration test passed")
    
    def test_security_groups(self):
        """Test security group configurations"""
        sg_ids = self.terraform_outputs.get('security_group_ids', [])
        assert sg_ids, "Security group IDs should be available"
        
        ec2 = self.aws_session.client('ec2')
        response = ec2.describe_security_groups(GroupIds=sg_ids)
        
        for sg in response['SecurityGroups']:
            # Test no overly permissive rules
            for rule in sg['IpPermissions']:
                for ip_range in rule.get('IpRanges', []):
                    if ip_range.get('CidrIp') == '0.0.0.0/0':
                        # Only allow HTTP/HTTPS from anywhere
                        allowed_ports = [80, 443]
                        port = rule.get('FromPort', rule.get('IpProtocol'))
                        
                        if isinstance(port, int):
                            assert port in allowed_ports, \
                                f"Port {port} should not be open to 0.0.0.0/0 in SG {sg['GroupId']}"
            
            # Test security group has proper tags
            sg_tags = {tag['Key']: tag['Value'] for tag in sg.get('Tags', [])}
            assert 'Environment' in sg_tags, f"Security group {sg['GroupId']} should have Environment tag"
        
        logger.info("✅ Security groups configuration test passed")
    
    def test_load_balancer_configuration(self):
        """Test Application Load Balancer setup"""
        alb_arn = self.terraform_outputs.get('alb_arn')
        target_group_arns = self.terraform_outputs.get('target_group_arns', [])
        
        assert alb_arn, "ALB ARN should be available in outputs"
        assert target_group_arns, "Target group ARNs should be available"
        
        elbv2 = self.aws_session.client('elbv2')
        
        # Test load balancer configuration
        lb_response = elbv2.describe_load_balancers(LoadBalancerArns=[alb_arn])
        lb = lb_response['LoadBalancers'][0]
        
        assert lb['State']['Code'] == 'active', "Load balancer should be active"
        assert lb['Type'] == 'application', "Should be application load balancer"
        assert lb['Scheme'] == 'internet-facing', "Should be internet-facing"
        
        # Test load balancer security
        lb_sg_ids = lb['SecurityGroups']
        ec2 = self.aws_session.client('ec2')
        sg_response = ec2.describe_security_groups(GroupIds=lb_sg_ids)
        
        for sg in sg_response['SecurityGroups']:
            # Verify HTTPS listener exists
            inbound_rules = sg['IpPermissions']
            https_rule = next(
                (rule for rule in inbound_rules if rule.get('FromPort') == 443), 
                None
            )
            assert https_rule, "Load balancer should allow HTTPS traffic"
        
        # Test target groups
        for tg_arn in target_group_arns:
            tg_response = elbv2.describe_target_groups(TargetGroupArns=[tg_arn])
            tg = tg_response['TargetGroups'][0]
            
            assert tg['Protocol'] in ['HTTP', 'HTTPS'], "Target group should use HTTP/HTTPS"
            assert tg['HealthCheckPath'], "Target group should have health check path"
            assert tg['HealthCheckIntervalSeconds'] <= 30, "Health check interval should be reasonable"
            
            # Test target health
            health_response = elbv2.describe_target_health(TargetGroupArn=tg_arn)
            
            if health_response['TargetHealthDescriptions']:
                healthy_targets = [
                    target for target in health_response['TargetHealthDescriptions']
                    if target['TargetHealth']['State'] == 'healthy'
                ]
                
                total_targets = len(health_response['TargetHealthDescriptions'])
                if total_targets > 0:
                    health_percentage = len(healthy_targets) / total_targets * 100
                    assert health_percentage >= 50, f"At least 50% of targets should be healthy, got {health_percentage}%"
        
        logger.info("✅ Load balancer configuration test passed")
    
    def test_rds_configuration(self):
        """Test RDS database configuration"""
        db_identifier = self.terraform_outputs.get('db_identifier')
        if not db_identifier:
            pytest.skip("No RDS database configured")
        
        rds = self.aws_session.client('rds')
        
        # Test database instance
        db_response = rds.describe_db_instances(DBInstanceIdentifier=db_identifier)
        db_instance = db_response['DBInstances'][0]
        
        assert db_instance['DBInstanceStatus'] == 'available', "Database should be available"
        assert db_instance['StorageEncrypted'] == True, "Database storage should be encrypted"
        assert db_instance['MultiAZ'] == True, "Database should be Multi-AZ for HA"
        assert db_instance['BackupRetentionPeriod'] >= 7, "Backup retention should be at least 7 days"
        
        # Test database security
        db_sg_ids = [sg['VpcSecurityGroupId'] for sg in db_instance['VpcSecurityGroups']]
        ec2 = self.aws_session.client('ec2')
        
        for sg_id in db_sg_ids:
            sg_response = ec2.describe_security_groups(GroupIds=[sg_id])
            sg = sg_response['SecurityGroups'][0]
            
            # Database should not be publicly accessible
            for rule in sg['IpPermissions']:
                for ip_range in rule.get('IpRanges', []):
                    assert ip_range.get('CidrIp') != '0.0.0.0/0', \
                        "Database should not be accessible from internet"
        
        assert not db_instance['PubliclyAccessible'], "Database should not be publicly accessible"
        
        logger.info("✅ RDS configuration test passed")
    
    def test_s3_bucket_configuration(self):
        """Test S3 bucket configuration and security"""
        bucket_names = self.terraform_outputs.get('s3_bucket_names', [])
        if not bucket_names:
            pytest.skip("No S3 buckets configured")
        
        s3 = self.aws_session.client('s3')
        
        for bucket_name in bucket_names:
            # Test bucket exists
            try:
                s3.head_bucket(Bucket=bucket_name)
            except Exception as e:
                pytest.fail(f"Bucket {bucket_name} should exist: {e}")
            
            # Test bucket encryption
            try:
                encryption_response = s3.get_bucket_encryption(Bucket=bucket_name)
                rules = encryption_response['ServerSideEncryptionConfiguration']['Rules']
                
                has_encryption = any(
                    rule['ApplyServerSideEncryptionByDefault']['SSEAlgorithm'] in ['AES256', 'aws:kms']
                    for rule in rules
                )
                assert has_encryption, f"Bucket {bucket_name} should have encryption enabled"
                
            except s3.exceptions.NoSuchBucket:
                pytest.fail(f"Bucket {bucket_name} should exist")
            except Exception:
                pytest.fail(f"Bucket {bucket_name} should have encryption configured")
            
            # Test bucket public access block
            try:
                pab_response = s3.get_public_access_block(Bucket=bucket_name)
                pab_config = pab_response['PublicAccessBlockConfiguration']
                
                assert pab_config['BlockPublicAcls'] == True, "Should block public ACLs"
                assert pab_config['IgnorePublicAcls'] == True, "Should ignore public ACLs"
                assert pab_config['BlockPublicPolicy'] == True, "Should block public policies"
                assert pab_config['RestrictPublicBuckets'] == True, "Should restrict public buckets"
                
            except Exception as e:
                pytest.fail(f"Bucket {bucket_name} should have public access block configured: {e}")
            
            # Test bucket versioning
            try:
                versioning_response = s3.get_bucket_versioning(Bucket=bucket_name)
                versioning_status = versioning_response.get('Status', 'Disabled')
                
                # For production buckets, versioning should be enabled
                if 'prod' in bucket_name.lower():
                    assert versioning_status == 'Enabled', f"Production bucket {bucket_name} should have versioning enabled"
                    
            except Exception as e:
                logger.warning(f"Could not check versioning for bucket {bucket_name}: {e}")
        
        logger.info("✅ S3 bucket configuration test passed")
    
    def test_iam_roles_and_policies(self):
        """Test IAM roles and policies configuration"""
        role_arns = self.terraform_outputs.get('iam_role_arns', [])
        if not role_arns:
            pytest.skip("No IAM roles configured")
        
        iam = self.aws_session.client('iam')
        
        for role_arn in role_arns:
            role_name = role_arn.split('/')[-1]
            
            # Test role exists
            try:
                role_response = iam.get_role(RoleName=role_name)
                role = role_response['Role']
                
                # Test assume role policy
                assume_role_policy = role['AssumeRolePolicyDocument']
                assert 'Statement' in assume_role_policy, "Role should have assume role policy"
                
                # Test role tags
                if 'Tags' in role:
                    role_tags = {tag['Key']: tag['Value'] for tag in role['Tags']}
                    assert 'Environment' in role_tags, f"Role {role_name} should have Environment tag"
                
            except iam.exceptions.NoSuchEntityException:
                pytest.fail(f"IAM role {role_name} should exist")
            
            # Test attached policies
            try:
                attached_policies_response = iam.list_attached_role_policies(RoleName=role_name)
                attached_policies = attached_policies_response['AttachedPolicies']
                
                # Check for overly permissive policies
                for policy in attached_policies:
                    policy_arn = policy['PolicyArn']
                    
                    # Skip AWS managed policies for this check
                    if policy_arn.startswith('arn:aws:iam::aws:policy/'):
                        continue
                    
                    # Get policy document for custom policies
                    try:
                        policy_response = iam.get_policy(PolicyArn=policy_arn)
                        version_id = policy_response['Policy']['DefaultVersionId']
                        
                        policy_version_response = iam.get_policy_version(
                            PolicyArn=policy_arn,
                            VersionId=version_id
                        )
                        
                        policy_document = policy_version_response['PolicyVersion']['Document']
                        
                        # Check for overly permissive statements
                        for statement in policy_document.get('Statement', []):
                            if statement.get('Effect') == 'Allow':
                                actions = statement.get('Action', [])
                                if isinstance(actions, str):
                                    actions = [actions]
                                
                                # Check for wildcard permissions
                                dangerous_actions = ['*', 'iam:*', 's3:*']
                                for action in actions:
                                    if action in dangerous_actions:
                                        resources = statement.get('Resource', [])
                                        if isinstance(resources, str):
                                            resources = [resources]
                                        
                                        # Only allow wildcard on specific resources
                                        if '*' in resources:
                                            logger.warning(f"Policy {policy_arn} has wildcard action {action} on wildcard resource")
                        
                    except Exception as e:
                        logger.warning(f"Could not analyze policy {policy_arn}: {e}")
                        
            except Exception as e:
                logger.warning(f"Could not check attached policies for role {role_name}: {e}")
        
        logger.info("✅ IAM roles and policies test passed")
    
    def test_cloudwatch_monitoring(self):
        """Test CloudWatch monitoring setup"""
        cloudwatch = self.aws_session.client('cloudwatch')
        logs = self.aws_session.client('logs')
        
        # Test CloudWatch alarms
        alarms_response = cloudwatch.describe_alarms()
        alarms = alarms_response['MetricAlarms']
        
        if alarms:
            # Test that alarms have appropriate configuration
            for alarm in alarms:
                assert alarm['ActionsEnabled'] == True, f"Alarm {alarm['AlarmName']} should have actions enabled"
                assert alarm['AlarmActions'], f"Alarm {alarm['AlarmName']} should have alarm actions"
                
                # Test alarm thresholds are reasonable
                metric_name = alarm['MetricName']
                threshold = alarm['Threshold']
                
                if metric_name == 'CPUUtilization':
                    assert 50 <= threshold <= 90, f"CPU alarm threshold should be between 50-90%, got {threshold}%"
                elif metric_name == 'MemoryUtilization':
                    assert 70 <= threshold <= 95, f"Memory alarm threshold should be between 70-95%, got {threshold}%"
        
        # Test log groups
        log_groups_response = logs.describe_log_groups()
        log_groups = log_groups_response['logGroups']
        
        if log_groups:
            for log_group in log_groups:
                # Test log retention
                retention_days = log_group.get('retentionInDays', 0)
                if retention_days > 0:
                    assert retention_days >= 7, f"Log group {log_group['logGroupName']} retention should be at least 7 days"
                
                # Test log group encryption
                if 'kmsKeyId' in log_group:
                    assert log_group['kmsKeyId'], f"Log group {log_group['logGroupName']} should be encrypted"
        
        logger.info("✅ CloudWatch monitoring test passed")
    
    def test_disaster_recovery_setup(self):
        """Test disaster recovery configuration"""
        # Test backup configurations
        backup = self.aws_session.client('backup')
        
        try:
            backup_plans_response = backup.list_backup_plans()
            backup_plans = backup_plans_response['BackupPlansList']
            
            if backup_plans:
                for plan in backup_plans:
                    plan_details = backup.get_backup_plan(BackupPlanId=plan['BackupPlanId'])
                    
                    rules = plan_details['BackupPlan']['Rules']
                    for rule in rules:
                        # Test backup frequency
                        schedule = rule.get('ScheduleExpression', '')
                        assert schedule, f"Backup rule {rule['RuleName']} should have schedule"
                        
                        # Test backup lifecycle
                        lifecycle = rule.get('Lifecycle', {})
                        if 'DeleteAfterDays' in lifecycle:
                            assert lifecycle['DeleteAfterDays'] >= 30, "Backup retention should be at least 30 days"
                        
                        if 'MoveToColdStorageAfterDays' in lifecycle:
                            assert lifecycle['MoveToColdStorageAfterDays'] >= 30, "Cold storage transition should be at least 30 days"
        
        except Exception as e:
            logger.warning(f"Could not test backup configuration: {e}")
        
        # Test cross-region replication for S3
        s3_bucket_names = self.terraform_outputs.get('s3_bucket_names', [])
        s3 = self.aws_session.client('s3')
        
        for bucket_name in s3_bucket_names:
            if 'critical' in bucket_name.lower() or 'backup' in bucket_name.lower():
                try:
                    replication_response = s3.get_bucket_replication(Bucket=bucket_name)
                    replication_config = replication_response['ReplicationConfiguration']
                    
                    rules = replication_config['Rules']
                    assert len(rules) > 0, f"Critical bucket {bucket_name} should have replication rules"
                    
                    for rule in rules:
                        assert rule['Status'] == 'Enabled', f"Replication rule should be enabled for {bucket_name}"
                        
                        destination = rule['Destination']
                        dest_bucket = destination['Bucket']
                        
                        # Verify destination bucket is in different region
                        dest_region = dest_bucket.split(':::')[-1].split('-')[1:3]
                        current_region = self.aws_session.region_name
                        
                        assert '-'.join(dest_region) != current_region, \
                            f"Replication should be to different region for {bucket_name}"
                
                except s3.exceptions.NoSuchBucket:
                    pass  # Bucket might not exist in test
                except Exception as e:
                    logger.warning(f"Could not test replication for bucket {bucket_name}: {e}")
        
        logger.info("✅ Disaster recovery setup test passed")
    
    def test_cost_optimization(self):
        """Test cost optimization measures"""
        # Test EC2 instance right-sizing
        ec2 = self.aws_session.client('ec2')
        
        try:
            instances_response = ec2.describe_instances(
                Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
            )
            
            for reservation in instances_response['Reservations']:
                for instance in reservation['Instances']:
                    instance_type = instance['InstanceType']
                    
                    # For test/staging environments, ensure using cost-effective instance types
                    instance_tags = {tag['Key']: tag['Value'] for tag in instance.get('Tags', [])}
                    environment = instance_tags.get('Environment', '').lower()
                    
                    if environment in ['test', 'staging', 'dev']:
                        # Should use smaller instance types for non-production
                        assert 't3' in instance_type or 't2' in instance_type or 'm5' in instance_type, \
                            f"Non-production instance should use cost-effective type, got {instance_type}"
                    
                    # Check for unused instances (no tags or improper tagging)
                    required_tags = ['Environment', 'Project', 'Owner']
                    missing_tags = [tag for tag in required_tags if tag not in instance_tags]
                    
                    if missing_tags:
                        logger.warning(f"Instance {instance['InstanceId']} missing tags: {missing_tags}")
        
        except Exception as e:
            logger.warning(f"Could not test EC2 cost optimization: {e}")
        
        # Test S3 storage classes
        s3_bucket_names = self.terraform_outputs.get('s3_bucket_names', [])
        s3 = self.aws_session.client('s3')
        
        for bucket_name in s3_bucket_names:
            try:
                # Check for lifecycle policies
                lifecycle_response = s3.get_bucket_lifecycle_configuration(Bucket=bucket_name)
                rules = lifecycle_response['Rules']
                
                has_transition_rule = any(
                    'Transitions' in rule for rule in rules
                )
                
                if not has_transition_rule:
                    logger.warning(f"Bucket {bucket_name} should have lifecycle transition rules for cost optimization")
                
            except s3.exceptions.NoSuchLifecycleConfiguration:
                logger.warning(f"Bucket {bucket_name} should have lifecycle configuration")
            except Exception as e:
                logger.warning(f"Could not test lifecycle for bucket {bucket_name}: {e}")
        
        logger.info("✅ Cost optimization test passed")

# Test execution
if __name__ == "__main__":
    import sys
    
    # Configure test environment
    test_suite = InfrastructureTestSuite()
    
    try:
        # Setup infrastructure
        test_suite.setup_class()
        
        # Run all tests
        test_methods = [
            test_suite.test_vpc_configuration,
            test_suite.test_subnet_configuration,
            test_suite.test_security_groups,
            test_suite.test_load_balancer_configuration,
            test_suite.test_rds_configuration,
            test_suite.test_s3_bucket_configuration,
            test_suite.test_iam_roles_and_policies,
            test_suite.test_cloudwatch_monitoring,
            test_suite.test_disaster_recovery_setup,
            test_suite.test_cost_optimization
        ]
        
        passed_tests = 0
        failed_tests = 0
        
        for test_method in test_methods:
            try:
                test_method()
                passed_tests += 1
                logger.info(f"✅ {test_method.__name__} PASSED")
            except Exception as e:
                failed_tests += 1
                logger.error(f"❌ {test_method.__name__} FAILED: {e}")
        
        # Summary
        total_tests = passed_tests + failed_tests
        logger.info(f"\n📊 Test Summary: {passed_tests}/{total_tests} tests passed")
        
        if failed_tests > 0:
            logger.error(f"❌ {failed_tests} tests failed")
            sys.exit(1)
        else:
            logger.info("🎉 All infrastructure tests passed!")
            
    except Exception as e:
        logger.error(f"💥 Infrastructure testing failed: {e}")
        sys.exit(1)
        
    finally:
        # Cleanup
        try:
            test_suite.teardown_class()
        except Exception as e:
            logger.error(f"Cleanup failed: {e}")
```

## Real-World Examples

### Container Testing Automation
```python
# container_testing.py - Comprehensive Container Testing
import docker
import subprocess
import json
import time
import requests
import pytest
from typing import Dict, List, Any
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class ContainerTestSuite:
    """Comprehensive container testing suite"""
    
    def __init__(self):
        self.docker_client = docker.from_env()
        self.test_containers = []
        
    def teardown_method(self):
        """Cleanup test containers"""
        for container in self.test_containers:
            try:
                container.stop()
                container.remove()
            except Exception as e:
                logger.warning(f"Failed to cleanup container {container.id}: {e}")
        self.test_containers.clear()
    
    def test_dockerfile_best_practices(self):
        """Test Dockerfile follows best practices"""
        with open('Dockerfile', 'r') as f:
            dockerfile_content = f.read()
        
        lines = dockerfile_content.split('\n')
        
        # Test 1: Should not run as root
        has_user_instruction = any('USER ' in line and 'USER root' not in line for line in lines)
        assert has_user_instruction, "Dockerfile should specify non-root user"
        
        # Test 2: Should use specific base image tags
        from_lines = [line for line in lines if line.startswith('FROM ')]
        for from_line in from_lines:
            assert ':latest' not in from_line, "Should not use 'latest' tag for base images"
            assert ':' in from_line, "Should specify explicit tag for base images"
        
        # Test 3: Should minimize layers
        run_lines = [line for line in lines if line.startswith('RUN ')]
        assert len(run_lines) <= 5, "Should minimize number of RUN instructions"
        
        # Test 4: Should have health check
        has_healthcheck = any('HEALTHCHECK' in line for line in lines)
        assert has_healthcheck, "Dockerfile should include HEALTHCHECK instruction"
        
        # Test 5: Should expose minimal ports
        expose_lines = [line for line in lines if line.startswith('EXPOSE ')]
        exposed_ports = []
        for line in expose_lines:
            ports = line.replace('EXPOSE ', '').split()
            exposed_ports.extend(ports)
        
        assert len(exposed_ports) <= 2, "Should expose minimal number of ports"
        
        logger.info("✅ Dockerfile best practices test passed")
    
    def test_image_build_optimization(self):
        """Test Docker image build optimization"""
        # Build image with build args
        image, build_logs = self.docker_client.images.build(
            path='.',
            tag='myapp:test',
            rm=True,
            pull=True,
            nocache=True,
            buildargs={
                'BUILD_DATE': str(int(time.time())),
                'VERSION': '1.0.0-test'
            }
        )
        
        # Test image size
        image_size_mb = image.attrs['Size'] / (1024 * 1024)
        assert image_size_mb < 500, f"Image size should be under 500MB, got {image_size_mb:.1f}MB"
        
        # Test image layers
        history = image.history()
        assert len(history) < 20, f"Image should have fewer than 20 layers, got {len(history)}"
        
        # Test for security vulnerabilities using Trivy
        try:
            trivy_result = subprocess.run([
                'docker', 'run', '--rm', '-v', '/var/run/docker.sock:/var/run/docker.sock',
                'aquasec/trivy:latest', 'image', '--format', 'json', 'myapp:test'
            ], capture_output=True, text=True, check=True)
            
            vulnerabilities = json.loads(trivy_result.stdout)
            
            # Count high and critical vulnerabilities
            high_critical_vulns = 0
            for result in vulnerabilities.get('Results', []):
                for vuln in result.get('Vulnerabilities', []):
                    if vuln['Severity'] in ['HIGH', 'CRITICAL']:
                        high_critical_vulns += 1
            
            assert high_critical_vulns < 5, f"Too many high/critical vulnerabilities: {high_critical_vulns}"
            
        except subprocess.CalledProcessError as e:
            logger.warning(f"Trivy scan failed: {e}")
        except FileNotFoundError:
            logger.warning("Trivy not available, skipping vulnerability scan")
        
        logger.info("✅ Image build optimization test passed")
    
    def test_container_runtime_security(self):
        """Test container runtime security"""
        # Start container with security options
        container = self.docker_client.containers.run(
            'myapp:test',
            detach=True,
            ports={'8080/tcp': '8080'},
            environment={
                'ENV': 'test',
                'DEBUG': 'false'
            },
            security_opt=['no-new-privileges:true'],
            read_only=True,
            tmpfs={'/tmp': 'noexec,nosuid,size=100m'},
            mem_limit='512m',
            cpu_quota=50000,  # 50% of one CPU
            remove=True
        )
        
        self.test_containers.append(container)
        
        # Wait for container to start
        time.sleep(10)
        
        # Test container is not running as root
        exec_result = container.exec_run('whoami')
        user = exec_result.output.decode().strip()
        assert user != 'root', f"Container should not run as root, running as: {user}"
        
        # Test container cannot escalate privileges
        try:
            exec_result = container.exec_run('sudo whoami')
            # If this succeeds, it's a security issue
            assert exec_result.exit_code != 0, "Container should not have sudo access"
        except Exception:
            pass  # Expected - no sudo should be available
        
        # Test filesystem is read-only (except allowed paths)
        exec_result = container.exec_run('touch /test-file')
        assert exec_result.exit_code != 0, "Root filesystem should be read-only"
        
        # Test can write to tmp
        exec_result = container.exec_run('touch /tmp/test-file')
        assert exec_result.exit_code == 0, "Should be able to write to /tmp"
        
        # Test memory limits
        stats = container.stats(stream=False)
        memory_limit = stats['memory_stats']['limit']
        expected_limit = 512 * 1024 * 1024  # 512MB in bytes
        assert memory_limit <= expected_limit * 1.1, "Memory limit should be enforced"
        
        logger.info("✅ Container runtime security test passed")
    
    def test_container_networking(self):
        """Test container networking configuration"""
        # Create custom network
        network = self.docker_client.networks.create(
            "test-network",
            driver="bridge",
            options={"com.docker.network.bridge.enable_icc": "false"}
        )
        
        try:
            # Start application container
            app_container = self.docker_client.containers.run(
                'myapp:test',
                detach=True,
                network=network.name,
                name='test-app',
                environment={'ENV': 'test'}
            )
            self.test_containers.append(app_container)
            
            # Start database container
            db_container = self.docker_client.containers.run(
                'postgres:13',
                detach=True,
                network=network.name,
                name='test-db',
                environment={
                    'POSTGRES_DB': 'testdb',
                    'POSTGRES_USER': 'testuser',
                    'POSTGRES_PASSWORD': 'testpass'
                }
            )
            self.test_containers.append(db_container)
            
            time.sleep(15)
            
            # Test containers can communicate by name
            exec_result = app_container.exec_run('nc -z test-db 5432')
            assert exec_result.exit_code == 0, "App container should reach database"
            
            # Test containers cannot reach external networks inappropriately
            exec_result = app_container.exec_run('nc -z 8.8.8.8 53', timeout=5)
            # This test depends on your security requirements
            
            # Test port isolation
            app_container.reload()
            db_container.reload()
            
            app_ports = app_container.attrs['NetworkSettings']['Ports']
            db_ports = db_container.attrs['NetworkSettings']['Ports']
            
            # Database should not expose ports to host
            for port, bindings in db_ports.items():
                if bindings:
                    logger.warning(f"Database port {port} is exposed to host")
        
        finally:
            # Cleanup network
            try:
                network.remove()
            except Exception as e:
                logger.warning(f"Failed to remove test network: {e}")
        
        logger.info("✅ Container networking test passed")
    
    def test_container_performance(self):
        """Test container performance characteristics"""
        # Start container with performance monitoring
        container = self.docker_client.containers.run(
            'myapp:test',
            detach=True,
            ports={'8080/tcp': '8080'},
            environment={'ENV': 'test'},
            mem_limit='1g',
            cpu_quota=100000  # 100% of one CPU
        )
        
        self.test_containers.append(container)
        
        # Wait for startup
        start_time = time.time()
        startup_timeout = 60
        
        while time.time() - start_time < startup_timeout:
            try:
                response = requests.get('http://localhost:8080/health', timeout=2)
                if response.status_code == 200:
                    break
            except requests.exceptions.RequestException:
                pass
            time.sleep(2)
        
        startup_time = time.time() - start_time
        assert startup_time < 30, f"Container should start within 30 seconds, took {startup_time:.1f}s"
        
        # Test response times under light load
        response_times = []
        for _ in range(10):
            start = time.time()
            response = requests.get('http://localhost:8080/health', timeout=5)
            response_time = time.time() - start
            
            assert response.status_code == 200, "Health check should return 200"
            response_times.append(response_time)
            time.sleep(0.1)
        
        avg_response_time = sum(response_times) / len(response_times)
        assert avg_response_time < 0.5, f"Average response time should be under 500ms, got {avg_response_time:.3f}s"
        
        # Test memory usage
        stats = container.stats(stream=False)
        memory_usage = stats['memory_stats']['usage']
        memory_limit = stats['memory_stats']['limit']
        memory_percent = (memory_usage / memory_limit) * 100
        
        assert memory_percent < 80, f"Memory usage should be under 80%, got {memory_percent:.1f}%"
        
        # Test CPU usage (basic check)
        cpu_stats = stats['cpu_stats']
        if 'cpu_usage' in cpu_stats and 'total_usage' in cpu_stats['cpu_usage']:
            logger.info(f"CPU usage stats collected: {cpu_stats['cpu_usage']['total_usage']}")
        
        logger.info("✅ Container performance test passed")
    
    def test_container_persistence(self):
        """Test container data persistence and volumes"""
        # Create volume for persistent data
        volume = self.docker_client.volumes.create(name='test-data-volume')
        
        try:
            # Start container with volume
            container1 = self.docker_client.containers.run(
                'myapp:test',
                detach=True,
                volumes={'test-data-volume': {'bind': '/app/data', 'mode': 'rw'}},
                environment={'ENV': 'test'},
                name='test-persistence-1'
            )
            
            time.sleep(5)
            
            # Write test data
            test_data = "persistent test data"
            exec_result = container1.exec_run(f'sh -c "echo \'{test_data}\' > /app/data/test.txt"')
            assert exec_result.exit_code == 0, "Should be able to write to persistent volume"
            
            # Stop and remove container
            container1.stop()
            container1.remove()
            
            # Start new container with same volume
            container2 = self.docker_client.containers.run(
                'myapp:test',
                detach=True,
                volumes={'test-data-volume': {'bind': '/app/data', 'mode': 'rw'}},
                environment={'ENV': 'test'},
                name='test-persistence-2'
            )
            
            self.test_containers.append(container2)
            time.sleep(5)
            
            # Verify data persisted
            exec_result = container2.exec_run('cat /app/data/test.txt')
            assert exec_result.exit_code == 0, "Should be able to read from persistent volume"
            
            retrieved_data = exec_result.output.decode().strip()
            assert retrieved_data == test_data, f"Data should persist, expected '{test_data}', got '{retrieved_data}'"
            
        finally:
            # Cleanup volume
            try:
                volume.remove()
            except Exception as e:
                logger.warning(f"Failed to remove test volume: {e}")
        
        logger.info("✅ Container persistence test passed")
    
    def test_multi_container_application(self):
        """Test multi-container application deployment"""
        # Use docker-compose for complex scenarios
        compose_file = '''
version: '3.8'
services:
  app:
    image: myapp:test
    ports:
      - "8080:8080"
    environment:
      - ENV=test
      - DATABASE_URL=postgresql://testuser:testpass@db:5432/testdb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    
  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=testdb
      - POSTGRES_USER=testuser
      - POSTGRES_PASSWORD=testpass
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U testuser -d testdb"]
      interval: 30s
      timeout: 10s
      retries: 3
    
  redis:
    image: redis:7
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3
'''
        
        with open('docker-compose.test.yml', 'w') as f:
            f.write(compose_file)
        
        try:
            # Start multi-container application
            subprocess.run([
                'docker-compose', '-f', 'docker-compose.test.yml', 'up', '-d'
            ], check=True)
            
            # Wait for all services to be healthy
            max_wait = 120
            start_time = time.time()
            
            while time.time() - start_time < max_wait:
                try:
                    # Check app health
                    response = requests.get('http://localhost:8080/health', timeout=5)
                    if response.status_code == 200:
                        health_data = response.json()
                        
                        # Verify all dependencies are healthy
                        if (health_data.get('database') == 'connected' and
                            health_data.get('redis') == 'connected'):
                            break
                except requests.exceptions.RequestException:
                    pass
                
                time.sleep(5)
            else:
                pytest.fail("Multi-container application failed to become healthy")
            
            # Test application functionality
            response = requests.get('http://localhost:8080/api/test', timeout=10)
            assert response.status_code == 200, "Application API should be accessible"
            
            # Test database connection
            response = requests.get('http://localhost:8080/api/db-test', timeout=10)
            assert response.status_code == 200, "Database connection should work"
            
            # Test Redis connection
            response = requests.get('http://localhost:8080/api/cache-test', timeout=10)
            assert response.status_code == 200, "Redis connection should work"
            
        finally:
            # Cleanup
            try:
                subprocess.run([
                    'docker-compose', '-f', 'docker-compose.test.yml', 'down', '-v'
                ], check=True)
            except Exception as e:
                logger.warning(f"Failed to cleanup docker-compose: {e}")
            
            try:
                import os
                os.remove('docker-compose.test.yml')
            except Exception as e:
                logger.warning(f"Failed to remove test compose file: {e}")
        
        logger.info("✅ Multi-container application test passed")

# Kubernetes Testing
class KubernetesTestSuite:
    """Kubernetes deployment testing suite"""
    
    def __init__(self, namespace: str = "test"):
        self.namespace = namespace
        try:
            from kubernetes import client, config
            config.load_kube_config()
            self.apps_v1 = client.AppsV1Api()
            self.core_v1 = client.CoreV1Api()
            self.networking_v1 = client.NetworkingV1Api()
        except ImportError:
            pytest.skip("Kubernetes client not available")
        except Exception as e:
            pytest.skip(f"Kubernetes not available: {e}")
    
    def test_deployment_rollout(self):
        """Test Kubernetes deployment rollout"""
        deployment_name = 'test-app'
        
        # Create deployment manifest
        deployment_manifest = {
            'apiVersion': 'apps/v1',
            'kind': 'Deployment',
            'metadata': {
                'name': deployment_name,
                'namespace': self.namespace,
                'labels': {'app': 'test-app'}
            },
            'spec': {
                'replicas': 3,
                'selector': {'matchLabels': {'app': 'test-app'}},
                'template': {
                    'metadata': {'labels': {'app': 'test-app'}},
                    'spec': {
                        'containers': [{
                            'name': 'app',
                            'image': 'myapp:test',
                            'ports': [{'containerPort': 8080}],
                            'resources': {
                                'requests': {'memory': '128Mi', 'cpu': '100m'},
                                'limits': {'memory': '512Mi', 'cpu': '500m'}
                            },
                            'livenessProbe': {
                                'httpGet': {'path': '/health', 'port': 8080},
                                'initialDelaySeconds': 30,
                                'periodSeconds': 10
                            },
                            'readinessProbe': {
                                'httpGet': {'path': '/ready', 'port': 8080},
                                'initialDelaySeconds': 5,
                                'periodSeconds': 5
                            }
                        }]
                    }
                }
            }
        }
        
        try:
            # Create namespace if it doesn't exist
            try:
                self.core_v1.create_namespace(
                    body={'metadata': {'name': self.namespace}}
                )
            except Exception:
                pass  # Namespace might already exist
            
            # Create deployment
            self.apps_v1.create_namespaced_deployment(
                namespace=self.namespace,
                body=deployment_manifest
            )
            
            # Wait for deployment to be ready
            timeout = 300  # 5 minutes
            start_time = time.time()
            
            while time.time() - start_time < timeout:
                deployment = self.apps_v1.read_namespaced_deployment(
                    name=deployment_name,
                    namespace=self.namespace
                )
                
                if (deployment.status.ready_replicas == deployment.spec.replicas and
                    deployment.status.available_replicas == deployment.spec.replicas):
                    break
                
                time.sleep(10)
            else:
                pytest.fail("Deployment failed to become ready within timeout")
            
            # Verify all pods are running
            pods = self.core_v1.list_namespaced_pod(
                namespace=self.namespace,
                label_selector='app=test-app'
            )
            
            assert len(pods.items) == 3, "Should have 3 pods running"
            
            for pod in pods.items:
                assert pod.status.phase == 'Running', f"Pod {pod.metadata.name} should be running"
                
                # Check container status
                for container_status in pod.status.container_statuses:
                    assert container_status.ready == True, f"Container should be ready in pod {pod.metadata.name}"
            
        finally:
            # Cleanup
            try:
                self.apps_v1.delete_namespaced_deployment(
                    name=deployment_name,
                    namespace=self.namespace
                )
            except Exception as e:
                logger.warning(f"Failed to cleanup deployment: {e}")
        
        logger.info("✅ Kubernetes deployment rollout test passed")
    
    def test_service_connectivity(self):
        """Test Kubernetes service connectivity"""
        service_name = 'test-service'
        deployment_name = 'test-app'
        
        # Service manifest
        service_manifest = {
            'apiVersion': 'v1',
            'kind': 'Service',
            'metadata': {
                'name': service_name,
                'namespace': self.namespace
            },
            'spec': {
                'selector': {'app': 'test-app'},
                'ports': [{'port': 80, 'targetPort': 8080}]
            }
        }
        
        try:
            # Create service
            self.core_v1.create_namespaced_service(
                namespace=self.namespace,
                body=service_manifest
            )
            
            # Wait for service to have endpoints
            timeout = 60
            start_time = time.time()
            
            while time.time() - start_time < timeout:
                endpoints = self.core_v1.read_namespaced_endpoints(
                    name=service_name,
                    namespace=self.namespace
                )
                
                if endpoints.subsets and endpoints.subsets[0].addresses:
                    break
                
                time.sleep(5)
            else:
                pytest.fail("Service endpoints not ready within timeout")
            
            # Test service from within cluster
            test_pod_manifest = {
                'apiVersion': 'v1',
                'kind': 'Pod',
                'metadata': {
                    'name': 'test-client',
                    'namespace': self.namespace
                },
                'spec': {
                    'containers': [{
                        'name': 'test',
                        'image': 'curlimages/curl:latest',
                        'command': ['sleep', '3600']
                    }],
                    'restartPolicy': 'Never'
                }
            }
            
            self.core_v1.create_namespaced_pod(
                namespace=self.namespace,
                body=test_pod_manifest
            )
            
            # Wait for test pod to be ready
            time.sleep(10)
            
            # Test service connectivity
            exec_command = [
                'curl', '-f', f'http://{service_name}.{self.namespace}.svc.cluster.local/health'
            ]
            
            from kubernetes.stream import stream
            
            response = stream(
                self.core_v1.connect_get_namespaced_pod_exec,
                'test-client',
                self.namespace,
                command=exec_command,
                stderr=True,
                stdin=False,
                stdout=True,
                tty=False
            )
            
            assert 'healthy' in response.lower() or '200' in response, "Service should be reachable and healthy"
            
        finally:
            # Cleanup
            try:
                self.core_v1.delete_namespaced_service(
                    name=service_name,
                    namespace=self.namespace
                )
                self.core_v1.delete_namespaced_pod(
                    name='test-client',
                    namespace=self.namespace
                )
            except Exception as e:
                logger.warning(f"Failed to cleanup service test resources: {e}")
        
        logger.info("✅ Kubernetes service connectivity test passed")

# Test execution
if __name__ == "__main__":
    container_suite = ContainerTestSuite()
    k8s_suite = KubernetesTestSuite()
    
    test_methods = [
        container_suite.test_dockerfile_best_practices,
        container_suite.test_image_build_optimization,
        container_suite.test_container_runtime_security,
        container_suite.test_container_networking,
        container_suite.test_container_performance,
        container_suite.test_container_persistence,
        container_suite.test_multi_container_application,
        k8s_suite.test_deployment_rollout,
        k8s_suite.test_service_connectivity
    ]
    
    passed = 0
    failed = 0
    
    for test_method in test_methods:
        try:
            test_method()
            passed += 1
            logger.info(f"✅ {test_method.__name__} PASSED")
        except Exception as e:
            failed += 1
            logger.error(f"❌ {test_method.__name__} FAILED: {e}")
        finally:
            # Cleanup after each test
            try:
                if hasattr(test_method, '__self__'):
                    test_method.__self__.teardown_method()
            except Exception:
                pass
    
    total = passed + failed
    logger.info(f"\n📊 Container Testing Summary: {passed}/{total} tests passed")
    
    if failed > 0:
        exit(1)
    else:
        logger.info("🎉 All container tests passed!")
```

## Resume Showcase Tips

### Strong CI/CD & DevOps Testing Experience
```
✅ Excellent Examples:
• "Architected end-to-end CI/CD pipeline reducing deployment time from 4 hours to 15 minutes with 99.5% success rate"
• "Implemented Infrastructure as Code testing framework preventing 95% of environment-related production issues"
• "Built comprehensive container testing suite covering security, performance, and compliance across 50+ microservices"
• "Designed blue-green deployment strategy with automated rollback, achieving zero-downtime deployments"
• "Created chaos engineering test suite improving system resilience by 80% through proactive failure testing"

✅ Technical Skills to Highlight:
• CI/CD Platforms: Jenkins, GitHub Actions, GitLab CI, Azure DevOps, CircleCI
• Infrastructure as Code: Terraform, CloudFormation, Pulumi, Ansible
• Containerization: Docker, Kubernetes, OpenShift, container security testing
• Cloud Platforms: AWS, GCP, Azure, multi-cloud deployment strategies
• Monitoring & Observability: Prometheus, Grafana, ELK stack, Jaeger, OpenTelemetry
• Security Integration: SAST, DAST, container scanning, policy as code
```

### Quantifiable Achievements
```
Deployment Metrics:
• "Increased deployment frequency from weekly to 50+ times per day"
• "Reduced mean time to recovery (MTTR) from 4 hours to 12 minutes"
• "Achieved 99.9% deployment success rate across 200+ services"
• "Decreased lead time from commit to production from 2 days to 2 hours"

Quality Improvements:
• "Prevented 300+ production issues through automated quality gates"
• "Reduced post-deployment defects by 85% through comprehensive testing"
• "Automated 90% of manual testing processes in CI/CD pipeline"
• "Improved test coverage from 60% to 95% through pipeline integration"

Cost & Efficiency:
• "Reduced infrastructure costs by 40% through automated resource optimization"
• "Decreased manual testing effort by 70% through automation"
• "Improved team productivity by 3x through self-service deployment capabilities"
```

## Learning Path 📚

### Beginner (0-4 months)
1. **CI/CD Fundamentals**
   ```
   Core Concepts:
   - Version control workflows (Git, GitHub Flow, GitLab Flow)
   - Continuous Integration principles and practices
   - Build automation and artifact management
   - Basic deployment strategies (rolling, blue-green)
   - Environment management (dev, staging, production)
   ```

2. **Pipeline Creation**
   ```yaml
   # Basic GitHub Actions pipeline
   name: Basic CI
   on: [push, pull_request]
   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         - name: Run tests
           run: npm test
   ```

3. **Essential Tools**
   - GitHub Actions or Jenkins basics
   - Docker fundamentals
   - Basic shell scripting
   - YAML configuration files

### Intermediate (4-8 months)
1. **Advanced Pipeline Design**
   ```python
   # Pipeline testing framework
   class PipelineValidator:
       def validate_pipeline_config(self, config):
           # Validate pipeline structure
           assert 'stages' in config
           assert 'test' in config['stages']
           assert 'deploy' in config['stages']
       
       def test_deployment_strategy(self, strategy):
           # Test deployment approach
           pass
   ```

2. **Infrastructure as Code**
   - Terraform basics and testing
   - Infrastructure validation and compliance
   - Environment provisioning automation
   - Configuration management

3. **Container Orchestration**
   - Kubernetes fundamentals
   - Helm charts and templating
   - Service mesh integration
   - Container security and scanning

### Advanced (8+ months)
1. **Enterprise DevOps**
   ```python
   # Advanced deployment patterns
   class AdvancedDeploymentStrategies:
       def canary_deployment(self, config):
           # Implement canary deployment logic
           pass
       
       def feature_flag_integration(self, flags):
           # Integrate with feature flag systems
           pass
       
       def chaos_engineering_tests(self):
           # Implement chaos experiments
           pass
   ```

2. **Platform Engineering**
   - Developer platform design
   - Self-service infrastructure
   - Multi-cloud strategies
   - Advanced monitoring and observability

3. **Site Reliability Engineering**
   - SLA/SLO implementation
   - Error budgets and policies
   - Incident response automation
   - Performance optimization

## Best Practices

### Pipeline Security
```yaml
# Secure pipeline example
name: Secure CI/CD
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Security Scan
        run: |
          # SAST scanning
          docker run --rm -v $(pwd):/code \
            securecodewarrior/semgrep-action:latest
          
          # Dependency scanning
          npm audit --audit-level high
          
          # Secret scanning
          docker run --rm -v $(pwd):/code \
            trufflesecurity/trufflehog:latest filesystem /code

  test-with-security:
    needs: security-scan
    runs-on: ubuntu-latest
    steps:
      - name: Run tests with security context
        run: |
          # Tests should include security validation
          pytest tests/ --security-tests
```

### Quality Gates
```python
# Quality gate implementation
class QualityGate:
    def __init__(self, config):
        self.config = config
        self.criteria = {
            'test_coverage': 80,
            'security_score': 90,
            'performance_score': 85,
            'code_quality_score': 85
        }
    
    def evaluate(self, metrics):
        """Evaluate if quality gate passes"""
        passed = True
        failures = []
        
        for criterion, threshold in self.criteria.items():
            if metrics.get(criterion, 0) < threshold:
                passed = False
                failures.append(f"{criterion}: {metrics.get(criterion, 0)} < {threshold}")
        
        return {
            'passed': passed,
            'failures': failures,
            'score': self.calculate_overall_score(metrics)
        }
    
    def calculate_overall_score(self, metrics):
        """Calculate weighted overall quality score"""
        weights = {
            'test_coverage': 0.3,
            'security_score': 0.3,
            'performance_score': 0.2,
            'code_quality_score': 0.2
        }
        
        weighted_score = sum(
            metrics.get(metric, 0) * weight
            for metric, weight in weights.items()
        )
        
        return weighted_score
```

### Monitoring and Observability
```python
# Pipeline monitoring
import prometheus_client
from prometheus_client import Counter, Histogram, Gauge

# Metrics
pipeline_runs = Counter('pipeline_runs_total', 'Total pipeline runs', ['status', 'branch'])
pipeline_duration = Histogram('pipeline_duration_seconds', 'Pipeline duration')
deployment_success_rate = Gauge('deployment_success_rate', 'Deployment success rate')

class PipelineMonitoring:
    def __init__(self):
        self.start_time = None
    
    def start_pipeline(self, branch):
        self.start_time = time.time()
        pipeline_runs.labels(status='started', branch=branch).inc()
    
    def finish_pipeline(self, status, branch):
        if self.start_time:
            duration = time.time() - self.start_time
            pipeline_duration.observe(duration)
        
        pipeline_runs.labels(status=status, branch=branch).inc()
    
    def update_deployment_success_rate(self, rate):
        deployment_success_rate.set(rate)
```

### Rollback Automation
```python
# Automated rollback system
class RollbackManager:
    def __init__(self, deployment_config):
        self.config = deployment_config
        self.health_check_url = deployment_config['health_check_url']
        self.rollback_threshold = deployment_config.get('error_threshold', 5)
    
    def monitor_deployment(self, deployment_id):
        """Monitor deployment health and trigger rollback if needed"""
        errors = 0
        max_errors = self.rollback_threshold
        
        for _ in range(10):  # Monitor for 10 iterations
            try:
                response = requests.get(self.health_check_url, timeout=10)
                if response.status_code != 200:
                    errors += 1
                    if errors >= max_errors:
                        self.trigger_rollback(deployment_id)
                        return False
                else:
                    errors = 0  # Reset error count on success
                
            except requests.exceptions.RequestException:
                errors += 1
                if errors >= max_errors:
                    self.trigger_rollback(deployment_id)
                    return False
            
            time.sleep(30)  # Check every 30 seconds
        
        return True  # Deployment is healthy
    
    def trigger_rollback(self, deployment_id):
        """Trigger automated rollback"""
        logger.error(f"Triggering rollback for deployment {deployment_id}")
        
        # Kubernetes rollback
        subprocess.run([
            'kubectl', 'rollout', 'undo', 
            f'deployment/{self.config["deployment_name"]}',
            '-n', self.config['namespace']
        ], check=True)
        
        # Notify team
        self.send_rollback_notification(deployment_id)
    
    def send_rollback_notification(self, deployment_id):
        """Send rollback notification to team"""
        # Send Slack, email, or other notifications
        pass
```

---

**Next Step**: Test mobile applications comprehensively! Continue to [Mobile App Testing](./09-mobile-app-testing.md) to learn Appium automation, real device testing, GPS functionality, and permission handling.
