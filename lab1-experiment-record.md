# Lab 1 CI/CD Pipeline Experiment Record

## 1. Basic Information

- Course: DevOps and Continuous Delivery
- Lab: Lab 1 - First CI/CD Pipeline
- Repository: git@github.com:yinghui-ren/lab1-cicd-pipeline.git
- Main branch: main

## 2. Team Work

- Team members:
- Responsibilities:
  - GitHub repository setup:
  - Web application selection:
  - Jenkins pipeline configuration:
  - Deployment and verification:
  - Documentation:

## 3. Technology Choices

| Component | Choice | Reason |
|---|---|---|
| Virtualisation | VirtualBox + Vagrant | Infrastructure-as-code, reproducible |
| CI/CD tool | Jenkins | Industry standard, flexible pipeline DSL |
| Web application | Spring PetClinic | Official Spring demo, Maven build, JUnit tests |
| OS (VM) | Ubuntu 22.04 (jammy64) | LTS, stable package ecosystem |

## 4. GitHub Repository Setup

### Commands

```bash
git init
git branch -M main
git remote add origin git@github.com:yinghui-ren/lab1-cicd-pipeline.git
git add .
git commit -m "Initial CI/CD lab setup"
git push -u origin main
```

### Repository Structure

```
lab1-cicd-pipeline/
├── Vagrantfile              # VM definition (Ubuntu + Jenkins)
├── Jenkinsfile              # Declarative pipeline (build/test/deploy/verify)
├── lab1-experiment-record.md
└── .gitignore
```

## 5. Environment Preparation

### Prerequisites (Windows host)

- VirtualBox: https://www.virtualbox.org/wiki/Downloads
- Vagrant: https://developer.hashicorp.com/vagrant/downloads

### Virtual Machine (auto-provisioned by Vagrant)

| Tool | Version |
|---|---|
| OS | Ubuntu 22.04 LTS |
| Java | OpenJDK 17 |
| Maven | 3.x (apt) |
| Jenkins | Latest stable (apt) |
| Git | Latest (apt) |

### VM Setup Commands

```bash
# On Windows host — inside lab1-cicd-pipeline/
vagrant up          # create VM, install all tools (~10 min first time)
vagrant ssh         # open shell inside the VM
vagrant halt        # stop VM
vagrant destroy -f  # delete VM completely
```

### Verify inside VM

```bash
java -version
mvn -version
git --version
systemctl status jenkins
```

## 6. Web Application — Spring PetClinic

- **Source**: https://github.com/spring-projects/spring-petclinic
- **Technology**: Java 17, Spring Boot, Maven, H2 in-memory DB
- **Build command**: `mvn clean package -DskipTests`
- **Test command**: `mvn test`
- **Run command**: `java -jar target/spring-petclinic-*.jar --server.port=8081`
- **Access URL (from Windows host)**: http://localhost:8081

### Manual Verification (inside VM)

```bash
# Clone the application
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic

# Build
mvn clean package -DskipTests

# Run tests
mvn test

# Start app (port 8081 to avoid conflict with Jenkins on 8080)
java -jar target/spring-petclinic-*.jar --server.port=8081

# In another terminal — verify it responds
curl http://localhost:8081
```

## 7. Jenkins Setup

### Access Jenkins

1. Open http://localhost:8080 in your Windows browser
2. Get the initial admin password:
   ```bash
   vagrant ssh -c "sudo cat /var/lib/jenkins/secrets/initialAdminPassword"
   ```
3. Install suggested plugins
4. Create admin user

### Configure Jenkins Pipeline

1. **New Item** → Pipeline → name it `petclinic-pipeline`
2. Under **Pipeline** section → Definition: **Pipeline script from SCM**
3. SCM: Git, Repository URL: `https://github.com/yinghui-ren/lab1-cicd-pipeline.git`
4. Branch: `*/main`
5. Script Path: `Jenkinsfile`
6. Save → **Build Now**

### Pipeline Stages

| Stage | What it does |
|---|---|
| Checkout | Clones Spring PetClinic from GitHub |
| Build | `mvn clean package -DskipTests` |
| Test | `mvn test`, publishes JUnit XML results |
| Deploy | Kills previous instance, starts JAR on port 8081 |
| Verify | `curl` health check confirms the app is up |

## 8. Port Forwarding Summary

| Service | VM port | Windows host port |
|---|---|---|
| Jenkins UI | 8080 | 8080 |
| PetClinic app | 8081 | 8081 |
| SSH | 22 | 2222 |

SSH from Windows: `ssh -p 2222 vagrant@127.0.0.1` (password: `vagrant`)
Or simply: `vagrant ssh`

## 9. Problems and Solutions

| Date | Problem | Cause | Solution | Result |
| --- | --- | --- | --- | --- |
| | | | | |

## 10. Progress Log

| Date | Work Done | Commit ID |
| --- | --- | --- |
| 2026-06-02 | Created Git repository and prepared lab record document. | |
| 2026-06-02 | Added Vagrantfile, Jenkinsfile, updated experiment record. | |
