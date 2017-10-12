---
layout: post
title: Empower Salesforce Team to run Agile with Continuous Delivery
---

Continuous Delivery and CI with Salesforce is one problem I have thougt about for some time. Source driven development with source controll is central to CD/CI process. Salesforce for a long time had been doing ORG centric development, based on Production org. On Many Salesforce projects I was able to use source control but it was sort of a crutch used to back up source rather than doing CI. Recently new DX (Pilot) tool been getting more attention as a solution to change a flow to Source and repository based developmnet however, for now real production deployments mostly done with ANT or Change Sets.

In our team we have various skill sets but not everyone has command line skills. Typical developers are good with CLI tools for git, heroku or sfdx CLIs. But how about Admins or Declaratove Developer who making customizations to models or process?

To include everyone to run development through github I had to create a deployment tool simple enough for anyone to use that did not require lots of diverse skills on git CLI or ant scrcipts or shell scripts.

Started this project a while back on GitHub [sf-deploy-tool](https://github.com/iandrosov/sf-agile-deploy) project. Here is my solution

Using [Agile Accelerator](https://appexchange.salesforce.com/listingDetail?listingId=a0N30000000ps3jEAA) package make it simple to run Agile Sprints and allows for many UI customization and keeping it all in Salesforcce. Our team was used to work on AA Stories with Agile tool.

I created a new Story View - Visual Force page to list out all current Sprint stories as Work Items in Agile Accelrator. Ideally I wanted to customize existing Work Item detail page but in Classic it is a Managed package VFP no way to modify it. In Lightning we are able to deploy direct from custom button action on Work Item because it uses standard page layouts. Code found on GitHub [sf-agile-deploy-pkg](https://github.com/iandrosov/sf-agile-deploy-pkg)

Next I needed a service to pull metadata and commit source to git repository. Build a good old Java [Spring-Boot](https://projects.spring.io/spring-boot/) app with REST services to program all needed actions.
I selected Java mainly for easy integration to other tool libraries such as [Apache ANT](http://ant.apache.org/) and [Jgit](https://www.eclipse.org/jgit/) to manage deployments.

[ANT Migration Tool](https://developer.salesforce.com/docs/atlas.en-us.daas.meta/daas/forcemigrationtool_install.htm) - a Salesforce deployment tool using ant to pull metadata from source Development org and deploy to target org. I used ANT with Change set to allow SF Users easy selection of metadata items and automatically assemble package.xml deployment file.

[JGit](https://www.eclipse.org/jgit/) - is common Java Git library used to run API with GitHub to work with branches do commits to repository with programmable APIs.

Next Tool is Salesforce [REST API](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/intro_what_is_rest_api.htm) to handle Chatter Posts to notify users about deployment results direct on the Story records. Another benefit keeping Agile stories in Salesforce.

[Salesforce Webservice Connector WSC](https://developer.salesforce.com/page/Introduction_to_the_Force.com_Web_Services_Connector) - Client Java library used to add package.xml as attachment to Story to keep package per story. Normal git flow will be merging branch stories and package will also merge so individual story components will be lost in the merge. Keeping attachments helps to track that in simple way.

Now assemble several command services that enable simple URL paramters to initiate a deployment.
Deploying this tool to Heroku with temporary file storage helps with keeping temp files for deployment no permanent storage, only what is needed during deployment time.

Set up URL to call deployment service from custom button or link on VFP for each story we are able to deploy individual stories/Change sets with simple button push without git CLI and keep source tracking in GitHub. Here is example deployment log visible to user.

```
Start deployment log...
Deployment Tool - Salesforce Agile Accelerator

*** START INITIAL COMMIT DEPLOYMENT Process - W-001111 ***


0. Cloning from https://github.com/MyOrg/project-salesforce-dev.git to deploystore/W-001111/project-salesforce-dev

1. Clonned repository: deploystore/W-001111/project-salesforce-dev/.git $git clone https://github.com/MyOrg/project-salesforce-dev.git

2. Switch to empty branch to start: $git checkout empty

3. Create new story branch from empty and checkout new branch: $git checkout -b W-001111

4. Run ant to pull changeset from source TESTDev org: $ant pullChangeSet

pullChangeSet:
[sf:retrieve] Request for a retrieve submitted successfully.
[sf:retrieve] Request ID for the current retrieve task: 09Sg0000005GdLeEAK
[sf:retrieve] Waiting for server to finish processing the request...
[sf:retrieve] Request Status: InProgress
[sf:retrieve] Request Status: Succeeded
[sf:retrieve] Finished request 09Sg0000005GdLeEAK successfully.

BUILD SUCCESSFUL
Total time: 13 seconds

5. Add story files to local repo tracking: $git add .

6. Commit story files to local repo with comments: $git commit -m "W-001111 Make it better"

7. Push all comimted story files to remote: $git push origin W-001111

8. Run ant to deploy story W-001111 to target TESTQA org: $ant deployChangeSet

deployChangeSet:
[sf:deploy] Request for a deploy submitted successfully.
[sf:deploy] Request ID for the current deploy task: 0Afc000000fI5A6CAK
[sf:deploy] Waiting for server to finish processing the request...
[sf:deploy] Request Status: Pending
[sf:deploy] Request Status: InProgress (0/15)  -- Processing Type: CustomObject
[sf:deploy] Request Status: Succeeded

BUILD SUCCEEDED
/app/build.xml:43: 
*********** DEPLOYMENT SUCCESS ***********
Request ID: 0Afc000000fI5A6CAK


Total time: 21 seconds


9. Post status to chatter @mention assigned user


```

App repository can be found here [sf-deploy-tool](https://github.com/iandrosov/sf-agile-deploy)

Now everyone on our team is able to contribute to Continuous Delivery with GitHub and learn in the process.

 
