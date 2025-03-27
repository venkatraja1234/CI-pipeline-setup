# CI Pipeline Setup with Various Tools

This guide provides step-by-step instructions to set up a CI/CD pipeline using AWS EC2, Jenkins, SonarQube, Docker, Maven, and Trivy.

## Step 1: Create an EC2 Instance

1. Log in to AWS and navigate to the EC2 Dashboard.
2. Launch an instance with:
   - **OS:** Ubuntu t2.medium
   - **Storage:** 30GB EBS
   - **Region:** US-EAST-1
   - **Security Group:** Open necessary ports (8080 for Jenkins, 9000 for SonarQube, etc.).
3. Connect to the instance using SSH:
   ```sh
   ssh -i <your-key.pem> ubuntu@<EC2_PUBLIC_IP>
   ```
4. Switch to root user:
   ```sh
   sudo -i
   ```

## Step 2: Install Jenkins

```sh
sudo apt update -y
sudo apt upgrade -y
sudo apt install openjdk-17-jre -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
```
- Allow **port 8080** in security groups.
- Access Jenkins at `http://<EC2_PUBLIC_IP>:8080/`
- Retrieve admin password:
  ```sh
  cat /var/lib/jenkins/secrets/initialAdminPassword
  ```
- Install recommended plugins and create an admin user.

## Step 3: Install Docker

```sh
sudo apt update -y
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable" -y
sudo apt update -y
sudo apt install docker-ce -y
sudo chmod 777 /var/run/docker.sock
docker -v  # Verify installation
```

## Step 4: Install and Configure SonarQube

```sh
docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube
docker ps -a  # Get container ID
docker start <containerID>
```
- Access SonarQube: `http://<EC2_PUBLIC_IP>:9000/`
- Login with:
  - Username: **admin**
  - Password: **admin**
- Generate a token under **Administration > My Account > Security**.

## Step 5: Integrate SonarQube with Jenkins

1. Go to **Manage Jenkins > Configure System**.
2. Locate **SonarQube servers** and add:
   - **URL:** `http://<EC2_IP>:9000/`
   - **Token:** (Generated in SonarQube)
3. Configure webhook:
   ```sh
   http://<EC2_IP>:8080/sonarqube-webhook/
   ```

## Step 6: Install Maven

```sh
sudo apt update -y
sudo apt install maven -y
mvn -version
```

## Step 7: Install Trivy for Security Scanning

```sh
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

## Step 8: Configure Jenkins Credentials

### Add Docker Hub Credentials:
1. **Manage Jenkins > Credentials > System > Global Credentials.**
2. Add Docker Hub credentials with ID: `docker`.

### Add Jenkins Shared Library:
1. **Manage Jenkins > Configure System > Global Pipeline Library.**
2. Add:
   - **Name:** `my-shared-library`
   - **Default Version:** `main`
   - **Git URL:** `https://github.com/venkatraja1234/jenkins_shared_lib.git`

## Step 9: Verify CI/CD Pipeline

After executing the pipeline, verify:
- Jenkins logs for errors.
- Trivy scan results for vulnerabilities.
- SonarQube dashboard for code quality reports.

---

For more details, check the full documentation in **CI_PIPELINE_SETUP_WITH_VARIOUS_TOOLS.pdf**.
