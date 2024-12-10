# web.demo.main
Prerequisites
EC2 Instance: Ensure your EC2 instance is running, and you have the .pem key file or access to it.
SSH Access: Verify you can SSH into the EC2 instance from your local machine.
GitHub Repository: Set up a GitHub repository.

-> Steps
1.Create SSH Key Pair
Generate an SSH key pair (if you don't already have one for GitHub Actions):

ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f github-actions

This generates:
github-actions (private key)
github-actions.pub (public key)

2.Add Public Key to EC2
Copy the content of github-actions.pub to the EC2 instance's ~/.ssh/authorized_keys file:

cat github-actions.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

3.Add Private Key to GitHub Secrets
Go to your GitHub repository.
Navigate to Settings > Secrets and variables > Actions > New repository secret.
Add the private key github-actions as a secret (e.g., EC2_SSH_KEY).

4.Update Workflow File
Create or update your GitHub Actions workflow file (.github/workflows/deploy.yml):
name: Deploy to EC2

on:
  push:
    branches:
      
main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    
name: Checkout code
    uses: actions/checkout@v3

    
name: Connect to EC2 via SSH
    uses: appleboy/ssh-action@v0.1.8
    with:
      host: ${{ secrets.EC2_HOST }}
      username: ec2-user  # Update based on your instance's username
      key: ${{ secrets.EC2_SSH_KEY }}
      script: |
        echo "Connected to EC2 instance successfully!"# Add your deployment commands here

5.Add EC2 Host to GitHub Secrets
Add the following secrets to your repository:

EC2_HOST: Public IP or DNS of your EC2 instance.
EC2_SSH_KEY: The private key content (github-actions).
Test the Workflow
Push your changes to the main branch to trigger the workflow. It should connect to the EC2 instance using SSH and execute the provided script.
