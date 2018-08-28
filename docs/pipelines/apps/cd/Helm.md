---
title: Create a Helm chart for a Docker enabled app 
description: Package and deploy a Docker-enabled app to an Azure Kubernetes Service (AKS) from Azure Pipelines
ms.assetid:
ms.prod: devops
ms.technology: devops-cicd
ms.topic: quickstart
ms.manager: douge
ms.author: ahomer
author: alexhomer1
ms.date: 08/28/2018
monikerRange: 'vsts'
---

# Helm

This guidance explains how to use **Helm**[https://www.helm.sh/] to package a Docker based application in to a Helm chart.

Helm is a tool that streamlines deploying and managing Kubernetes applications using a packaging format called **charts**[https://github.com/helm/helm/blob/master/docs/charts.md].
When the deployment target is Kubernetes, Helm Charts make it simple to package and deploy common applications on Kubernetes.  
You can define, version, share, install, and upgrade even the most complex Kubernetes application using Helm. 

A Helm chart consists of metadata, definitions, config and documentation. This can be either stored in the same code repository as your application code or in a separate repository. 
Helm can package these files into a chart archive (*.tgz file), which gets deployed to a Kubernetes cluster. 

A typical Helm chart will have the following structure: 
![Helm Chart Example](_img/Helmchart_example.png)


## Define your CI build process using Helm

We'll show you how to set up a continuous intergration workflow for your containerized application using Azure Pipelines and Helm.


## Prerequisites

You'll need an Azure subscription. You can get one free through [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/).

You'll need a Docker container image published to a Kubernetes Cluster (for example: Azure Kubernetes Service). 
To set up a continuous integration (CI) build process, see:

* [Build and publish a Docker image](../../languages/docker.md).

## Example

If you want some sample code that works with this guidance, import (into VSTS or TFS) or fork (into GitHub) this repo:

```
https://github.com/adventworks/dotnetcore-k8s-sample

```




## Package and publish a Helm chart

Extend the build pipeline created to build Docker image](../../languages/docker.md) to use Helm:

1. Add **Helm tool installer** task
2. Click on **+** icon again to add new **Package and deploy Helm charts** task
   Configure the properties as follows:
   
   - **Azure Subscription**: Select a connection from the list under **Available Azure Service Connections** or create a more restricted permissions connection to your Azure subscription.
     If you are using VSTS and if you see an **Authorize** button next to the input, click on it to authorize VSTS to connect to your Azure subscription. If you are using TFS or if you do not see
     the desired Azure subscription in the list of subscriptions, see [Azure Resource Manager service connection](../../library/connect-to-azure.md) to manually set up the connection.

   - **Resource Group**: Enter or select the resource group of your **AKS cluster**.  
   
   - **Kubernetes cluster**: Enter or select the **AKS cluster** you have created.  
   
   - **Command**: Select **init** as Helm command.
   
3. Again click on **+** icon to add another **Package and deploy Helm charts** task
   Configure the properties as follows:
   
   - **Azure Subscription**: Select a connection from the list under **Available Azure Service Connections** or create a more restricted permissions connection to your Azure subscription.
     If you are using VSTS and if you see an **Authorize** button next to the input, click on it to authorize VSTS to connect to your Azure subscription. If you are using TFS or if you do not see
     the desired Azure subscription in the list of subscriptions, see [Azure Resource Manager service connection](../../library/connect-to-azure.md) to manually set up the connection.

   - **Resource Group**: Enter or select the resource group of your **AKS cluster**.  
   
   - **Kubernetes cluster**: Enter or select the **AKS cluster** you have created.  
   
   - **Namespace**: Enter your Kubernetes cluster namespace where you want to deploy. If you don't have one, enter **dev**.

   - **Command**: Select **upgrade** as Helm command.

   When you select the **upgrade** as helm command, the task recognizes it and shows some additional fields.

   - **Chart Type**: Select **File Path** as Chart type.

   - **Chart Path**: Enter the path to your Helm chart. If you are publishing it using CI build, you can pick the file package by clicking on file picker.
   Or simply enter $(System.DefaultWorkingDirectory)/**/*.tgz

   - **Release Name**: Give any name to your release. For example **azuredevops**
   
   - **Arguments**: Enter the arguments and their value here. If you are using sample application for this document
   
    ```
    --set image.repository=$(imageRepoName) --set image.tag=$(Build.BuildId) 
    --set ingress.enabled=true --set ingress.hostname=$(hostName)

    ```
   > Either set the values of $(imageRepoName) in the variable section or replace it with your image repository name, which is typically of format `name.azurecr.io/coderepository`
   > You can find $(hotname) values in the Azure portal in the **Overview** and **Repositories** tabs for your AKS Cluster.

4. Save the build pipeline.

