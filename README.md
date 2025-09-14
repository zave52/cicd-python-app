# CI/CD Pipeline for Python Django Application

This project demonstrates a complete **CI/CD pipeline implementation** for a Python Django web application using GitHub Actions, Docker, and Kubernetes deployment with Helm charts. The main focus is on showcasing modern DevOps practices and automated deployment workflows.

## Project Purpose

This repository serves as a **demonstration of CI/CD best practices** featuring:
- GitHub Actions workflows for automated testing and deployment
- Multi-matrix CI builds across different Python versions and operating systems
- Docker containerization and automated image building
- Helm chart packaging and deployment automation
- Kubernetes deployment with proper environment management
- Artifact management and workflow orchestration

## Application Overview

A Django todo list application that includes:
- User authentication system (`accounts` app)
- Todo list management with CRUD operations (`lists` app)
- REST API endpoints (`api` app)
- Static file management and responsive UI
- Database integration with migration support

## Technologies Used

### Core Application
- **Python 3.8/3.9** - Application runtime with multi-version support
- **Django 4.x** - Web framework
- **Django REST Framework** - API development
- **MySQL** - Database (deployed via Helm sub-chart)

### CI/CD & DevOps
- **GitHub Actions** - CI/CD automation
- **Docker** - Application containerization
- **Helm 3.x** - Kubernetes package management
- **Kubernetes** - Container orchestration
- **Kind** - Local Kubernetes development

### Testing & Quality
- **Coverage.py** - Code coverage reporting
- **Flake8** - Code linting and style checking
- **Django Test Framework** - Unit and integration testing

## CI/CD Pipeline Architecture

### GitHub Actions Workflow (`.github/workflows/main.yml`)

#### 1. **Multi-Matrix CI (`python-ci` job)**
```yaml
strategy:
  matrix:
    PythonVersion: [ 3.8, 3.9 ]
    OsTypes: [ ubuntu-latest, windows-latest ]
```
- **Cross-platform testing** on Ubuntu and Windows
- **Multi-version Python support** (3.8, 3.9)
- **Automated dependency installation** and testing
- **Code coverage reporting** with coverage.py
- **Code quality checks** with flake8 linting
- **Complexity analysis** for maintainable code

#### 2. **Docker Build & Push (`docker-ci` job)**
- **Conditional execution** (only on main branch)
- **Artifact-based deployment** using matrix job outputs
- **Docker Hub integration** with secure credential management
- **Tagged releases** using commit SHA

#### 3. **Helm Chart CI (`helm-ci` job)**
- **Helm chart validation** with linting
- **Template verification** and testing
- **Chart packaging** for deployment
- **Artifact publishing** for deployment workflows

#### 4. **Multi-Environment Deployment**
- **Development environment** - Automatic deployment
- **Staging environment** - Promotion-based deployment
- **Reusable deployment workflows** for consistency

### Workflow Features
- **Concurrency control** - Prevents conflicting deployments
- **Manual workflow dispatch** - On-demand deployments with environment selection
- **Artifact management** - Efficient build artifact sharing between jobs
- **Environment-specific configurations** - Separate values for different stages

## Project Structure

```
├── .github/workflows/
│   ├── main.yml                             # Main CI/CD pipeline
│   └── reusable-deployment.yml              # Reusable deployment workflow
├── src/                                     # Django application source
│   ├── Dockerfile                           # Application containerization
│   ├── requirements.txt                     # Python dependencies
│   ├── manage.py                           # Django management script
│   ├── accounts/                           # User authentication app
│   ├── api/                                # REST API endpoints
│   ├── lists/                              # Todo list management
│   └── todolist/                           # Django project settings
├── helm-charts/todoapp/                     # Helm chart for Kubernetes
│   ├── Chart.yaml                          # Chart metadata
│   ├── values.yaml                         # Default configuration
│   ├── charts/mysql/                       # MySQL sub-chart
│   ├── templates/                          # Kubernetes manifests
│   └── values/stg.yaml                     # Staging environment config
├── cluster.yml                             # Kind cluster configuration
└── bootstrap.sh                            # Local deployment script
```

## Getting Started

### Prerequisites
- **Git** - Version control
- **Python 3.8+** - Local development
- **Docker** - Container runtime
- **Kind** - Local Kubernetes (optional)
- **Helm 3.x** - Chart management (optional)
- **kubectl** - Kubernetes CLI (optional)

### Local Development Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/zave52/cicd-python-app.git
   cd cicd-python-app
   ```

2. **Set up Python environment**
   ```bash
   # Create virtual environment
   python -m venv venv
   
   # Activate virtual environment
   # On Windows:
   venv\Scripts\activate
   # On Unix/macOS:
   source venv/bin/activate
   
   # Install dependencies
   cd src
   pip install -r requirements.txt
   ```

3. **Run the application locally**
   ```bash
   # Apply database migrations
   python manage.py migrate
   
   # Create superuser (optional)
   python manage.py createsuperuser
   
   # Run development server
   python manage.py runserver
   ```

4. **Access the application**
   - **Web Interface**: http://localhost:8000
   - **Admin Panel**: http://localhost:8000/admin
   - **API Endpoints**: http://localhost:8000/api/

### Testing and Quality Checks

```bash
# Run tests
python manage.py test

