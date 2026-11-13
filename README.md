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



## 5. Evidence of Successful Pipeline

### Job 1 - Successful Tests

After pushing changes to the 'dev' branch, the GitHub webhook triggered Job 1 automatically.

The tests passed, and the build completed successfully.

**Job 1 Console Output:**

```text
Started by user Trainee
Running as SYSTEM
Building remotely on EC2 (sparta-aws) - jenkins-node-2204-java17v5 (i-01869545601d0a1d0) in workspace /var/jenkins/workspace/piravien-spapp-job1-ci-test
The recommended git tool is: NONE
using credential piravien_github_key
Cloning the remote Git repository
Cloning repository git@github.com:piravienfive/pre-tech613-sparta-app-cicd-jenkins.git
 > git init /var/jenkins/workspace/piravien-spapp-job1-ci-test # timeout=10
Fetching upstream changes from git@github.com:piravienfive/pre-tech613-sparta-app-cicd-jenkins.git
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
using GIT_SSH to set credentials to read and write to github
 > git fetch --tags --force --progress -- git@github.com:piravienfive/pre-tech613-sparta-app-cicd-jenkins.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git config remote.origin.url git@github.com:piravienfive/pre-tech613-sparta-app-cicd-jenkins.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
Avoid second fetch
 > git rev-parse refs/remotes/origin/dev^{commit} # timeout=10
Checking out Revision e73c9c39ff93e74b787a236666a74b9e9f5e47fb (refs/remotes/origin/dev)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f e73c9c39ff93e74b787a236666a74b9e9f5e47fb # timeout=10
Commit message: "add message testing webhook on frontpage"
 > git rev-list --no-walk e73c9c39ff93e74b787a236666a74b9e9f5e47fb # timeout=10
[piravien-spapp-job1-ci-test] $ /bin/sh -xe /tmp/jenkins15672312421482799629.sh
+ cd app
+ npm install
npm warn deprecated inflight@1.0.6: This module is not supported, and leaks memory. Do not use it. Check out lru-cache if you want a good and tested way to coalesce async requests by a key value, which is much more comprehensive and powerful.
npm warn deprecated rimraf@3.0.2: Rimraf versions prior to v4 are no longer supported
npm warn deprecated glob@8.1.0: Old versions of glob are not supported, and contain widely publicized security vulnerabilities, which have been fixed in the current version. Please update. Support for old versions may be purchased (at exorbitant rates) by contacting i@izs.me
npm warn deprecated glob@7.2.3: Old versions of glob are not supported, and contain widely publicized security vulnerabilities, which have been fixed in the current version. Please update. Support for old versions may be purchased (at exorbitant rates) by contacting i@izs.me
npm warn deprecated glob@7.2.3: Old versions of glob are not supported, and contain widely publicized security vulnerabilities, which have been fixed in the current version. Please update. Support for old versions may be purchased (at exorbitant rates) by contacting i@izs.me
npm warn deprecated glob@7.2.3: Old versions of glob are not supported, and contain widely publicized security vulnerabilities, which have been fixed in the current version. Please update. Support for old versions may be purchased (at exorbitant rates) by contacting i@izs.me
npm warn deprecated glob@7.2.3: Old versions of glob are not supported, and contain widely publicized security vulnerabilities, which have been fixed in the current version. Please update. Support for old versions may be purchased (at exorbitant rates) by contacting i@izs.me
npm warn deprecated superagent@8.1.2: Please upgrade to superagent v10.2.2+, see release notes at https://github.com/forwardemail/superagent/releases/tag/v10.2.2 - maintenance is supported by Forward Email @ https://forwardemail.net
npm warn deprecated uuid@8.3.2: uuid@10 and below is no longer supported.  For ESM codebases, update to uuid@latest.  For CommonJS codebases, use uuid@11 (but be aware this version will likely be deprecated in 2028).

> sparta-test-app@1.0.1 postinstall
> node seeds/seed.js

Database connection closed

added 371 packages, and audited 372 packages in 19s

59 packages are looking for funding
  run `npm fund` for details

6 vulnerabilities (5 moderate, 1 high)

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
+ npm test

> sparta-test-app@1.0.1 test
> npx mocha --exit

Your app is ready and listening on port 3000


  Homepage
    ✔ should display the homepage at / GET (51ms)
    ✔ should contain the word Sparta at / GET

  Fibonacci
    ✔ should display the correct fibonacci value at /fibonacci/10 GET


  3 passing (76ms)

Triggering a new build of piravien-spapp-job2-ci-merge
Finished: SUCCESS
```

### Job 2 - Successful Merge

After Job 1 passed, Job 2 was automatically triggered.

Job 2 merged the changes from 'dev' into 'main' and pushed them to GitHub.

**Job 2 Console Output:**

