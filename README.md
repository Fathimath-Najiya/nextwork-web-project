# CI/CD Pipeline Automation on AWS

A fully automated CI/CD pipeline built on AWS that takes code from GitHub 
to a live production web server without any manual intervention.

---

## 🏗️ Architecture

![Pipeline Flow](architecture-diagram.png)

**Development Environment → GitHub → CodeBuild → S3 → CodeDeploy → 
Web Server → Live Website**

---

## 🛠️ AWS Services Used

| Service | Purpose |
|---|---|
| AWS CodePipeline | Orchestrates the full CI/CD pipeline |
| AWS CodeBuild | Compiles and packages the Java web app |
| AWS CodeDeploy | Deploys the build artifact to EC2 |
| AWS CodeArtifact | Private Maven package repository |
| Amazon EC2 | Development & production web server |
| Amazon S3 | Stores build artifacts |
| AWS IAM | Roles and policies for secure access |
| AWS CloudFormation | Infrastructure as Code for production env |
| Amazon CloudWatch | Build monitoring and logging |
| GitHub | Source code repository |

---

## 📋 Prerequisites

- AWS Account
- GitHub Account
- Java (Corretto 8)
- Apache Maven
- VS Code with Remote - SSH extension

---

## 🚀 Pipeline Stages

### Stage 1 — Source
- GitHub hosts the web application source code
- AWS CodeConnections links GitHub to AWS via GitHub App
- Webhook events trigger the pipeline on every push to `master`

### Stage 2 — Build
- CodeBuild reads `buildspec.yml` to execute the build
- Installs Java Corretto 8
- Authenticates with CodeArtifact using a temporary IAM token
- Compiles and packages the app into a `.war` file using Maven
- Stores the build artifact in an S3 bucket

### Stage 3 — Deploy
- CodeDeploy reads `appspec.yml` for deployment instructions
- Runs deployment scripts on the production EC2 instance
- Starts Apache and Tomcat servers to serve the web app

---

## 📁 Project Structure
nextwork-web-project/
├── src/
│   └── main/
│       └── webapp/
│           ├── index.jsp
│           └── WEB-INF/
│               └── web.xml
├── scripts/
│   ├── install_dependencies.sh
│   ├── start_server.sh
│   └── stop_server.sh
├── buildspec.yml
├── appspec.yml
├── settings.xml
└── pom.xml

---

## 📄 Key Configuration Files

### buildspec.yml
Instructs CodeBuild on how to build the project:
- **Install phase** — sets Java runtime to Corretto 8
- **Pre-build phase** — retrieves CodeArtifact auth token
- **Build phase** — runs `mvn clean install`
- **Post-build phase** — packages app with `mvn package`

### appspec.yml
Instructs CodeDeploy on how to deploy the project:
- **BeforeInstall** — runs `install_dependencies.sh`
- **ApplicationStart** — runs `start_server.sh`
- **ApplicationStop** — runs `stop_server.sh`

---

## 🔐 IAM Roles Created

| Role | Purpose |
|---|---|
| EC2 CodeArtifact Role | Allows EC2 to fetch CodeArtifact auth token |
| CodeBuild Service Role | Allows CodeBuild to access CodeArtifact, S3, CloudWatch |
| CodeDeploy Service Role | Allows CodeDeploy to manage EC2 instances |

---

## ⚙️ Setup Instructions

### Step 1 — Set Up Development Environment
```bash
# Launch EC2 instance and connect via SSH
ssh -i your-keypair.pem ec2-user@your-ec2-dns

# Install Java and Maven
sudo dnf install java-1.8.0-amazon-corretto
sudo dnf install maven

# Generate Maven web app
mvn archetype:generate -DgroupId=com.nextwork \
  -DartifactId=nextwork-web-project \
  -DarchetypeArtifactId=maven-archetype-webapp
```

### Step 2 — Push Code to GitHub
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/your-username/nextwork-web-project.git
git push -u origin master
```

### Step 3 — Set Up CodeArtifact
- Create a domain and repository in AWS CodeArtifact
- Set upstream repository to `maven-central-store`
- Attach IAM policy to EC2 role for CodeArtifact access
- Configure `settings.xml` with CodeArtifact endpoint and auth token

### Step 4 — Set Up CodeBuild
- Create a CodeBuild project
- Connect to GitHub repository via CodeConnections
- Add `buildspec.yml` to project root
- Attach CodeArtifact policy to CodeBuild service role
- Set S3 bucket as artifact output location

### Step 5 — Set Up CodeDeploy
- Use CloudFormation to provision production EC2 + VPC
- Create CodeDeploy application and deployment group
- Tag production EC2 with `role: webserver`
- Add `appspec.yml` and deployment scripts to project
- Install CodeDeploy agent on production EC2

### Step 6 — Set Up CodePipeline
- Create pipeline with 3 stages: Source, Build, Deploy
- Enable webhook trigger for automatic pipeline runs
- Test by pushing a code change to GitHub

---

## ✅ Result

Every push to GitHub automatically triggers the full pipeline:
GitHub Push → CodePipeline → CodeBuild → S3 → CodeDeploy → Live Web App

**Code-to-production delivery time: ~3 minutes**

---

## 🐛 Common Issues & Fixes

| Issue | Fix |
|---|---|
| EC2 cannot access CodeArtifact | Attach IAM policy with `codeartifact:GetAuthorizationToken` |
| CodeBuild cannot find buildspec.yml | Add buildspec.yml to root of GitHub repository |
| CodeBuild cannot access CodeArtifact | Attach CodeArtifact policy to CodeBuild service role |
| CodeDeploy agent not responding | Verify agent is installed and running on EC2 instance |

---

## 📚 What I Learned

- How CI/CD pipelines automate the software delivery lifecycle
- How IAM roles and policies control AWS service permissions
- How to use Infrastructure as Code with CloudFormation
- How webhook events enable event-driven automation
- How CodeArtifact improves dependency security and reliability

---

## 👩‍💻 Author

**Fathimath Najiya CK**  
- AWS Certified Solutions Architect – Associate
- AWS Certified Cloud Practitioner
- [LinkedIn](#) | [GitHub](#)
