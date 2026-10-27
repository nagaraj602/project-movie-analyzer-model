# Project Movie Analyzer - Sentiment Analysis Model Microservice

[![Python](https://img.shields.io/badge/Python-3.13%20%7C%203.15-blue.svg)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-3.1.2-lightgrey.svg)](https://palletsprojects.com/p/flask/)
[![Port](https://img.shields.io/badge/Service%20Port-7380-green.svg)](#port-configuration)
[![GitHub Repository](https://img.shields.io/badge/GitHub-nagaraj602%2Fproject--movie--analyzer--model-blue?logo=github)](https://github.com/nagaraj602/project-movie-analyzer-model.git)

This repository contains the AI/ML NLP microservice for the **Movie Analyzer Platform**, developed and maintained by **nagaraj602**. The service provides real-time natural language processing, sentiment polarity analysis, confidence level evaluation, and automated 1-5 star rating generation for user movie reviews.

---

## 🔗 Platform Ecosystem & Repositories

| Component | Repository Link | Tech Stack & Role |
| :--- | :--- | :--- |
| **Main Orchestration & IaC** | [movie-analyzer-project](https://github.com/nagaraj602/movie-analyzer-project.git) | Terraform (AWS EKS 1.37, RDS PostgreSQL 17, VPC, Jenkins), Docker Compose |
| **Backend API Service** | [project-movie-analyzer-backend](https://github.com/nagaraj602/project-movie-analyzer-backend.git) | Java 21/25 LTS, Spring Boot 3.5, Hibernate/JPA, Actuator (Port `7280`) |
| **Frontend Portal** | [project-movie-analyzer-frontend](https://github.com/nagaraj602/project-movie-analyzer-frontend.git) | React 19, Express 4, Node 24 LTS, White Aesthetic Design (Port `7180`) |
| **AI/ML Model Service** | [project-movie-analyzer-model](https://github.com/nagaraj602/project-movie-analyzer-model.git) | Python 3.13/3.15, Flask 3.1, TextBlob, NLTK (Port `7380`) |
| **Jenkins Shared Libraries** | [jenkins-sharedlibraries](https://github.com/nagaraj602/jenkins-sharedlibraries.git) | Reusable Jenkins Groovy pipelines (Trivy, SonarQube, Helm, ECR) |

---

## 🚀 Port & Networking Architecture

To avoid standard port conflicts (such as default 5000, 8080, 3000), this service is configured to run on non-standard port **`7380`**:

| Setting | Default Value | Description |
| :--- | :--- | :--- |
| **`MODEL_PORT`** | `7380` | Internal application listening port |
| **Docker Host Mapping** | `7380:7380` | Container external-to-internal port binding |
| **Kubernetes Service Port** | `7380` | ClusterIP service target port |
| **Health Check Path** | `http://localhost:7380/health` | Kubernetes Liveness & Readiness probe path |

---

## 🛠️ API Specifications & Endpoints

### 1. `GET /health`
Returns service health status for Kubernetes readiness/liveness probes and backend status polling.
```json
{
  "service": "model",
  "status": "healthy",
  "timestamp": 1793123248.74,
  "version": "1.0.0"
}
```

### 2. `POST /analyze`
Performs sentiment analysis on submitted movie review text.
* **Request Body:**
  ```json
  {
    "text": "This movie was an absolute masterpiece! Incredible direction and acting."
  }
  ```
* **Response Body:**
  ```json
  {
    "sentiment": "positive",
    "score": 0.85,
    "confidence": "high",
    "rating": 4.8,
    "text_length": 72,
    "processed_by": "textblob",
    "timestamp": 1793123248.80
  }
  ```

### 3. `GET /admin/status`
Returns real-time diagnostics, uptime, and NLP analysis capabilities for the live admin panel.

### 4. `POST /admin/toggle-health`
Simulates model failure / degradation for DevOps chaos testing and circuit breaker verification.

---

## 📦 Running Locally & Container Execution

### 1. Run with Python 3.13+
```bash
# Clone the repository
git clone https://github.com/nagaraj602/project-movie-analyzer-model.git
cd project-movie-analyzer-model

# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run server on port 7380
export MODEL_PORT=7380
python app.py
```

### 2. Run with Docker
```bash
# Build the container image
docker build -t nagaraj602/movie-analyzer-model:latest .

# Run on port 7380
docker run -d --name movie-model -p 7380:7380 -e MODEL_PORT=7380 nagaraj602/movie-analyzer-model:latest

# Verify
curl http://localhost:7380/health
```

---

## ☸️ Kubernetes & Helm Deployment

Helm configurations are maintained in the `helm/` directory:
```bash
# Deploy to Dev namespace
helm upgrade --install model-service helm/ \
  --namespace dev --create-namespace \
  -f helm/values/DEV.yaml

# Deploy to Production namespace
helm upgrade --install model-service helm/ \
  --namespace prod --create-namespace \
  -f helm/values/PROD.yaml
```

---

## 🔄 CI/CD Pipeline

The repository includes:
* `Jenkinsfile.pr`: Automated quality gate for pull requests (linting, vulnerability scan with Trivy, test execution).
* `Jenkinsfile`: Multibranch production delivery pipeline building and publishing ECR images and executing Helm chart releases to AWS EKS cluster.
