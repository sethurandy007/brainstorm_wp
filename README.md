**Automated Deployment of WordPress Website with Nginx and GitHub Actions**

**Overview**
This document provides a step-by-step guide to configuring an automated deployment pipeline for a WordPress website hosted on an AWS EC2 Instance, using GitHub Actions as the CI/CD tool and Nginx as the web server.

**Technologies Used:**
VPS server (AWS EC2) configured with Nginx, MySQL, and PHP 8.3.
GitHub repository for the WordPress project.
Domain name configured with SSL/TLS (Let's Encrypt).

**Steps**
**1. Server Provisioning and Initial Setup**
Provision an AWS EC2 instance.
Set up a secure Linux distribution (e.g., Ubuntu 24.04).
Configure firewall rules to allow SSH, HTTP, and HTTPS access.
Install the required software stack:
Nginx for the web server.
MySQL (or MariaDB) for the database.
PHP 8.3 for dynamic content processing.

**2. WordPress Setup and Configuration**
Deploy WordPress files to the EC2 instance.
Configure wp-config.php with the appropriate database credentials and other settings.
Install an SSL certificate using Let's Encrypt for secure communication (HTTPS).

**3. Nginx Configuration for WordPress**
Optimize the Nginx configuration for improved security and performance.
Attached is the nginx.conf file used for the setup, which is also committed to the GitHub repository.

**4. Configure DNS**
Point the domain (brainstorm.servehttp.com) to the AWS EC2 Elastic IP (13.200.83.50).

**5. GitHub Actions Workflow for Deployment**
Configure a GitHub Actions workflow for automated deployment. Below is the YML file used for deployment:

yaml
Copy code
name: Deploy WordPress

on:
  push:
    branches:
      - main  # Adjust based on branch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Create SSH directory and Add server to known_hosts
        run: |
          mkdir -p ~/.ssh
          ssh-keyscan -H 13.200.83.50 >> ~/.ssh/known_hosts

      - name: Set up SSH
        uses: webfactory/ssh-agent@v0.5.3
        with:
          ssh-private-key: ${{ secrets.SSH_KEY }}

      - name: Sync files with rsync
        run: |
          rsync -avz --exclude '.git' --exclude '.github' . ubuntu@13.200.83.50:/var/www/html/wordpress

**6. Explanation of Workflow Steps**
Checkout code: Checks out the code from the GitHub repository.
Create SSH directory and Add server to known_hosts: Creates the SSH directory and adds the server to the known_hosts to avoid SSH verification errors.
Set up SSH: Starts the SSH agent and adds the private key from GitHub Secrets.
Sync files with rsync: Deploys the files to the EC2 instance, excluding .git and .github.

**7. Testing and Verification**
Trigger the GitHub Actions workflow by pushing to the main branch.
Verify that the files are correctly deployed to the EC2 instance.
Test the website to confirm that it’s loading as expected with SSL and security headers.

**8. Security Measures Taken to Protect Website**
**Server Security:**
Allow only required access (HTTP and HTTPS) for the server in security groups.
Ensure other services (e.g., MySQL, SSH) are accessible only from trusted sources (IP whitelisting).
**WordPress Security:**
Configure WordPress database users with limited access to only the required database.
**Nginx Security:**
Implement a strict Content Security Policy (CSP) to block inline scripts and restrict sources.
Deny access to sensitive files such as wp-config.php and wp-admin.
Ensure the Nginx configuration file is not committed to the Git repository.
**SSL Configuration:**
Configure SSL using Let’s Encrypt.
Ensure all HTTP traffic is redirected to HTTPS.

**Conclusion**
This setup provides a secure and automated deployment pipeline for a WordPress site using GitHub Actions, Nginx, and rsync. It ensures that sensitive user-uploaded files are kept out of version control, and minimizes the risk of accidental overwrites.