# Generate coverage report
coverage run --source='.' manage.py test
coverage report

# Run linting
flake8 . --show-source --statistics

# Check code complexity
flake8 . --max-complexity=6
```

## Docker Deployment

### Build and Run Locally

```bash
# Build Docker image
cd src
docker build -t todoapp:local .

# Run container
docker run -p 8000:8000 todoapp:local
```

### Using Docker Compose (if available)

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

## Kubernetes Deployment

### Quick Deployment with Kind

1. **Create local cluster**
   ```bash
   kind create cluster --config cluster.yml
   ```

2. **Deploy using bootstrap script**
   ```bash
   chmod +x bootstrap.sh
   ./bootstrap.sh
   ```

3. **Access the application**
   ```bash
   kubectl port-forward svc/todoapp-service 8080:80 -n todoapp
   # Open http://localhost:8080
   ```

### Manual Helm Deployment

```bash
# Install dependencies
helm dependency update helm-charts/todoapp

# Deploy to development
helm install todoapp-dev helm-charts/todoapp

# Deploy to staging
helm install todoapp-staging helm-charts/todoapp -f helm-charts/todoapp/values/stg.yaml

# Upgrade deployment
helm upgrade todoapp-dev helm-charts/todoapp
```

## CI/CD Pipeline Configuration

### Required GitHub Environments

This project uses GitHub Environments for deployment management. You need to create two environments in your GitHub repository:

1. **development** - For development deployments
2. **staging** - For staging deployments

To create environments:
1. Go to your GitHub repository
2. Navigate to **Settings** → **Environments**
3. Click **New environment**
4. Create both `development` and `staging` environments

### Required GitHub Secrets

#### Repository Secrets
Set up the following secrets at the repository level:

```bash
DOCKERHUB_USERNAME    # Docker Hub username
DOCKERHUB_TOKEN       # Docker Hub access token
```

#### Environment Secrets
Configure the following secrets for **BOTH** environments (`development` and `staging`):

```bash
# Database Configuration
MYSQL_ROOT_PASSWORD   # MySQL root password
MYSQL_USER           # MySQL database user
MYSQL_PASSWORD       # MySQL user password

# Application Configuration  
APP_SECRET_KEY       # Django secret key
APP_DB_NAME         # Database name for the application
APP_DB_HOST         # Database host (usually mysql-service for K8s)
```

**Example values for development environment:**
```bash
MYSQL_ROOT_PASSWORD=rootpassword123
MYSQL_USER=todoapp_user
MYSQL_PASSWORD=userpassword123
APP_SECRET_KEY=your-super-secret-django-key-here
APP_DB_NAME=todoapp_db
APP_DB_HOST=mysql-0.mysql.mysql.svc.cluster.local
```

**Note**: Use different, more secure values for the staging environment.

### Workflow Triggers

- **Push to main** - Full CI/CD pipeline execution
- **Pull requests** - CI testing only
- **Manual dispatch** - On-demand deployment with environment selection

### Environment Variables

```yaml
env:
  DockerImageName: todoapp    # Docker image name for builds
```

## Key CI/CD Features Demonstrated

### 1. **Matrix Strategy Testing**
```yaml
strategy:
  matrix:
    PythonVersion: [ 3.8, 3.9 ]
    OsTypes: [ ubuntu-latest, windows-latest ]
```

### 2. **Conditional Job Execution**
```yaml
if: ${{ github.ref_name == 'main' }}
```

### 3. **Artifact Management**
```yaml
- name: Upload python artifacts
  uses: actions/upload-artifact@v4
  with:
    name: python-artifacts-${{ matrix.OsTypes }}-${{ matrix.PythonVersion }}
    path: .
```

### 4. **Reusable Workflows**
```yaml
uses: ./.github/workflows/reusable-deployment.yml
secrets: inherit
with:
  environment: staging
  version: ${{ github.sha }}
```

### 5. **Environment-Specific Deployments**
```yaml
with:
  environment: staging
  helm-values-path: ./todoapp/values/stg.yaml
  helm-release-name: todoapp-staging
```

## Monitoring and Observability

The application includes health check endpoints for Kubernetes probes:
- **Health Check**: `/api/health/`
- **Readiness Check**: `/api/ready/`

## Contributing

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/amazing-feature`
3. **Make your changes** and ensure tests pass
4. **Commit your changes**: `git commit -m 'Add amazing feature'`
5. **Push to the branch**: `git push origin feature/amazing-feature`
6. **Open a Pull Request**

## Troubleshooting

### Common Issues

1. **Docker build failures**
   - Ensure all dependencies are in `requirements.txt`
   - Check Dockerfile syntax and paths

2. **Test failures in CI**
   - Run tests locally first: `python manage.py test`
   - Check for environment-specific issues

3. **Helm deployment issues**
   - Validate charts: `helm lint helm-charts/todoapp`
   - Check Kubernetes cluster status: `kubectl get nodes`

4. **GitHub Actions failures**
   - Check secret configuration
   - Verify artifact dependencies between jobs

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.