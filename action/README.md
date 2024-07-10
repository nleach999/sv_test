# Checkmarx CxFlow++ GitHub Action

This action uses the same underlying CxFlow scan orchestrator as the [checkmarx-cxflow-github-action](https://github.com/checkmarx-ts/checkmarx-cxflow-github-action).

This action has some significant differences:

* It executes SCAResolver in containerized environments with customized tooling built using the [cx-supply-chain-toolkit](https://github.com/checkmarx-ts/cx-supply-chain-toolkit/releases).
* It is a composite action that does not use the CxFlow container to execute.
* It can be configured to automatically select a feedback channel appropriate for a push or a pull-request event.
* The design focus is for it to be used in large scale deployments.


# Quick Start

Execute the CxFlow++ action by including it as a step in your workflow and supplying
the minimal required parameters:

```
- name: Scan with CxFlow++
  id: cxflow
  uses:  checkmarx-ts/checkmarx-cxflow-github-pp-action@v1
  with:
    sast-url: https://...
    sast-team: /CxServer/...
    project-name: myproject-{branch}
    app-name: MyProject
    sast-username: ${{ secrets.<YOUR VALUE HERE> }}
    sast-password: ${{ secrets.<YOUR VALUE HERE> }}
    sca-tenant: ${{ secrets.<YOUR VALUE HERE> }}
    sca-username: ${{ secrets.<YOUR VALUE HERE> }}
    sca-password: ${{ secrets.<YOUR VALUE HERE> }}

```

By default, the action execution will:

* Perform a SAST and SCA scan.
* Select Sarif as the feedback channel for push requests and update the GitHub
Security tab with results.
* Block merges for pull-requests targeting the defined default branch.
* Post a scan result summary to pull request comments.

All CxFlow configuration options can be overridden with additional configuration.

---


# Custom Dependency Resolution Build Environments

The Checkmarx SCA project is used for scanning project dependencies for known vulnerabilities.  To obtain an accurate dependency tree, the dependency
resolution part of your build must be executed.  Dependency resolution usually requires a properly configured build environment.  Deploying
a security GitHub action at enterprise scale can make it difficult to ensure that the scanning is performed in an appropriate build environment
across hundreds or thousands of repositories.  

This action, when used in conjunction with containerized build environments generated using the [cx-supply-chain-toolkit](https://github.com/checkmarx-ts/cx-supply-chain-toolkit/releases) can assist with enterprise-scale deployments of scanning actions.


## Supply Chain Toolkit Container Compatibility

The containerized dependency resolution environment must execute with
User Id 1001 and Group Id 127 for compatibility with Github action
execution environments.  Compatible containerized build environments
created with [cx-supply-chain-toolkit](https://github.com/checkmarx-ts/cx-supply-chain-toolkit/releases) can be created using a container build
tool (such as Docker buildx) like so:

```
docker build -t your_tag_here \
  --build-arg BASE=default-build:latest \
  --target=resolver-debian \
  --build-arg USER_ID=1001 \
  --build-arg GROUP_ID=127 \
  -f Dockerfile .
``````

# Configuration Parameters

## Required Parameters

* `app-name`: The name of the application used when making issue tracker tags.
* `project-name`: The name of the scan project.  This should include the branch being scanned using your organization's established system naming convention. The string `{branch}` will be replaced by the event's branch name if found in the project name.


**Required unless `disable-sast-scan` is set to `true`**
* `sast-url`: The base URL to the Checkmarx SAST server (do not include `CxWebClient`).
* `sast-team`: The full path of the SAST team that owns the project to scan.
* `sast-username`: The username of the SAST account to use for scanning.
* `sast-password`: The password for the selected SAST account.


**Required unless `disable-sca-scan` is set to `true`**

* `sca-tenant`: The name of the SCA tenant.
* `sca-username`: The username of the SCA account to use for scanning.
* `sca-password`: The password for the selected SCA account.


## Optional Parameters

### CxFlow Parameters
* `cxflow-jar-path`: (default: blank) The local path to the CxFlow jar.  If blank, CxFlow will be downloaded.
* `cxflow-version`: (default: latest) The CxFlow version to use for execution.  Ignored if `cxflow-jar-path` is set.
* `cxflow-params`: Command line parameters to pass to CxFlow at every execution.
* `pull-request-cxflow-params`: Command line parameters passed to CxFlow only when executing during a pull-request event.
* `push-cxflow-params`: Command line parameters passed to CxFlow only when executing during a push event.

### Docker Registry Parameters

* `docker-login-registry`: (default: docker.io) The name of the container registry to use for login.
* `docker-login-username`: (default: blank) The username for the container registry login.
* `docker-login-password`: (default: blank) The password for the container registry login.

### Execution Parameters

* `upload-sarif-file`: (default: true) Uploads the Sarif file to create entries on the GitHub security tab during push events. 
* `delete-sarif-file`: (default: true) Set to "false" when using the Sarif feedback channel to keep the Sarif file at the path returned in the `sarif-path` output parameter.
* `default-branch`: (default: `${{github.event.repository.default_branch}}`) The default branch of the repository used as the root parent project in SAST from which branch projects are derived.  This defaults to the repo's defined default branch.
* `disable-sast-scan`: (default: false) Set to true to disable SAST scanning.
* `disable-sca-scan`: (default: false) Set to true to disable SCA scanning.
* `enable-wire-trace`: (default: false) Set to true to emit web API I/O wire tracing to the action logs.
* `sast-log-level`: (default: INFO) The logging level for CxFlow output emitted as the action executes. (TRACE, DEBUG, ERROR, INFO)
* `push-feedback-channel`: (default: Sarif) The feedback channel to use when scanning for a push.
* `pull-request-feedback-channel`: (default: GITHUBPULL) The feedback channel to use when scanning for a pull request.
* `application-yaml-path`: (default: blank) A path to a CxFlow yaml configuration file.
* `scaresolver-tag`: (default: blank) The tag for the containerized SCAResolver build environment where the action will execute dependency scanning.  The image must not be built with a `*-bare` target. If this is not supplied, the SCA dependency scan will execute on the SCA server.

### Java Parameters
* `java-opts`: (default: -Xms512m -Xmx2048m) Options to pass to the JVM.
* `java-props`: (default: -Djava.security.egd=file:/dev/./urandom) Properties to pass to the JVM.
* `spring-props`: (default: blank) Spring boot properties to pass to the JVM.
* `custom-ca-path`: (default: blank) Path to a file or directory containing a custom CA cert chain that is imported into the Java runtime cacerts store. 
