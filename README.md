# When droplet you're created on digital ocean 
# Public DEPLOY_PUBLIC key haing in Git Secretes Environment Use it On Droplet Authorized_keys

# During the creation of a DigitalOcean droplet and the setup of GitHub Actions, you'll be dealing with SSH keys to ensure secure access and automated deployment. Here's how you manage the SSH keys for both processes:

### SSH Key for Droplet Creation

When you create a droplet on DigitalOcean, you'll need to add an SSH key to the droplet. This allows you to securely connect to your droplet without using passwords.

1. **Generate an SSH Key** (if you haven't already):
   ```sh
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

   This will generate a public and private key pair, typically saved in `~/.ssh/id_rsa` (private key) and `~/.ssh/id_rsa.pub` (public key).

2. **Add the SSH Public Key to DigitalOcean**:
   - During droplet creation, there's an option to add an SSH key.
   - Copy the contents of your public key (`~/.ssh/id_rsa.pub`) and paste it into the SSH key field in the DigitalOcean dashboard.

### SSH Key for GitHub Actions

For GitHub Actions to deploy to your droplet, you need to use the same SSH key or generate a new one for automation purposes.

1. **Add the SSH Private Key to GitHub Secrets**:
   - Go to your repository on GitHub.
   - Navigate to `Settings` > `Secrets and variables` > `Actions`.
   - Add a new secret named `DO_SSH_PRIVATE_KEY` and paste the content of your private key (`~/.ssh/id_rsa`).

2. **Add Other Secrets**:
   - Add the droplet IP as `DO_HOST`.
   - Add the username you use to SSH into the droplet (typically `root` or another user you've created) as `DO_USER`.

### Step 3: GitHub Actions Workflow

Create a GitHub Actions workflow file to automate the deployment process. This file should be created in `.github/workflows/deploy.yml`.

Here is a complete example:

```yaml
name: Deploy to DigitalOcean

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up SSH
      uses: webfactory/ssh-agent@v0.5.3
      with:
        ssh-private-key: ${{ secrets.DO_SSH_PRIVATE_KEY }}

    - name: Deploy to server
      env:
        HOST: ${{ secrets.DO_HOST }}
        USER: ${{ secrets.DO_USER }}
      run: |
        echo "Deploying to $HOST"
        ssh $USER@$HOST << EOF
          # Install necessary packages
          sudo apt-get update
          sudo apt-get install -y nginx nodejs npm

          # Install PM2
          sudo npm install -g pm2

          # Set up firewall
          sudo ufw allow OpenSSH
          sudo ufw allow 'Nginx Full'
          sudo ufw enable

          # Set up project directories
          mkdir -p /var/www/mern-backend
          mkdir -p /var/www/mern-frontend

          # Clone repository
          cd /var/www/mern-backend
          git clone https://github.com/your-username/your-repo.git .

          # Install dependencies and build
          npm install
          npm run build

          # Start backend with PM2
          pm2 restart all || pm2 start index.mjs --name backend

          # Set up Nginx for frontend
          sudo cp -r build/* /var/www/html/
          sudo systemctl restart nginx
        EOF
```

### Step 4: Configure the Droplet

Here’s a more detailed breakdown of the droplet configuration:

1. **Update System Packages**:
   ```sh
   sudo apt-get update && sudo apt-get upgrade -y
   ```

2. **Install Node.js and npm**:
   ```sh
   curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
   sudo apt-get install -y nodejs
   ```

3. **Install PM2**:
   ```sh
   sudo npm install -g pm2
   ```

4. **Install Nginx**:
   ```sh
   sudo apt-get install -y nginx
   ```

5. **Set Up the Firewall**:
   ```sh
   sudo ufw allow OpenSSH
   sudo ufw allow 'Nginx Full'
   sudo ufw enable
   ```

### Step 5: Set Up Nginx

1. **Edit Nginx Configuration**:
   ```sh
   sudo nano /etc/nginx/sites-available/default
   ```

2. **Update the Configuration**:
   Replace the content with the following configuration:

   ```nginx
   server {
       listen 80;
       server_name your_domain_or_ip;

       # Serve React frontend
       location / {
           root /var/www/html;
           try_files $uri /index.html;
       }

       # Proxy API requests to the backend
       location /api/ {
           proxy_pass http://localhost:5000; # Adjust the port as needed
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

3. **Test and Reload Nginx**:
   ```sh
   sudo nginx -t
   sudo systemctl reload nginx
   ```

### Step 6: Directory Setup on Droplet

1. **Create Directories**:
   ```sh
   mkdir -p /var/www/mern-backend
   mkdir -p /var/www/mern-frontend
   ```

### Step 7: Security

1. **Add User and Disable Root Login**:
   - Create a new user:
     ```sh
     adduser your_new_user
     usermod -aG sudo your_new_user
     ```
   - Disable root login by editing the SSH config:
     ```sh
     sudo nano /etc/ssh/sshd_config
     ```
     - Set `PermitRootLogin` to `no`.
     - Restart SSH:
       ```sh
       sudo systemctl restart ssh
       ```

### Step 8: GitHub Actions Workflow Breakdown

1. **Checkout Code**:
   ```yaml
   - name: Checkout code
     uses: actions/checkout@v2
   ```

2. **Set Up SSH**:
   ```yaml
   - name: Set up SSH
     uses: webfactory/ssh-agent@v0.5.3
     with:
       ssh-private-key: ${{ secrets.DO_SSH_PRIVATE_KEY }}
   ```

3. **Deploy to Server**:
   ```yaml
   - name: Deploy to server
     env:
       HOST: ${{ secrets.DO_HOST }}
       USER: ${{ secrets.DO_USER }}
     run: |
       echo "Deploying to $HOST"
       ssh $USER@$HOST << EOF
         # Install necessary packages
         sudo apt-get update
         sudo apt-get install -y nginx nodejs npm

         # Install PM2
         sudo npm install -g pm2

         # Set up firewall
         sudo ufw allow OpenSSH
         sudo ufw allow 'Nginx Full'
         sudo ufw enable

         # Set up project directories
         mkdir -p /var/www/mern-backend
         mkdir -p /var/www/mern-frontend

         # Clone repository
         cd /var/www/mern-backend
         git clone https://github.com/your-username/your-repo.git .

         # Install dependencies and build
         npm install
         npm run build

         # Start backend with PM2
         pm2 restart all || pm2 start index.mjs --name backend

         # Set up Nginx for frontend
         sudo cp -r build/* /var/www/html/
         sudo systemctl restart nginx
       EOF
   ```

### Summary

1. **Create and configure a DigitalOcean droplet** with SSH access.
2. **Set up necessary software** (Node.js, npm, PM2, Nginx) and security configurations on the droplet.
3. **Configure GitHub Actions** to automate deployment with secure SSH keys and environment variables.
4. **Ensure proper security** by adding a non-root user and configuring the firewall.
5. **Set up Nginx** to serve your MERN stack application, with specific configurations for frontend and backend.

This approach provides a complete end-to-end solution for deploying a MERN stack application using GitHub Actions and a DigitalOcean droplet.

# Here are the detailed steps to add the secrets `DO_SSH_PRIVATE_KEY`, `DO_HOST`, and `DO_USER` to your GitHub repository:

### Adding Secrets to GitHub

1. **Navigate to Repository Settings**:
   - Go to your GitHub repository.
   - Click on `Settings`.

2. **Go to Secrets**:
   - On the left sidebar, click on `Secrets` and then `Actions`.

3. **Add a New Secret**:
   - Click the `New repository secret` button.

4. **Add `DO_SSH_PRIVATE_KEY`**:
   - In the `Name` field, enter `DO_SSH_PRIVATE_KEY`.
   - In the `Value` field, paste your SSH private key.
   - Click `Add secret`.

5. **Add `DO_HOST`**:
   - Click the `New repository secret` button again.
   - In the `Name` field, enter `DO_HOST`.
   - In the `Value` field, enter the IP address of your DigitalOcean droplet.
   - Click `Add secret`.

6. **Add `DO_USER`**:
   - Click the `New repository secret` button again.
   - In the `Name` field, enter `DO_USER`.
   - In the `Value` field, enter the username you use to SSH into your DigitalOcean droplet.
   - Click `Add secret`.

### Example Values for Secrets

- **DO_SSH_PRIVATE_KEY**: This is the private key you use for SSH. It should be in the format:

  ```
  -----BEGIN OPENSSH PRIVATE KEY-----
  ...
  -----END OPENSSH PRIVATE KEY-----
  ```

- **DO_HOST**: This is the IP address of your DigitalOcean droplet, for example:

  ```
  203.0.113.0
  ```

- **DO_USER**: This is the username you use to log into your DigitalOcean droplet, for example:

  ```
  root
  ```

### Final GitHub Action Workflow

Make sure your GitHub Actions workflow file (`.github/workflows/deploy.yml`) looks like this:

```yaml
name: Deploy to DigitalOcean

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up SSH
      uses: webfactory/ssh-agent@v0.5.3
      with:
        ssh-private-key: ${{ secrets.DO_SSH_PRIVATE_KEY }}

    - name: Build and deploy
      env:
        HOST: ${{ secrets.DO_HOST }}
        USER: ${{ secrets.DO_USER }}
      run: |
        echo "Deploying to $HOST"
        scp -r ./backend $USER@$HOST:/path/to/your/backend
        scp -r ./frontend $USER@$HOST:/path/to/your/frontend
        ssh $USER@$HOST << 'EOF'
          cd /path/to/your/backend
          npm install
          pm2 restart all || pm2 start server.js --name backend

          cd /path/to/your/frontend
          npm install
          npm run build
          sudo cp -r build/* /var/www/html/
          sudo systemctl restart nginx
        EOF
```

### Summary

By following these steps, you ensure that the necessary secrets (`DO_SSH_PRIVATE_KEY`, `DO_HOST`, and `DO_USER`) are securely stored in your GitHub repository. The workflow file will use these secrets to securely connect to your DigitalOcean droplet and deploy the application every time there is a push to the `main` branch.

# To run a GitHub Action from a specific file like `deploy.xml`, you'll need to ensure that it's configured correctly within your GitHub repository. Here’s a step-by-step guide:

1. **Create a GitHub Actions Workflow File:**
   - GitHub Actions workflows are typically written in YAML format and stored in the `.github/workflows/` directory of your repository. 
   - Since you mentioned `deploy.xml`, it suggests that the actual script or configuration might be XML-based, but the workflow to run it needs to be in YAML.

2. **Define the Workflow:**
   - Create a new file in `.github/workflows/`. For example, you might name it `deploy.yml`.

3. **Configure the Workflow:**
   - Inside your `.yml` file, you'll specify the necessary steps to run your `deploy.xml` script. Here’s a basic example of what that might look like:

```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up Java
      uses: actions/setup-java@v2
      with:
        java-version: '11'

    - name: Run deploy.xml
      run: java -jar path/to/your/deploy.xml
```

4. **Customize the Workflow:**
   - Depending on what your `deploy.xml` does, you might need additional setup steps, environment variables, or dependencies. Adjust the steps accordingly.

5. **Push the Workflow to GitHub:**
   - Commit and push your `.yml` file to the main branch (or the branch you want to use) of your repository. This will trigger the workflow based on the conditions defined in the `on` section.

Here’s a more detailed breakdown of the example workflow:

- **`name: Deploy`**: Names the workflow.
- **`on:`**: Specifies the trigger for the workflow. In this case, it triggers on a push to the `main` branch.
- **`jobs:`**: Defines the jobs to run as part of the workflow.
- **`runs-on:`**: Specifies the environment for the job (e.g., `ubuntu-latest`).
- **`steps:`**: Lists the individual steps within the job.
  - **`uses: actions/checkout@v2`**: Checks out the repository content.
  - **`uses: actions/setup-java@v2`**: Sets up the Java environment.
  - **`run: java -jar path/to/your/deploy.xml`**: Runs the `deploy.xml` script.

Make sure the path to `deploy.xml` is correct relative to the root of your repository. If your `deploy.xml` file requires specific dependencies or environment setup, you can add additional steps to handle those.

Feel free to share more details about your `deploy.xml` file and its requirements if you need further customization of the workflow.

**Before Apply Github Action In Platform Deploy or Running Server
**
Here’s how you can address this:

### 1. Ensure Java is Installed on the Server

1. **Check if Java is Installed:**
   SSH into your server and check if Java is installed:

   ```bash
   ssh your_username@your_server_ip
   java -version
   ```

   If Java is installed, it should display the version. If not, you need to install Java.

2. **Install Java (if not installed):**
   Install Java using your package manager. For example, on a Debian-based system:

   ```bash
   sudo apt update
   sudo apt install default-jdk
   ```

   For CentOS/RHEL-based systems:

   ```bash
   sudo yum install java-11-openjdk-devel
   ```

3. **Verify Java Installation:**
   After installing Java, verify the installation:

   ```bash
   java -version
   ```

### 2. Update GitHub Actions Workflow

Once Java is installed on your server, ensure your GitHub Actions workflow is correctly configured to run the deployment script.

Here is an updated example of your workflow:

```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up SSH agent
      uses: webfactory/ssh-agent@v0.5.3
      with:
        ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

    - name: Test SSH connection and Java installation
      run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'java -version'

    - name: Deploy script
      run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'java -jar path/to/your/deploy.xml'
```

### Debugging Steps

1. **Test SSH Connection and Java:**

   The `Test SSH connection and Java installation` step in the workflow checks if Java is correctly installed and accessible on the server.

2. **Ensure PATH is Set Correctly:**
   If Java is installed but still not found, ensure that the Java binary is in the system's PATH. You can do this by updating the `.bashrc` or `.bash_profile` file on your server:

   ```bash
   echo 'export PATH=$PATH:/path/to/java/bin' >> ~/.bashrc
   source ~/.bashrc
   ```

3. **Install Java in Workflow (Alternative Approach):**
   If you have control over the server setup and want to ensure Java is always installed, you can include a step to install Java directly in your GitHub Actions workflow (if running commands on a remote server, ensure permissions and sudo access are correctly handled):

   ```yaml
   - name: Install Java
     run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'sudo apt update && sudo apt install -y default-jdk'
   ```

### Summary Checklist

1. **Ensure Java is installed on the server:**
   - SSH into the server and install Java if not already installed.
   - Verify the Java installation with `java -version`.

2. **Update GitHub Actions Workflow:**
   - Add steps to test the SSH connection and Java installation.
   - Run the deployment script with Java.
  

# ===============================================================================

# To demonstrate HelloWorld Program Execution On Server

To demonstrate a simple "Hello World" example in your GitHub Actions workflow, we can modify the workflow to execute a basic Java program that prints "Hello World". This example assumes you have a simple Java program packaged in a JAR file named `helloworld.jar`.

### Steps to Create and Deploy a Simple Java "Hello World" Program

1. **Create a Simple Java Program:**

   Create a Java file named `HelloWorld.java`:

   ```java
   public class HelloWorld {
       public static void main(String[] args) {
           System.out.println("Hello, World!");
       }
   }
   ```

2. **Compile and Package the Java Program:**

   Compile the Java program and package it into a JAR file:

   ```sh
   javac HelloWorld.java
   jar cfe helloworld.jar HelloWorld HelloWorld.class
   ```

   Now you have a JAR file named `helloworld.jar` that contains your "Hello World" program.

3. **Upload the JAR File to Your Repository:**

   Add the `helloworld.jar` file to your GitHub repository.

4. **Create a GitHub Actions Workflow:**

   Create a workflow file (e.g., `.github/workflows/deploy.yml`) to run the Java program on your server:

   ```yaml
   name: Deploy

   on:
     push:
       branches:
         - main

   jobs:
     deploy:
       runs-on: ubuntu-latest

       steps:
       - name: Checkout repository
         uses: actions/checkout@v2

       - name: Set up SSH agent
         uses: webfactory/ssh-agent@v0.5.3
         with:
           ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

       - name: Test SSH connection and Java installation
         run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'java -version'

       - name: Copy JAR file to server
         run: scp -o StrictHostKeyChecking=no helloworld.jar your_username@your_server_ip:/path/to/your/directory

       - name: Run Hello World Java program on server
         run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'java -jar /path/to/your/directory/helloworld.jar'
   ```

### Explanation

1. **Checkout repository:** This step checks out the repository containing your JAR file.

2. **Set up SSH agent:** This step sets up the SSH agent and adds your SSH private key to authenticate with the server.

3. **Test SSH connection and Java installation:** This step verifies that the SSH connection and Java installation on the server are working.

4. **Copy JAR file to server:** This step copies the `helloworld.jar` file to your server.

5. **Run Hello World Java program on server:** This step runs the Java "Hello World" program on your server.

### Add SSH Key to GitHub Secrets

Make sure to add your SSH private key to GitHub secrets with the name `SSH_PRIVATE_KEY`:

1. Go to your GitHub repository.
2. Navigate to **Settings** > **Secrets and variables** > **Actions**.
3. Click **New repository secret**.
4. Name the secret `SSH_PRIVATE_KEY`.
5. Paste the contents of your SSH private key and click **Add secret**.

By following these steps, you can create a simple "Hello World" Java program, package it into a JAR file, and deploy it using GitHub Actions to run on your server.


# Complete step by step Process how to deploy application on Droplet or Instance By following steps

The error `Unable to access jarfile helloworld.jar` typically occurs when the JAR file path is incorrect or the file does not exist at the specified location. Below, I'll go through the possible causes and solutions for this issue and provide a refined GitHub Actions workflow to ensure that the JAR file is correctly created and accessed.

### Troubleshooting and Solutions for `Unable to access jarfile helloworld.jar`

#### 1. **Verify the JAR File Creation**

Make sure the `helloworld.jar` file is correctly created and contains the `HelloWorld` class. Check if you have followed these steps correctly:

- **Create the Java Source File and Compile**

  ```sh
  echo 'public class HelloWorld {
      public static void main(String[] args) {
          System.out.println("Hello, World!");
      }
  }' > HelloWorld.java

  javac HelloWorld.java
  ```

- **Create the Manifest File**

  ```sh
  echo 'Main-Class: HelloWorld' > manifest.txt
  ```

- **Build the JAR File**

  ```sh
  jar cfm helloworld.jar manifest.txt HelloWorld.class
  ```

- **Verify the JAR File**

  ```sh
  jar tf helloworld.jar
  ```

  The output should list `HelloWorld.class` and the `META-INF/MANIFEST.MF` file.

#### 2. **Update GitHub Actions Workflow**

Ensure that your GitHub Actions workflow file correctly creates and deploys the JAR file. Here's a refined version of the workflow to cover all necessary steps and check for common pitfalls:

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up Java
      uses: actions/setup-java@v3
      with:
        java-version: '11'  # Specify the Java version you need

    - name: Create the Java source file
      run: |
        echo 'public class HelloWorld {
            public static void main(String[] args) {
                System.out.println("Hello, World!");
            }
        }' > HelloWorld.java

    - name: Compile the Java source file
      run: |
        javac HelloWorld.java

    - name: Create the manifest file
      run: |
        echo 'Main-Class: HelloWorld' > manifest.txt

    - name: Rebuild the JAR file with the correct manifest
      run: |
        jar cfm helloworld.jar manifest.txt HelloWorld.class

    - name: Verify JAR file contents
      run: |
        jar tf helloworld.jar
        ls -l

    - name: Set up SSH agent
      uses: webfactory/ssh-agent@v0.5.3
      with:
        ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

    - name: Test SSH connection and Java installation
      run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'java -version'

    - name: Copy JAR file to server
      run: scp -o StrictHostKeyChecking=no helloworld.jar your_username@your_server_ip:/path/to/your/directory

    - name: Verify JAR file on server
      run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'ls -l /path/to/your/directory/helloworld.jar'

    - name: Run Hello World Java program on server
      run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'java -jar /path/to/your/directory/helloworld.jar'
```

#### 3. **Detailed Steps in Workflow**

Here's a breakdown of the steps with explanations and additional checks:

- **Checkout Repository:**

  ```yaml
  - name: Checkout repository
    uses: actions/checkout@v2
  ```

  This step ensures that your code, including `HelloWorld.java`, is available in the runner environment.

- **Set up Java:**

  ```yaml
  - name: Set up Java
    uses: actions/setup-java@v3
    with:
      java-version: '11'  # Adjust this to the required Java version
  ```

  Ensures that Java is installed and available for subsequent commands.

- **Create the Java Source File:**

  ```yaml
  - name: Create the Java source file
    run: |
      echo 'public class HelloWorld {
          public static void main(String[] args) {
              System.out.println("Hello, World!");
          }
      }' > HelloWorld.java
  ```

  Creates the `HelloWorld.java` file if not present in the repository.

- **Compile the Java Source File:**

  ```yaml
  - name: Compile the Java source file
    run: |
      javac HelloWorld.java
  ```

  Compiles the Java source file to produce `HelloWorld.class`.

- **Create the Manifest File:**

  ```yaml
  - name: Create the manifest file
    run: |
      echo 'Main-Class: HelloWorld' > manifest.txt
  ```

  Creates the `manifest.txt` with the `Main-Class` entry.

- **Rebuild the JAR File:**

  ```yaml
  - name: Rebuild the JAR file with the correct manifest
    run: |
      jar cfm helloworld.jar manifest.txt HelloWorld.class
  ```

  Packages the `HelloWorld.class` file into `helloworld.jar` with the `manifest.txt`.

- **Verify JAR File Contents:**

  ```yaml
  - name: Verify JAR file contents
    run: |
      jar tf helloworld.jar
      ls -l
  ```

  Checks the contents of the JAR file and lists files to ensure `helloworld.jar` exists and is properly built.

- **Set up SSH Agent:**

  ```yaml
  - name: Set up SSH agent
    uses: webfactory/ssh-agent@v0.5.3
    with:
      ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
  ```

  Adds the SSH private key for server authentication.

- **Test SSH Connection and Java Installation:**

  ```yaml
  - name: Test SSH connection and Java installation
    run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'java -version'
  ```

  Verifies that you can connect to the server and that Java is available.

- **Copy JAR File to Server:**

  ```yaml
  - name: Copy JAR file to server
    run: scp -o StrictHostKeyChecking=no helloworld.jar your_username@your_server_ip:/path/to/your/directory
  ```

  Transfers the `helloworld.jar` file to the specified directory on the server.

- **Verify JAR File on Server:**

  ```yaml
  - name: Verify JAR file on server
    run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'ls -l /path/to/your/directory/helloworld.jar'
  ```

  Ensures that `helloworld.jar` is in the correct location on the server.

- **Run Hello World Java Program on Server:**

  ```yaml
  - name: Run Hello World Java program on server
    run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'java -jar /path/to/your/directory/helloworld.jar'
  ```

  Runs the `helloworld.jar` file on the server.

### 4. **Common Fixes for JAR File Access Issues**

If you are still experiencing the `Unable to access jarfile helloworld.jar` error, consider the following:

1. **Check JAR File Path:**

   Ensure the path to the `helloworld.jar` file on the server is correct. Verify that it matches the path used in the `scp` and `ssh` commands.

2. **Permissions:**

   Ensure that the `helloworld.jar` file has the correct permissions for execution. You can set permissions using:

   ```sh
   chmod +x /path/to/your/directory/helloworld.jar
   ```

3. **Path Issues:**

   Make sure there are no typos or incorrect paths in your `scp` and `ssh` commands. The path to the JAR file on the server should be exact.

4. **Verify File Transfer:**

   Double-check that the `scp` command successfully copied the JAR file. You can use the `ls -l` command on the server to verify that `helloworld.jar` is present:

   ```sh
   ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'ls -l /path/to/your/directory/helloworld.jar'
   ```

5. **Rebuild and Redeploy:**

   Sometimes, starting fresh helps. Rebuild the JAR file and redeploy it:

   ```sh
   rm helloworld.jar
   jar cfm helloworld.jar manifest.txt HelloWorld.class
   ```

6. **Ensure `ssh` Command is Correct:**

   Make sure you are using the correct `ssh` command to run the JAR file:

   ```sh
   ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'java -jar /path/to/your/directory/helloworld.jar'
   ```

### Example Workflow for Existing JAR File (Revised)

If you only need to deploy an existing JAR file, you can use the following simplified workflow:

```yaml
name: Deploy

on:
  push:
    branches

:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up SSH agent
      uses: webfactory/ssh-agent@v0.5.3
      with:
        ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

    - name: Verify JAR file
      run: ls -l path/to/your/helloworld.jar

    - name: Copy JAR file to server
      run: scp -o StrictHostKeyChecking=no path/to/your/helloworld.jar your_username@your_server_ip:/path/to/your/directory

    - name: Verify JAR file on server
      run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'ls -l /path/to/your/directory/helloworld.jar'

    - name: Run Hello World Java program on server
      run: ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'java -jar /path/to/your/directory/helloworld.jar'
```

### Summary

By ensuring correct file creation, proper paths, and permissions, and using the revised workflow, you can address the `Unable to access jarfile helloworld.jar` issue effectively.

Feel free to adjust the workflow to fit your specific project setup and requirements.

### Common Commands and Explanations

Here are some commands for reference:

- **List files in the current directory:**

  ```sh
  ls -l
  ```

- **Check the contents of a JAR file:**

  ```sh
  jar tf helloworld.jar
  ```

- **Verify the Java version on the server:**

  ```sh
  ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'java -version'
  ```

- **Copy a file to the server:**

  ```sh
  scp -o StrictHostKeyChecking=no helloworld.jar your_username@your_server_ip:/path/to/your/directory
  ```

- **Run a JAR file on the server:**

  ```sh
  ssh -o StrictHostKeyChecking=no your_username@your_server_ip 'java -jar /path/to/your/directory/helloworld.jar'
  ```

Adjust these commands and workflow steps to suit your specific needs.
