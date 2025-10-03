# Universal CI/CD Pipeline

A flexible and customizable GitHub Actions CI/CD pipeline that can be adapted for various types of projects. This pipeline provides comprehensive testing, validation, and deployment workflows out of the box.

## 🚀 Features

- **Multi-Platform Testing**: Runs tests on Windows, Linux, and macOS
- **Multi-Version Support**: Tests across different Node.js versions
- **Automated Validation**: Code linting and type checking
- **Security Scanning**: Dependency vulnerability checks
- **Scheduled Runs**: Automated weekly security scans
- **Artifact Management**: Build artifacts storage and release packaging
- **Flexible Deployment**: Configurable deployment to multiple environments

## 📁 File Structure

```
.github/
└── workflows/
    └── universal-ci-cd.yml
```

## ⚙️ Configuration

### Environment Variables

The pipeline uses these default environment variables:

```yaml
env:
  NODE_VERSION: '20.x'
  PYTHON_VERSION: '3.11'
```

### Trigger Events

The workflow automatically runs on:

- **Push** to `main` or `develop` branches
- **Pull requests** to `main` branch
- **Scheduled** runs every Monday at 2 AM UTC

```yaml
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * 1'  # Weekly security scan
```

## 🔧 Jobs Overview

### 1. Validate Job
- **Purpose**: Code quality and static analysis
- **Runs**: On all triggers
- **Activities**:
  - Setup Node.js and Python environments
  - Install dependencies (npm & pip)
  - Run linting and type checking
  - Python code style validation with flake8

### 2. Test Job
- **Purpose**: Comprehensive testing across multiple environments
- **Runs**: After successful validation
- **Matrix Strategy**:
  - **OS**: Ubuntu, Windows, macOS
  - **Node.js**: Versions 18.x and 20.x
- **Activities**:
  - Install dependencies
  - Run test suite
  - Upload test results and coverage reports as artifacts

### 3. Build Job
- **Purpose**: Build and package the application
- **Runs**: After successful validation and testing
- **Activities**:
  - Build the project
  - Create release package (tar.gz)
  - Upload build artifacts

### 4. Deploy Job
- **Purpose**: Deployment to production environment
- **Runs**: Only on `main` branch after successful build
- **Activities**:
  - Download build artifacts
  - Deploy to production server
  - Create GitHub Release (on tags)

## 🛠️ Customization Guide

### For Node.js Projects

1. Update build commands in the `build` job:
```yaml
- name: Build project
  run: |
    npm run build
    # Add your specific build commands
```

2. Modify test commands if needed:
```yaml
- name: Install and test
  run: |
    npm ci
    npm run test:coverage  # Your custom test command
```

### For Python Projects

1. Add Python-specific steps in the `validate` job:
```yaml
- name: Run Python specific checks
  run: |
    python -m pylint src/
    python -m mypy src/
```

2. Update test commands:
```yaml
- name: Run Python tests
  run: |
    pytest --cov=src --cov-report=xml
```

### For Docker Projects

Add a container build job:
```yaml
docker-build:
  runs-on: ubuntu-latest
  steps:
    - name: Build Docker image
      run: docker build -t my-app .
```

### Environment-Specific Configurations

#### Staging Deployment
Add a staging environment deployment:

```yaml
deploy-staging:
  needs: build
  runs-on: ubuntu-latest
  if: github.ref == 'refs/heads/develop'
  environment: staging
  steps:
    # Staging deployment steps
```

#### Development Branch
Modify the trigger for feature branches:

```yaml
on:
  push:
    branches: [ main, develop, feature/* ]
```

## 🔐 Secrets Configuration

Set these secrets in your GitHub repository settings:

| Secret | Purpose |
|--------|---------|
| `DEPLOY_KEY` | SSH key for server deployment |
| `SERVER_URL` | Target deployment server URL |
| `SNYK_TOKEN` | Snyk security scanning token |
| `PYPI_TOKEN` | PyPI package deployment token |
| `DOCKER_REGISTRY_TOKEN` | Container registry access token |

## 📊 Artifacts and Reports

The pipeline generates and stores:

- **Test Results**: Available in GitHub Actions artifacts
- **Coverage Reports**: Uploaded for code coverage analysis
- **Build Packages**: Release-ready tar.gz packages
- **Security Reports**: Vulnerability scan results

## 🎯 Usage Examples

### Basic Setup
1. Copy the workflow file to `.github/workflows/ci-cd.yml`
2. Push to your repository
3. Monitor runs in the "Actions" tab

### Custom Build Commands
Replace the build step with your project's build process:

```yaml
- name: Build project
  run: |
    npm run build:production
    # OR for other project types:
    # docker build -t my-app .
    # python setup.py sdist bdist_wheel
```

### Conditional Deployment
Add environment-specific conditions:

```yaml
deploy:
  if: |
    github.ref == 'refs/heads/main' &&
    github.event_name == 'push' &&
    !contains(github.event.head_commit.message, '[skip ci]')
```

## 🔍 Monitoring and Debugging

### Viewing Logs
- Go to repository → Actions → Select workflow run
- Click on individual jobs to see detailed logs

### Artifact Access
1. Navigate to workflow run
2. Scroll to "Artifacts" section
3. Download test results or build packages

### Common Issues

1. **Dependency Cache**: The workflow uses GitHub's cache action for faster runs
2. **Matrix Builds**: Adjust the matrix strategy based on your testing needs
3. **Timeout Issues**: Increase timeout for large projects if needed

## 📈 Performance Optimization

- **Caching**: Dependencies are cached between runs
- **Parallel Jobs**: Multiple jobs run in parallel when possible
- **Matrix Optimization**: Adjust OS/version matrix based on needs
- **Conditional Steps**: Skip unnecessary steps when possible

## 🤝 Contributing

To modify this pipeline for your project:

1. Fork the repository
2. Adjust the workflow file according to your needs
3. Test with feature branches before merging to main
4. Update this documentation accordingly

## 📄 License

This CI/CD template is open source and available under the MIT License.

---

**Note**: Always test the pipeline in a development environment before using it in production. Adjust resource limits and timeouts based on your project's requirements.
