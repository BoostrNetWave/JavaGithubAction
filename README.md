# JavaScriptGithubAction

Here we test Github Action Through Deploy.yaml file

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
        ssh-private-key: ${{ secrets.DEPLOY_KEY }}

    - name: Deploy script
      run: java -jar path/to/your/deploy.xml