```text
Started by upstream project "piravien-spapp-job1-ci-test" build number 9
originally caused by:
 Started by user Trainee
Running as SYSTEM
Building remotely on EC2 (sparta-aws) - jenkins-node-2204-java17v5 (i-01869545601d0a1d0) in workspace /var/jenkins/workspace/piravien-spapp-job2-ci-merge
[ssh-agent] Looking for ssh-agent implementation...
[ssh-agent]   Exec ssh-agent (binary ssh-agent on a remote machine)
$ ssh-agent
SSH_AUTH_SOCK=/tmp/ssh-XXXXXXmIy9rn/agent.4015
SSH_AGENT_PID=4017
[ssh-agent] Started.
Running ssh-add (command line suppressed)
Identity added: /var/jenkins/workspace/piravien-spapp-job2-ci-merge@tmp/private_key_3870323804628766716.key (jenkins@spapp-scm-ci)
[ssh-agent] Using credentials piravien_github_key (to read and write to github)
The recommended git tool is: NONE
using credential piravien_github_key
 > git rev-parse --resolve-git-dir /var/jenkins/workspace/piravien-spapp-job2-ci-merge/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url git@github.com:piravienfive/pre-tech613-sparta-app-cicd-jenkins.git # timeout=10
Fetching upstream changes from git@github.com:piravienfive/pre-tech613-sparta-app-cicd-jenkins.git
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
using GIT_SSH to set credentials to read and write to github
 > git fetch --tags --force --progress -- git@github.com:piravienfive/pre-tech613-sparta-app-cicd-jenkins.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
Checking out Revision e73c9c39ff93e74b787a236666a74b9e9f5e47fb (refs/remotes/origin/main)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f e73c9c39ff93e74b787a236666a74b9e9f5e47fb # timeout=10
Commit message: "add message testing webhook on frontpage"
 > git rev-list --no-walk e73c9c39ff93e74b787a236666a74b9e9f5e47fb # timeout=10
[piravien-spapp-job2-ci-merge] $ /bin/sh -xe /tmp/jenkins893908112141552180.sh
+ git checkout main
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
+ git merge origin/dev
Already up to date.
+ git push origin main
Everything up-to-date
$ ssh-agent -k
unset SSH_AUTH_SOCK;
unset SSH_AGENT_PID;
echo Agent pid 4017 killed;
[ssh-agent] Stopped.
Finished: SUCCESS
```


## 6. Set Up Job 3 - CD Deployment

Job 3 automatically deploys the application to AWS EC2 after Job 2 successfully merges the code into `main`.

I used an EC2 instance launched from an AMI, which already had the application and required software installed.

### 6.1 Prepare the EC2 Instance

I launched an EC2 instance from the AMI and configured its security group to allow ports 22 (SSH) and 3000 (application).

I connected to the instance and checked the existing application:

```bash
ls /home/ubuntu/repo/app/
sudo ss -ltnp | grep :3000
sudo pm2 list
```

The existing application was located at:

```text
/home/ubuntu/repo/app/
```

### 6.2 Add AWS Credentials to Jenkins

I went to **Manage Jenkins > Credentials > Global > Add Credentials**.

I selected **SSH Username with private key** and entered:

- **Username:**
- **ID:** piravien_aws_key
- **Private Key:** My EC2 SSH private key

### 6.3 Create Job 3

I created a new Freestyle project called:

```text
piravien-ttt-job3-cd-deploy
```

Under **Source Code Management**, I selected Git, added my repository SSH URL and selected my GitHub credentials.

I set the branch specifier to:

```text
*/main
```

Under **Build Triggers**, I selected **Build after other projects are built**.

I entered Job 2's name:

```text
piravien-ttt-job2-ci-merge
```

I selected **Trigger only if build is stable** and unchecked the GitHub webhook trigger.

Under **Build Environment**, I enabled **SSH Agent** and selected `piravien_aws_key`.

### 6.4 Add the Deployment Script

Under **Build Steps**, I selected **Execute shell** and added:

```bash
#!/bin/bash
set -e

mkdir -p ~/.ssh
chmod 700 ~/.ssh

ssh-keyscan -H 52.16.137.203 >> ~/.ssh/known_hosts

rsync -av app/ ubuntu@52.16.137.203:/home/ubuntu/repo/app/

ssh ubuntu@52.16.137.203 "cd /home/ubuntu/repo/app && npm install --ignore-scripts && sudo pm2 restart APP_NAME"
```

The script:

1. Configures SSH access to EC2.
2. Copies the latest files into the existing application directory, overwriting matching files.
3. Installs dependencies.
4. Restarts the existing application.

I replaced `APP_NAME` with the existing PM2 process name.

Before running the job, I needed to verify the SSH host key, ensure `ubuntu` could overwrite the application files, and allow the required restart command to run without a sudo password.

### 6.5 Test Job 3

I clicked **Build Now** and checked the console output.

A successful build should finish with:

```text
Finished: SUCCESS
```

I then checked the application on EC2:

```bash
curl http://localhost:3000
```

I also opened the website:

```text
http://52.16.137.203:3000
```

### 6.6 Test the Full Pipeline

I pushed a change to the `dev` branch.

The pipeline automatically:

1. **Job 1:** Tested the code.
2. **Job 2:** Merged `dev` into `main`.
3. **Job 3:** Overwrote the existing application files on EC2 and restarted the application.

This completed the automated CI/CD pipeline without creating a second application directory.

