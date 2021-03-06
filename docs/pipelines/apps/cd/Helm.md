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

This guidance explains how to use [Helm](https://www.helm.sh/) to package a Docker based application in to a Helm chart.

Helm is a tool that streamlines deploying and managing Kubernetes applications using a packaging format called [charts](https://github.com/helm/helm/blob/master/docs/charts.md).
You can define, version, share, install, and upgrade even the most complex Kubernetes application using Helm. 

A Helm chart consists of metadata, definitions, config and documentation. This can be either stored in the same code repository as your application code or in a separate repository. 
Helm can package these files into a chart archive (*.tgz file), which gets deployed to a Kubernetes cluster. 

A typical Continuous integration flow with Helm will have the following structure: 
![Helm Chart Example](_img/Helmchart_example.png)
The steps required to build a container image and pushing it to a container registry remains the same. Once that has been the done, we start creating a Helm Chart archive package. 

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

1. In the **Build &amp; Release** hub, open the the [build pipeline](../../languages/docker.md) created to build and publish a Docker image.

2. Select Tasks tab and click on **+** icon  to add **Helm tool installer** task  to ensure that the agent which runs the subsequent tasks has Helm and Kubernetes installed on it.
3. Click on **+** icon again to add new **Package and deploy Helm charts** task.
Configure the properties as follows:
   
   - **Azure Subscription**: Select a connection from the list under **Available Azure Service Connections** or create a more restricted permissions connection to your Azure subscription.
     If you are using VSTS and if you see an **Authorize** button next to the input, click on it to authorize VSTS to connect to your Azure subscription. If you are using TFS or if you do not see
     the desired Azure subscription in the list of subscriptions, see [Azure Resource Manager service connection](../../library/connect-to-azure.md) to manually set up the connection.

   - **Resource Group**: Enter or select the resource group of your **AKS cluster**.  
   
   - **Kubernetes cluster**: Enter or select the **AKS cluster** you have created.  
   
   - **Command**: Select **init** as Helm command.
     
   -**Arguments**: Provide additional arguments for the Helm command. In this case set this field as "--client-only" to ensure the installation of only the Helm client.
   
4. Again click on **+** icon to add another **Package and deploy Helm charts** task
   Configure the properties as follows:
   
   - **Command**: Select **package** as Helm command.    When you select **package** as the helm command, the task recognizes it and shows only the relevant fields.

   - **Chart Path**: Enter the path to your Helm chart. 
   
   -**Version**: Specify the exact chart version to install. If this is not specified, the latest version is installed. Set the version on the chart to this semver version.
   
   -**Destination**: Choose the destination to publish the Helm chart. If it is the working directory, just set "$(Build.ArtifactStagingDirectory)"
   
   -**Update dependency**: Tick this checkbox to run helm dependency update before installing the chart. It updates dependencies from 'requirements.yaml' to dir 'charts/' before packaging
   
5. Again click on **+** icon to add a **Publish Artifacts** task

6. Save the build pipeline.

