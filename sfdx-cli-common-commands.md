# Frequently used Salesforce CLI commands
## Online documentation:
[https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_force.htm](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_force.htm)

### Note:
If you are reading this in text mode, ignore the ">" at the start of the line, it is for MD formatting purposes only.

It is recommended to copy the contents to a text file where you can play with creating the commands with your specific values and to save the text file to the project for easy reference.
## Working with orgs
### Retrieve current list of orgs
>sfdx force:org:list

### Authorize an org and set alias
>sfdx force:auth:web:login --setalias [ALIAS] --instanceurl [MY.DOMAIN.URL]
example:
>sfdx force:auth:web:login --setalias sandbox01 --instanceurl https://logic--integration.my.salesforce.com -d

*If there are issues with authentication...*

First check if you have the correct permissions. The simplest configuration consists of **System Permissions**:*API Enabled* and **Assigned Connected Apps**:*Force.com IDE*

If you do not have access to configure these permissions or are denied a request from the Administator, this will generally work:

>sfdx force:auth:device:login -r [MY DOMAIN URL]  

followed by setting the alias:  

>sfdx force:alias:set [ALIAS]=[USERNAME]

### Set Alias as default hub
>sfdx force:auth:web:login -d -a [ALIAS]  
>sfdx force:auth:web:login -d -a [ALIAS] -r [MYDOMAINURL]

### Set Alias as default user
>sfdx force:auth:web:login -s -a [ALIAS]  
>sfdx force:auth:web:login -s -a [ALIAS] -r [MYDOMAINURL]

### Log Out
>sfdx force:auth:logout -u [USER]

### Log Out of all (clean out after a project, for instance)
>sfdx force:auth:logout --all

### Create Scratch Org 
*(make sure current default is org you want scratch from and project-scratch-def.json is configured properly)*  
>sfdx force:org:create -f config/project-scratch-def.json --durationdays 30 --setalias [ALIAS]

### Generate password for scratch org
>sfdx force:user:password:generate -u [SCRATCH ORG USERNAME]

### Connect to or Open ORG
>sfdx force:org:open -u [ALIAS]

After a time you may end up with a bunch of old aliases, which can be annoying. To remove aliases, go to the config list path:  
    mac/linux: *cd ~/.sfdx*  
    Windows: *Open %USERPROFILE%\.sfdx*

Find the json file with the username you want to remove and delete it.

## Moving metadata in and out of the Org
First, some assumptions. The default project layout includes the path /manifest for package manifests and the location where the downloaded data will be is one folder level above the project and the commands are issued from the defalut path when opening a VSS terminal.  

If you need help creating your package, try the [Salesforce package.xml Builder](https://packagebuilder.herokuapp.com/) for a broadly-focused download, or try using the [Download Salesforce Change Set package.xml](https://chrome.google.com/webstore/detail/download-salesforce-chang/olkmefomaellbafiabkljcemiljkkbeh) with a targeted changeset to create your package xml file. Note that is a good practice to name your package xml file based on the features in the package rather than the default of "package.xml"

With the package xml file created, use the following:  
>sfdx force:mdapi:retrieve -r ../ -u [USERNAME] -k manifest/package.xml

When the retrieve is complete, unzip the resulting *unpackaged.zip* file and run the following:  
>sfdx force:mdapi:convert -r ../unpackaged -d force-app

To deploy your changes back to the org, first make any changes to the package.xml as necessary and then:  
>sfdx force:source:deploy -u <USERNAME> -x <PATH-TO-PACKAGE.XML>  

Example with our assumptions:  
>sfdx force:source:deploy -u [USERNAME] -x ../[PROJECT_FOLDER_NAME]/manifest/[PACKAGE_NAME].xml 

Once work has begun, there are push and pull commands. These will generally work with scratch orgs and rarely with standard sandboxes  
### Push from VS Code Project to Org
>sfdx force:source:push -u [ORG USERNAME FROM THE sfdx force:org:list]

### Pull from Org to VS Code Project
>sfdx force:source:pull -u [ORG USERNAME FROM THE sfdx force:org:list]

## Creating Unlocked Packages 
Unlocked packages enable the creation of package versions.  
When building a package, the *-u* switch to specify which connection to use does not work. You have to set the **default** connection to the org you are building the package from before running the build command.

### Prequisite:
>sfdx force:auth:web:login --setdefaultdevhubusername --setalias [ALIAS]  

### Create the pacakge
>sfdx force:package:create --name [PACKAGE_NAME] --description "[DESCRIPTION]" --packagetype Unlocked --path force-app --nonamespace 

Example:  
>sfdx force:package:create --name SAP_CUST_OBJS --description "SAP Custom Objects for API pushes from SAP" --packagetype Unlocked --path force-app --nonamespace

### Create Package version (used for distribution)
>sfdx force:package:version:create -p [PACKAGE_NAME] -d force-app -k [INSTALLATIONKEY] --wait 10

Note 1: The **INSTALLATIONKEY** is the password used to retrieve the package. Either don't use symbols or put the "INSTALLATIONKEY" in quotes.

Note 2: When the command completes, it will return the Package ID

Note 3: This section may require some revision

### Promote to Release verion of package (can install to any org)
>sfdx force:package:version:promote -p [PACKAGE ID or ALIAS]

### Create the install url for the package
> packaging/installPackage.apexp?p0=<04t id>

Example:
>packaging/installPackage.apexp?p0=04t4p0000006QooAAE

