# universal-containers

> Demo project showing an opinionated Development Lifecycle for Force.com

* [![Build Status](https://travis-ci.org/amtrack/universal-containers.svg?branch=master)](https://travis-ci.org/amtrack/universal-containers) master
* [![Build Status](https://travis-ci.org/amtrack/universal-containers.svg?branch=develop)](https://travis-ci.org/amtrack/universal-containers) develop
* [![Build Status](https://travis-ci.org/amtrack/universal-containers.svg?branch=feature/vat)](https://travis-ci.org/amtrack/universal-containers) feature/vat

## Prerequisites
### Salesforce
The following demo is based on a Salesforce org with at least three sandboxes:

 * `mydev` (a personal sandbox)
 * `ci` (a dedicated sandbox with all custom metadata removed)
 * `qa`

Note: For this demo we are using a *Consulting Partner Edition* org from the **Environment Hub**. If you use another type of Salesforce org, the metadata might be slightly different.

### Local development environment

 * MavensMate and an editor like Sublime Text
 * NodeJS
 * force-dev-tool `npm install force-dev-tool --global`
 * git

## Getting started

### Setting up the project
Step 1: Create a MavensMate project with the credentials for your personal sandbox

Step 2: Link the MavensMate project with a git repository

```console
$ cd path/to/your/mavensmate-project
$ git init
$ git remote add origin https://github.com/amtrack/universal-containers.git
$ git fetch origin
$ git checkout -b master origin/master --force
```

Step 3: Install dependencies

```console
$ npm install
```

Step 4: Configure the remotes to be used with `force-dev-tool`

```console
$ force-dev-tool remote add production user@example.com passToken https://login.salesforce.com
$ force-dev-tool remote add mydev user@example.com.mydev passToken --default
$ force-dev-tool remote add ci user@example.com.ci passToken
$ force-dev-tool remote add qa user@example.com.qa passToken
```

## Development
### Initial retrieving of metadata
Step 1: Create a `develop` branch as a base branch for all development

```console
$ git checkout -b develop
```

Step 2: Fetch all necessary information from the `production` org

```console
$ force-dev-tool fetch production --progress
API Versions
Available Metadata Types
Folders
Metadata Components
RecordTypes of PersonAccount
Active Flow versions
Created config/production-fetch-result.json
Fetching remotes finished.
```

Step 3: Build a `package.xml` file and retrieve the metadata

```console
$ force-dev-tool package -a production
Created src/package.xml
$ force-dev-tool retrieve production
Retrieving from remote production to directory src
{"fileName":"unpackaged/package.xml","problem":"Entity of type 'ListView' named 'WorkOrder.My_WorkOrders' cannot be found"}
{"fileName":"unpackaged/package.xml","problem":"Entity of type 'ListView' named 'SocialPost.AllSocialPosts' cannot be found"}
{"fileName":"unpackaged/package.xml","problem":"Entity of type 'ListView' named 'WorkFeedbackRequest.AboutTopics' cannot be found"}
{"fileName":"unpackaged/package.xml","problem":"Entity of type 'ListView' named 'WorkFeedbackRequest.All' cannot be found"}
{"fileName":"unpackaged/package.xml","problem":"Entity of type 'ListView' named 'Idea.Ideas_Last_7_Days' cannot be found"}
Succeeded
```

Note: Recognized the errors on `retrieve`?

To fix those errors, add the following lines to `.forceignore` and repeat step 3.

```text
# Entity of type 'ListView' named 'XXX' cannot be found
ListView/WorkOrder.My_WorkOrders
ListView/SocialPost.AllSocialPosts
ListView/WorkFeedbackRequest.AboutTopics
ListView/WorkFeedbackRequest.All
ListView/Idea.Ideas_Last_7_Days
```

Step 4: *Build* the metadata (test deploying the metadata to the empty `ci` sandbox)

```console
$ force-dev-tool validateTest ci
Running Validation with test execution of directory src to remote ci
Error: Validation with test execution failed.
 - Error in PersonalJourneySettings component 'PersonalJourney': Not available for deploy for this organization
 - Error in Profile component 'Admin': Unknown user permission: EditBillingInfo
Visit https://universal-containers--ci.cs83.my.salesforce.com/changemgmt/monitorDeploymentsDetails.apexp?asyncId=0Af4E000007OqbySAC for more information.
```

Note: Recognized the errors on `validateTest`?

To fix those errors, run the following command

add the following lines to `.forceignore` and repeat step 4:

```text
# Deployment fails to sandbox: Not available for deploy for this organization
Settings/PersonalJourney
```

Then run the following command to remove the billing from the `Admin.profile`:

```console
$ npm run fix:profile
```

Congrats! You successfully retrieved almost all available metadata.

Step 5: Version control the retrieved metadata

```console
$ git add -A
$ git commit -m "initially retrieve metadata from production"
```

### Developing a feature
Step 1: First, create a new feature branch

```console
$ git checkout -b feature/vat
```

Step 2: Do some declarative development in the Salesforce Setup Menu

Step 3: Retrieve all metadata

Step 4: Version control declarative metadata

Step 5: Do some programmatic development using MavensMate

Step 6: Retrieve all metadata

Step 7: Version control programmatic metadata

### Testing your feature
In order to make sure that all metadata which is version controlled, is still valid altogether, we can validate the complete set of metadata against an empty org.

```console
$ force-dev-tool validateTest ci
```

Note: This process is the same as for Continuous Integration.

### Testing the deployment of your feature
In order to create a deployment only containing your changes, we can prepare a changeset (deployment) based on a `git diff` output.

As a result, we get a directory only containing the changed metadata. This directory can then be validated or deployed.

```console
$ git diff develop feature/vat | force-dev-tool changeset create vat
$ force-dev-tool validateTest -d config/deployments/vat qa
```

Note: This process is the same as for Continuous Delivery.

### Deploying your feature
```console
$ force-dev-tool deployTest -d config/deployments/vat qa
```

### Undeploying a feature
If you finished your feature and want to clean up your personal sandbox for other development, you can undeploy your feature from your personal sandbox.

Hereby we leverage the information of an inverse diff.

```console
$ git diff feature/vat develop | force-dev-tool changeset create undo-vat
$ force-dev-tool deployTest -d config/deployments/undo-vat mydev
```

Note: Another use case would be to undeploy the feature from `qa`.
