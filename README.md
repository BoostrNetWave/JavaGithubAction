# JavaScriptGithubAction

# Here we test Github Action Through Deploy.yaml file

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
        ssh-private-key: ${{ secrets.DEPLOY }}

    - name: Test SSH connection and Java installation
      run: ssh -o StrictHostKeyChecking=no root@170.64.166.103 'java -version'

    - name: Copy JAR file to server
      run: scp -o StrictHostKeyChecking=no helloworld.jar root@170.64.166.103:/root/k

    - name: Verify JAR file on server
      run: ssh -o StrictHostKeyChecking=no root@170.64.166.103 'ls -l /root/k/helloworld.jar'

    - name: Run Hello World Java program on server
      run: ssh -o StrictHostKeyChecking=no root@170.64.166.103 'java -jar /root/k/helloworld.jar'
