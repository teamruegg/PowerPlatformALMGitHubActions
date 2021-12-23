# Power Platform ALM using GitHub Actions

For Power Platform ALM we used Power Platform Build Tools for Azure DevOps to establish continuous integration and continuous deployment pipeline. Azure DevOps has his strengths working with internal teams but gets challenging if you want to build app with different externals or a community. 

Recently we built some Power Apps within our Smart Factory community and want to share and contribute with a broader audience. GitHub was the preferred repository! And of course, we want to establish ALM CI/CD similar as we have it in Azure DevOps.

Microsoft launched GitHub Actions for Microsoft Power Platform and it’s relatively straightforward to establish it for you own GitHub repository. Enclose I would like to share on one example “Visitor Management” how we used GitHub Actions and established organization wide action templates for any other repository.

## Visitor Management solution
The Visitor Management is a collection of four Power Apps to manage smart factory visitors. We build this apps during a three-day Hackathon with the Swiss Smart Factory team.

![Image of Visitor Management](/images/SwissSmartFactoryVisitorManagementApps.png)

Let’s use this solution to implement a healthy application lifecycle management (ALM) using Microsoft Power Platform and GitHub.

## Steps
There are six steps to establish the GitHub Action for one repository and to publish those to the organization templates to ne used for all other repositories too.
1.	Create required environments for development, build and production
2.	Create the service principal account and give it rights to the environments created
3.	Application user creation
4.	Create a new GitHub repository
5.	Create GitHub workflows using GitHub Actions for Microsoft Power Platform
6.	Add the GitHub Actions to the organization templates

## Step 1 - Create required environments for development, build and production
You will need to create, or have access to, three Dataverse environments in your tenant. I recommend establishing some naming conventions to group the artefacts logically and standardize naming for environment, security group, etc.
All three environments should have equal configurations, except

Environment | Development | Test | Production 
------------ | ------------- | ------------- | ------------- 
Team huddle review work in progress | Update work in progress | Confirm current utilization and availability | Management reporting of work in progress and resourcing 
Environment type | Staging, as you might want to be able to copy, backup or reset the environment | Staging, as you might want to be able to copy, backup or reset the environment | Production
Environment URL | Use dev as a postfix of the production environment name. For example:
[production_name]dev.crm[region].dynamics.com | Use test as a postfix of the production environment name. For example:
[production_name]test.crm[region].dynamics.com | No postfix. Only production environment name. For example:
[production_name].crm[region].dynamics.com
Security group | Assign the Security Group where your developers are member of. For example, <App Name>_Developers | Assign the Security Group where your developers are member of. For example, <App Name>_Testers | Assign the Security Group where your production users are member of. For example, <App Name>_Users
Updates | Release wave Off, as you want to control what apps/resources are deployed. | Release wave Off, as you want to control what apps/resources are deployed. | Release wave Off, as you want to control what apps/resources are deployed.



This project contains a collection of GitHub Actions for Microsoft Power Platform development

* Export a solution from a development environment.
* Generate a build artifact (managed solution) and import into a production environment.

## Reference
* [GitHub Actions for Microsoft Power Platform (GitHub)](https://github.com/microsoft/powerplatform-actions).
* [GitHub Actions for Microsoft Power Platform (Docs)](https://docs.microsoft.com/en-us/power-platform/alm/devops-github-actions)
* [GitHub Sharing workflows, secrets, and runners with your organization](https://docs.github.com/en/actions/learn-github-actions/sharing-workflows-secrets-and-runners-with-your-organization)
