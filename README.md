# Deploying Static Resume Website on AWS EC2 Using GitHub Actions

This guide explains how to deploy your static resume website on an AWS EC2 instance using GitHub Actions.

---

## Prerequisites

1. **AWS Account**: Ensure you have an active AWS account.
2. **EC2 Instance**:
   - Launch an EC2 instance with the following specifications:
     - OS: Amazon Linux 2 or Ubuntu.
     - Open ports 22 (SSH) and 80 (HTTP) in the Security Group.
   - Note down the public IP address or domain name.
3. **GitHub Repository**:
   - Push your `index.html` and `style.css` files to a GitHub repository.
4. **IAM Role/User**:
   - Create an IAM user/role with permissions for EC2 and attach it to your instance.
5. **Install GitHub CLI**:
   - Use `gh` to configure GitHub Actions secrets for your repository.

---

## Setup GitHub Actions Workflow

### 1. Create a GitHub Actions Workflow File

In your repository, create a directory `.github/workflows` and add a file named `deploy.yml` with the following content:

```yaml
name: Deploy to AWS EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Set up SSH
      uses: webfactory/ssh-agent@v0.5.3
      with:
        ssh-private-key: ${{ secrets.EC2_SSH_KEY }}

    - name: Copy files to EC2
      run: |
        scp -o StrictHostKeyChecking=no -r index.html style.css ec2-user@${{ secrets.EC2_PUBLIC_IP }}:/var/www/html/

    - name: Restart Web Server
      run: |
        ssh -o StrictHostKeyChecking=no ec2-user@${{ secrets.EC2_PUBLIC_IP }} "sudo systemctl restart httpd || sudo systemctl restart apache2"
```

### 2. Configure GitHub Secrets

Add the following secrets to your GitHub repository:

1. **EC2_SSH_KEY**: Paste the private key you use to SSH into your EC2 instance.
2. **EC2_PUBLIC_IP**: Enter the public IP address or domain of your EC2 instance.

### 3. Configure Web Server on EC2

SSH into your EC2 instance and set up a web server:

1. Install Apache:
   ```bash
   sudo yum install httpd -y   # For Amazon Linux
   sudo apt install apache2 -y # For Ubuntu
   ```

2. Start the Apache service:
   ```bash
   sudo systemctl start httpd  # Amazon Linux
   sudo systemctl start apache2 # Ubuntu
   ```

3. Ensure `/var/www/html` is writable:
   ```bash
   sudo chmod -R 755 /var/www/html
   ```

4. Verify the server by accessing `http://<EC2_PUBLIC_IP>`.

---

## Deployment Process

1. Push changes to the `main` branch of your GitHub repository.
2. GitHub Actions will:
   - Connect to your EC2 instance using SSH.
   - Copy the `index.html` and `style.css` files to `/var/www/html/`.
   - Restart the Apache web server to apply changes.

---

## Verify Deployment

Visit `http://<EC2_PUBLIC_IP>` in your browser to see the static resume website.

---

## Troubleshooting

1. **SSH Key Issues**:
   - Ensure the private key matches the public key added to the EC2 instance.
   - Test SSH manually before running GitHub Actions.

2. **Web Server Errors**:
   - Check server logs:
     ```bash
     sudo tail -f /var/log/httpd/error_log   # Amazon Linux
     sudo tail -f /var/log/apache2/error.log # Ubuntu
     ```

3. **GitHub Actions Failures**:
   - Review the Actions logs for detailed errors.
   - Confirm secrets are correctly configured in the repository.

---

Congratulations! Your static resume website is now live on an AWS EC2 instance.
