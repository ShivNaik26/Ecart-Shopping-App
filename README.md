# DevSecOps CI/CD Pipeline on AWS EKS – E-Cart Application

## Project Overview
This project implements an end-to-end automated **DevSecOps CI/CD Pipeline** to build, scan, package, and deploy a Java Spring Boot E-Commerce web application (**E-Cart**) onto an **AWS EKS (Elastic Kubernetes Service)** cluster. 

The pipeline enforces security and code quality checks at every stage before rolling out zero-downtime updates to production.

---

## Tech Stack & Tools Used
* **Source Code Management:** GitHub (Webhook-triggered automated builds)
* **CI/CD Orchestration:** Jenkins (Declarative Pipeline via `Jenkinsfile`)
* **Build & Unit Testing:** Apache Maven, JUnit, JaCoCo (Code Coverage)
* **Static Application Security Testing (SAST):** SonarQube (with Quality Gate enforcement)
* **Software Composition Analysis (SCA):** OWASP Dependency-Check (integrated with NVD API)
* **Artifact Repository:** Sonatype Nexus (`maven-releases` & `maven-snapshots`)
* **Containerization:** Docker & Docker Hub
* **Cloud & Container Orchestration:** AWS EKS, Kubernetes (`Deployment` & `LoadBalancer` Service), AWS CLI, `kubectl`
* **Application Stack:** Java, Spring Boot, Spring Security, Thymeleaf, Spring Data JPA, H2 Database

---

## CI/CD Pipeline Architecture (Stages)

The `Jenkinsfile` automates the following multi-stage workflow:

1. **Git Checkout:** Pulls the latest source code from the GitHub repository triggered automatically via GitHub Webhooks.
2. **Compile:** Compiles the Java source code using Maven (`mvn compile`).
3. **Unit Tests & Coverage:** Executes unit tests (`mvn test`) and generates JaCoCo coverage reports.
4. **SonarQube Analysis (SAST):** Scans the codebase for bugs, code smells, and security vulnerabilities.
5. **SonarQube Quality Gate:** Halts the pipeline automatically if the code fails to meet defined quality and security thresholds.
6. **OWASP Dependency-Check (SCA):** Scans project dependencies against the National Vulnerability Database (NVD) using an NVD API Key to detect known CVEs.
7. **Build Artifact:** Packages the application into an executable `.jar` file (`mvn package`).
8. **Deploy to Nexus:** Uploads versioned build artifacts (`shopping-cart-0.0.1-SNAPSHOT.jar`) to the Nexus Maven Repository.
9. **Build & Tag Docker Image:** Builds an optimized Docker image using `docker/Dockerfile` and tags it with the Jenkins build number and `latest`.
10. **Push Image to Docker Hub:** Authenticates securely using Jenkins Credentials Manager and pushes the image to Docker Hub.
11. **EKS & Kubectl Configuration:** Configures `kubeconfig` via AWS CLI to connect to the `aws-devops-eks` cluster in `ap-south-1`.
12. **Zero-Downtime K8s Deployment:** Applies `deploymentservice.yml` to deploy 2 replicas with a `RollingUpdate` strategy (`maxUnavailable: 0`, `maxSurge: 1`) exposed via an AWS LoadBalancer Service on port `8070`.

---

## Kubernetes Deployment Details (`deploymentservice.yml`)
* **Deployment Name:** `ekart-deployment`
* **Replicas:** `2`
* **Strategy:** `RollingUpdate` (Zero-downtime deployment)
* **Service Name:** `ekart-ssvc`
* **Service Type:** `LoadBalancer`
* **Application Port:** `8070`

---

## Running the Application Locally

### 1. Using Maven
bash
mvn clean package
java -jar target/shopping-cart-0.0.1-SNAPSHOT.jar

### 2. Using Docker
docker build -t ekart:latest -f docker/Dockerfile .
docker run --rm -i -p 8070:8070 --name ekart ekart:latest

### 3. Accessing the App
° Application URL: http://localhost:8070/home (or http://<AWS-LOADBALANCER-DNS>:8070/home)

° H2 Database Console: http://localhost:8070/h2-console (JDBC URL: jdbc:h2:mem:shopping_cart_db)

Default Login Credentials:

° Admin: Username: admin | Password: admin

° User: Username: user | Password: password
