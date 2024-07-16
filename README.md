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
