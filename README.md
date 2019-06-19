# WebGoat 8 on Azure Container Instances

## Introduction

WebGoat is a deliberately insecure web application maintained by OWASP designed to teach web application security lessons. The project is available at [Github](https://github.com/WebGoat/WebGoat) and an official [homepage](https://www.owasp.org/index.php/Category:OWASP_WebGoat_Project).

This guide shows how to run WebGoat 8 container version on Azure Container Instances.

## Prerequisites

* **An Azure subscription**. To create a free account go [here](https://azure.microsoft.com/en-gb/free/?utm_source=jeliknes&utm_medium=blog&utm_campaign=storage&WT.mc_id=storage-blog-jeliknes)
* **Azure Command Line (Azure CLI)** installed on your machine. To install go [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
* **Azure DevOps account**. To create account follow [here](https://azure.microsoft.com/en-in/services/devops/pipelines/)
* **Github account**. To join Github go [here](https://github.com/join) 

## Create Azure DevOps project and prepare WebGoat source code

In Azure DevOps create the new project *WebGoat*

![](https://githubpictures.blob.core.windows.net/webgoataci/CreateProject.png)

Clone WebGoat project to newly created project from the Github repository: https://github.com/WebGoat/WebGoat.git
Go to Repos from menu and choose `or import a repository`

![](https://githubpictures.blob.core.windows.net/webgoataci/ImportProject.png)

When source code is imported to Azure Repos fix the Dockerfile with the proper WebGoat version's snapshot. To find the current snapshot version open `pom.xml` file located in the root directory and find line `<version>v8.0.0.M25</version>` (currently line is number 9 and Snapshot version is M25). After checking the proper version go to the directory webgoat-server and edit Dockerfile by replacing in line `ARG webgoat_version=v8.0.0-SNAPSHOT` SNAPSHOT to M25 (or other version pointed in pom.xml) so it should be `ARG webgoat_version=v8.0.0.M25`

Enable `Multi-stage pipelines` in Preview feautures by clicking on Azure DevOps user icon and choosing `Preview feautures`

![](https://githubpictures.blob.core.windows.net/webgoataci/PreviewFeauture.png)

This is important as Pipeline will be based fully on yml syntax for both: Build and Release pipelines.

## Create Azure container registry and Azure Key Vault

Create an Azure container registry (ACR) to store WebGoat docker containers using Azure CLI. More information about ACR is [here](https://docs.microsoft.com/en-us/azure/container-registry/)

Create Resource Group where all resources, including ACR will be located:
`az group create --name webGoatRG --location westeurope`

Create a container registry with admin user enabled:
`az acr create --resource-group webGoatRG --name acrWebGoat --sku Basic --admin-enabled true --location westeurope`

Create a Key Vault for storing sensitive keys and passwords like ACR username and password.
`az keyvault create --name "webGoatKV" --resource-group "webGoatRG" --location westeurope`

## Configure CI/CD pipeline for Webgoat

Download `azure-pipelines.yml` from this repository: https://github.com/texnokot/WebGoatonAzureACI and upload it to the root directory of Azure Repos repository.
![](https://githubpictures.blob.core.windows.net/webgoataci/CommitPipeline.png)

The pipeline contains two stages:

* **Build** - builds and pushes WebGoat container to the ACR
* **DeployToDev** - deploys pushed container from ACR to ACI. The first time running also creates ACI



  