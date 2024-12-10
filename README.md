Prerequisites::

EC2 Instance Setup:
Ensure Docker is installed on your EC2 instance.
Verify you have SSH access to the EC2 instance.

GitHub Repository:
Your repository should include a Docker file to build your application.

Steps::

1. Create an SSH Key Pair
Generate an SSH key pair (if not already done):
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f github-actions

2. Add Public Key to EC2
Add the generated public key to the EC2 instance:
cat github-actions.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

3. Add GitHub Secrets
In your GitHub repository:
Go to Settings > Secrets and variables > Actions > New repository secret.
Add the following secrets:
EC2_HOST: Public IP or DNS of your EC2 instance.
EC2_SSH_KEY: The private key (github-actions).
EC2_USER: The username for your EC2 instance (e.g., ec2-user or ubuntu).

4. Update the Workflow File
Create or update the workflow file (.github/workflows/deploy.yml):

name: Deploy Docker to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Build Docker image
      run: docker build -t my-app .

    - name: Save Docker image to tarball
      run: docker save my-app -o my-app.tar

    - name: Copy Docker image to EC2
      uses: appleboy/scp-action@v0.1.8
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USER }}
        key: ${{ secrets.EC2_SSH_KEY }}
        source: my-app.tar
        target: ~/my-app.tar

    - name: Load and run Docker container on EC2
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USER }}
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          docker load < ~/my-app.tar
          docker run -d -p 80:80 my-app

Explanation::

1)Build Docker Image:
The docker build step creates the Docker image from the Dockerfile.

2)Save and Copy Image:
The image is saved as a tarball and copied to the EC2 instance using scp-action.

3)Load and Run on EC2:
The ssh-action connects to the EC2 instance, loads the image, and runs the container.

Test the Workflow::
Push changes to the main branch to trigger the workflow. It will build the Docker image, transfer it to the EC2 instance, and run the container.
