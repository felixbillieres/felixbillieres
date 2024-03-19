# CI/CD and Build Security

CI/CD stands for Continuous Integration and Continuous Delivery/Continuous Deployment. Build security, in the context of CI/CD, refers to the practices and measures taken to ensure the security of the software build process.

<figure><img src="../../../.gitbook/assets/image (119).png" alt=""><figcaption><p>Initial infrastructure</p></figcaption></figure>

### Fundamentals of CI/CD 

According to [Gitlab](https://about.gitlab.com/topics/ci-cd/), there are eight fundamentals for CI/CD:

* A single source repository - Source code management should be used to store all the necessary files and scripts required to build the application.\

* Frequent check-ins to the main branch - Code updates should be kept smaller and performed more frequently to ensure integrations occur as efficiently as possible.\

* Automated builds - Build should be automated and executed as updates are being pushed to the branches of the source code storage solution.\

* Self-testing builds - As builds are automated, there should be steps introduced where the outcome of the build is automatically tested for integrity, quality, and security compliance.\

* Frequent iterations - By making frequent commits, conflicts occur less frequently. Hence, commits should be kept smaller and made regularly.\

* Stable testing environments - Code should be tested in an environment that mimics production as closely as possible.\

* Maximum visibility - Each developer should have access to the latest builds and code to understand and see the changes that have been made.\

* Predictable deployments anytime - The pipeline should be streamlined to ensure that deployments can be made at any time with almost no risk to production stability.

### A Typical CI/CD Pipeline

* Developer workstations - Where the coding magic happens, developers craft and build code. In this network, this is simulated through your AttackBox.\

* Source code storage solution - This is a central placeholder to store and track different code versions. This is the Gitlab server found in our network.\

* Build orchestrator - Coordinates and manages the automation of the build and deployment environments. Both Gitlab and Jenkins are used as build servers in this network.\

* Build agents - These machines build, test and package the code. We are using GitLab runners and Jenkins agents for our build agents.\

* Environments - Briefly mentioned above, there are typically environments for development, testing (staging) and production (live code). The code is built and validated through the stages. In our network, we have both a DEV and PROD environment.

### Creating your own Pipeline

Create a GitHub account via the GitLab IP and go to the dashboard:

<figure><img src="../../../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

Then i went in the "explore" folder to go on this "Basic Build" project to play around with pipelines:

<figure><img src="../../../.gitbook/assets/image (121).png" alt=""><figcaption></figcaption></figure>

went and forked this project to add some specific username values:

<figure><img src="../../../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>

In GitLab, forking is a process where you create a personal copy of another user's project. This allows you to freely experiment with changes, additions, or improvements without affecting the original project.

### CI/CD Configuration

GitLab CI files allow you to define various jobs. Each job has different steps that must be executed, as found in the _script_ section of the job. Usually, there are three stages to any CI pipeline, namely the build, test, and deploy stage, but there can be more.

### **Build:**

```markdown
build-job:
  stage: build
  script:
    - echo "Hello, $GITLAB_USER_LOGIN!"
```

### Tests

Tests jobs are meant to perform tests on the build to ensure everything works as expected. Usually, you would execute more than one test job to ensure that you can individually test portions of the application. If one test job fails, the other test jobs will still continue, allowing you to determine all the issues with the current build, rather than having to do multiple builds.

```markdown
test-job1:
  stage: test
  script:
    - echo "This job tests something"

test-job2:
  stage: test
  script:
    - echo "This job tests something, but takes more time than test-job1."
    - echo "After the echo commands complete, it runs the sleep command for 20 seconds"
    - echo "which simulates a test that runs 20 seconds longer than test-job1"
    - sleep 20
```

### Deploy

In the deployment stage, we want to deploy our application to the relevant environment if both the build and test stages succeed. Usually, branches are used in source code repos, with the main branch being the only one that can deploy to production. Other branches will deploy to environments such as DEV or UAT

```markdown
deploy-prod:
  stage: deploy
  script:
- echo "This job deploys something from the $CI_COMMIT_BRANCH branch."
    - echo "Making the website directory"
    - mkdir -p /tmp/time/cicd
    - echo "Copying the website files"
    - cp website_src/* /tmp/time/cicd/
    - echo "Hosting website using a screen"
    - screen -d -m php -S 127.0.0.1:8081 -t /tmp/time/cicd/ &    
    - echo "Deployment complete! Navigate to http://localhost:8081/ to test!"
  environment: production
```
