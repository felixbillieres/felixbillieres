# 🗞️ Pipeline Automation

Before learning about automation security, we should start by defining the pipeline and showing where automation can take place. The diagram below shows what a typical pipeline can look like, as well as the software that could be used for this purpose:

<figure><img src="../../../../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>

### Source Code Storage

This is the start of our pipeline is `source code and version control`&#x20;

We need to consider several things when deciding where to store our code:

* How can we perform access control for our source code?
* How can we make sure that changes made are tracked?
* Can we integrate our source code storage system with our development tools?
* Can we store and actively use multiple different versions of our source code?
* Should we host our source code internally, or can we use an external third party to host our code?

### Version Control

Version control allows us to keep multiple versions of the code. This can be the specific version each developer is working on, but it can also be completely different versions of our application, including minor and major versions.

### Security Considerations

The two most common source code storage and version control systems are Git and SubVersion

Our source code is often our secret sauce. As such, we want to make sure it is not exposed. Source code cannot be fully secret since developers need access to it. As such, we should be careful not to confuse source code storage with secret management. We need to make sure not to store secrets, such as database connection strings and credentials, in our source code.

If an attacker got access to the repo, they could use a tool such as [GittyLeaks](https://github.com/kootenpv/gittyleaks), which would scan through the commits for sensitive information. Even if this information no longer exists in the current version, these tools can scan through all previous versions and uncover these secrets.

### External vs Internal Dependencies

Unless you are coding in binary, chances are you are actually only writing a fraction of the actual code. This is because a lot of the code has already been written for us in the form of libraries and software development kits (SDKs)

External dependencies are publicly available libraries and SDKs. These are hosted on external dependency managers such as PyPi for Python, NuGet for .NET, and Gems for Ruby libraries. Internal dependencies are libraries and SDKs that an organisation develops and maintains internally. For example, an organisation might develop an authentication library. This library could then be used for all applications developed by the organisation.

### Common Tools

A dependency manager, also called a package manager, is required to manage libraries and SDKs. As mentioned before, tools such as PyPi, NuGet, and Gems are used for external dependencies. The management of internal dependencies is a bit more tricky. For these, we can use tools such as JFrog Artifactory or Azure Artifacts to manage these dependencies.

### Unit and Integration Testing

When talking about automated testing in a pipeline, this will be the first type of testing that most developers and software engineers are familiar with. A unit test is a test case for a small part of the application or service. The idea is to test the application in smaller parts to ensure that all the functionality works as it should.

Another common testing method is integration testing. Where unit tests focus on small parts of the application, integration testing focuses on how these small parts work together.

### Security Testing SAST & DAST

Static Application Security Testing (SAST) works by reviewing the source code of the application or service to identify sources of vulnerabilities. SAST tools can be used to scan the source code for vulnerabilities.

Dynamic Application Security Testing (DAST) is similar to SAST but performs dynamic testing by executing the code. This allows DAST tools to detect additional vulnerabilities that would not be possible with just a source code review.

There are several common tools that can be used for automated testing. Both [GitHub](https://github.com/features/security/code) and [Gitlab](https://docs.gitlab.com/ee/user/application\_security/sast/) have built-in SAST tooling. Tools such as [Snyk](https://snyk.io/) and [Sonarqube](https://www.sonarqube.org/) are also popular for SAST and DAST.

### CI/CD

_Continuous Integration and Continuous Delivery_

In modern pipelines, software isn't manually moved between different environments. Instead, an automated process can be followed to compile, build, integrate, and deploy new software features. This process is called CI/CD.

We can create what is called a CI/CD pipeline. These pipelines usually have the following distinct elements:

* Starting Trigger - The action that kicks off the pipeline process. For example, a push request is made to a specific branch.
* Building Actions - Actions taken to build both the project and the new feature.
* Testing Actions - Actions that will test the project to ensure that the new feature does not interfere with any of the current features of the application.
* Deployment Actions - Should a pipeline succeed, the deployment actions detail what should happen with the build. For example, it should then be pushed to the Testing Environment.
* Delivery Actions - As CI/CD processes have evolved, the focus is now no longer just on the deployment itself, but all aspects of the delivery of the solution. This includes actions such as monitoring the deployed solution.

### Environments

Most pipelines have several environments. Each of these environments has a specific use case, and their security posture often differs.

<figure><img src="../../../../.gitbook/assets/image (140).png" alt=""><figcaption></figcaption></figure>
