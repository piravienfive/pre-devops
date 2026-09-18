# Pre-tech613 Learning

## CI/CD + Jenkins Intro

What is CI?
- Continuous Integration
- Autotmate the merging and testing of code

What is CD?
- Continuous Deployment/Delivery
- Delivery of code which is ready to be deployed
- Deployment does continuos delivery and deploys it

What is Jenkins?
- automation sever
- often used for CI/CD pipelines


# Jenkins CI/CD Pipeline

## Overview

In this project, I used Jenkins and GitHub to create a CI/CD pipeline for the Sparta test app.

The pipeline has two jobs:

- **Job 1:** Runs tests when code is pushed to the dev branch.
- **Job 2:** Merges the code into main if the tests pass.

The aim is to automate testing and merging instead of doing everything manually.

## 1. Generate an SSH Key for Jenkins

Jenkins needs an SSH key to securely connect to GitHub.

On the Jenkins server, I generated a new SSH key:

```bash
ssh-keygen -t ed25519 -a 100 -C "yourname-jenkins-2-github-key"
```

I saved the key in the user's `.ssh` folder.

I then:

1. Copied the public key.
2. Added it to my GitHub repository under **Settings > Deploy keys**.
3. Enabled **Allow write access** so Jenkins could push changes.
4. Added the private key to **Manage Jenkins > Credentials** as an SSH credential.

This allows Jenkins to access GitHub without storing a password in the build scripts.

## 2. Set Up Job 1

I created a new Freestyle project in Jenkins called:

```text
<yourname>-job1-ci-test
```
Under **GithHub Project** I added the repo URL.

Under **Source Code Management**, I selected Git and added my repository SSH URL.

I selected my SSH credentials and set the branch specifier to:

```text
*/dev
```

Under **Build Environment** I checked Provide Node & npm bin/ folder to PATH and chose Node JS version 20.


Under **Build Steps**, I selected **Execute shell** and added:

```bash
cd app
npm install
npm test
```

These commands install the dependencies and run the application tests.

I clicked **Build Now** to check that the job worked.

The build should finish with:

```text
Finished: SUCCESS
```

## 3. Set Up the GitHub Webhook

The webhook allows GitHub to automatically trigger Jenkins when code is pushed.

In Job 1, I went to **Configure > Build Triggers** and enabled:

```text
GitHub hook trigger for GITScm polling
```

I then opened my GitHub repository and went to:

**Settings > Webhooks > Add webhook**

I entered my Jenkins webhook URL:

```text
http://YOUR-JENKINS-IP:8080/github-webhook/
```

I saved the webhook.

## 4. Set Up Job 2

I created another Freestyle project called:

```text
<yourname>-spapp-job2-ci-merge
```
Then set **Copy from** to use job 1.

I connected it to the same GitHub repository using my SSH credentials.

I set the branch to:

```text
*/main
```

Under **Build Triggers**, I selected **Build after other projects are built**.

I entered the name of Job 1 and selected **Trigger only if build is stable**.

Uncheck:

```text
GitHub hook trigger for GITScm polling
```

Under **Build Environment**, check SSH Agent and select the specific credentials you created.

Under **Build Steps**, I selected **Execute shell** and replace the commands with:

```bash

git checkout main
git merge origin/dev
git push origin main
```

These commands merge the code from 'dev' into 'main' and push the changes to GitHub.
