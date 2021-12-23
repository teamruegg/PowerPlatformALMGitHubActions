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

**Environment** | **Development** | **Test** | **Production** 
------------ | ------------- | ------------- | ------------- 
Environment type | **Staging**, as you might want to be able to copy, backup or reset the environment | **Staging**, as you might want to be able to copy, backup or reset the environment | **Production**
Environment URL | Use **dev** as a postfix of the production environment name. For example: [production_name]dev.crm[region].dynamics.com | Use **test** as a postfix of the production environment name. For example: [production_name]test.crm[region].dynamics.com | No postfix. Only production environment name. For example: [production_name].crm[region].dynamics.com
Security group | Assign the Security Group where your developers are member of. For example, [App Name]_Developers | Assign the Security Group where your developers are member of. For example, [App Name]_Testers | Assign the Security Group where your production users are member of. For example, [App Name]_Users
Updates | Release wave **Off**, as you want to control what apps/resources are deployed. | Release wave **Off**, as you want to control what apps/resources are deployed. | Release wave **Off**, as you want to control what apps/resources are deployed.


To create these three environments …
1.	Sign into the Power Platform admin center with credentials that provide access to a tenant with a minimum 3-GB available capacity (required to create the three environments).
2.	Select **Environments** in the navigation area.
3.	Select **+ New** to create your first new environment.
4.	Apply the required environment configuration for the application.
5.	After the environment is created, check under **Resources > Dynamics 365 apps** if all solutions are up to date. If not initiate update for each solution who is not up to date.
6.	Replay Step 3 – 5 for **test** and **production** environment.
7.	Final cross check. Go back to the **environment detail** screen of each created environment and check under **Version** the current **Database version**. Those should be equal for all new created environments.

![Image of Power Platform environments](/images/PowerPlatformALMUsingGitHubActions_Step1_Environments.png)


## Step 2 - Create the service principal account and give it rights to the environments created
Many of the actions require you to connect to a Microsoft Dataverse environment. You can add service principal or user credentials as secrets in your GitHub repository and then use them in the workflow. In our case we are using service principal to establish a connection to the related Microsoft Dataverse environment.

1.	You will need to create an application registration within Azure Active Directory.
Display name: **GitHubActions**
Supported account types: **My organization only**

![Image of Azure AD Application Registration](/images/PowerPlatformALMUsingGitHubActions_Step2_AADAppRegistration.png)

2.	Upon creation of the application registration, please note and save the **Directory (tenant) ID** and the of the application.
3.	On the navigation panel of the **Overview** page, select **API permissions**.
4.	Choose + Add a permission Application (client) ID, and in the Microsoft APIs tab, Choose **Dynamics CRM**.
5.	In the **Request API permissions form**, select **Delegated permissions**, check **user_impersonation**, and then choose **Add permissions**.
6.	From the **Request API permissions form**, choose **PowerApps Runtime Service**, select **Delegated permissions**, check **user_impersonation**, and then choose **Add permissions**.
7.	From the **Request API permissions form**, choose **APIs my organization uses**, search for **"PowerApps-Advisor"** using the search field, select **PowerApps-Advisor** in the results list, select **Delegated permissions**, check **Analysis**. All rights, and then choose **Add permissions**.

![Image of Azure AD Application Certificates & Secrets](/images/PowerPlatformALMUsingGitHubActions_Step2_CertificatesAndSecrets.png)

8.	Next, proceed to create a client secret, in the navigation panel, select **Certificates & secrets**.
9.	Below **Client secrets**, select **+ New client secret**.
10.	In the form, enter a description and select **Add**. Record the secret string, you will not be able view the secret again once you leave the form.

![Image of Azure AD Application Configured Permissions](/images/PowerPlatformALMUsingGitHubActions_Step2_ConfiguredPermissions.png)

## Step 3 - Application user creation
In order for the GitHub workflow to deploy solutions as part of a CI/CD pipeline an "Application user" needs to be given access to the environment. An "Application user" represents an unlicensed user that is authenticated using the application registration completed in the prior steps.

1.	Navigate to your Dataverse environment (https://[org].crm[region].dynamics.com).
2.	Navigate to **Settings > Security > Users**.
3.	Select the link **app users list**.
4.	Application user list
5.	Select **+ new app user**. A panel will open on the right hand side of the screen.
6.	Select **+ Add an app**. A list of all the application registrations in your Azure AD tenant is shown. Proceed to select the application name **GitHubActions** from the list of registered apps.
7.	Under **Business unit**, in the drop-down box, select your environment as the business unit.
8.	Under **Security roles**, select **System administrator**, and then select **create**. This will allow the service principal access to the environment.
9.	Repeat Step 1 – 8 for **test** and **production** environment.

![Image of Azure AD Application Configured Permissions](/images/PowerPlatformALMUsingGitHubActions_Step3_GitHubActions.png)

Now that you have created and configured the service principal for all environments and are ready to develop your solution within the development environment. I skip this step as we have our Visitor Management solution and proceed directly to the next step configuring your GitHub Workflows within your GitHub Repository.

## Step 4 - Create a new GitHub repository
Create your new repository and name it “Your application name”. Make sure you select Add a README file to initiate the repo and choose Create repository. In our case we created “VisitorManagement” repository.
Create a new secret to be used by GitHub Actions. We need to have the following secrets to be referenced within our GitHub Actions workflows:
* Password, for the username you are using to connect to Microsoft Power Platform
* PowerPlatformSPN, for the Service Principal Authentication
* ClientID, for the Application (client) ID reference
* TentantID, for the Directory (tenant) ID reference
* DevEnvironmentUrl, for the development environment link reference
* BuildEnvironmentUrl, for the build and test environment link reference
* ProdEnvironmentUrl, for the production environment link reference

1.	Navigate to the repo from the link in the import wizard and select Settings, navigate to Secrets, and then click New secret.
2.	On the secrets page, name the secret ‘password’. Type the password for the username you are using to connect to Microsoft Power Platform into the Value field and select Add secret. The password will be referenced in the YML files used to define the GitHub workflows later.
3.	Repeat Step 1 – 2 for all other secrets, PowerPlatformSPN, ClientID, TenantID, DevEnvironmentUrl, BuildEnvironmentUrl and ProdEnvironmentUrl. 

![Image of Azure AD Application Configured Permissions](/images/PowerPlatformALMUsingGitHubActions_Step4_GitHubCreateNewSecrets.png)


## Step 5 - Create GitHub workflows using GitHub Actions for Microsoft Power Platform


## Step 6 - Add the GitHub Actions to the organization templates




This project contains a collection of GitHub Actions for Microsoft Power Platform development

* Export a solution from a development environment.
* Generate a build artifact (managed solution) and import into a production environment.

## Reference
* [GitHub Actions for Microsoft Power Platform (GitHub)](https://github.com/microsoft/powerplatform-actions).
* [GitHub Actions for Microsoft Power Platform (Docs)](https://docs.microsoft.com/en-us/power-platform/alm/devops-github-actions)
* [GitHub Sharing workflows, secrets, and runners with your organization](https://docs.github.com/en/actions/learn-github-actions/sharing-workflows-secrets-and-runners-with-your-organization)
