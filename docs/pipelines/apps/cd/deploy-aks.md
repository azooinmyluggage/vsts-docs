---
title: Deploy a Docker container app to an Azure Kubernetes Service (AKS)
description: Set up continuous deployment (CD) of a Docker-enabled app to an Azure Kubernetes Service (AKS) from Azure Pipelines
ms.assetid:
ms.prod: devops
ms.technology: devops-cicd
ms.topic: quickstart
ms.manager: douge
ms.author: ahomer
author: alexhomer1
ms.date: 04/09/2018
monikerRange: 'vsts'
---

# Deploy to an Azure Kubernetes Service

We'll show you how to set up continuous deployment of your containerized application to an Azure Kubernetes Service (AKS) using
Azure Pipelines.

After you commit and push a code change, it is be automatically built and then deployed. The results will automatically show up on your site.

## Example

If you want some sample code that works with this guidance, import (into VSTS or TFS) or fork (into GitHub) this repo:

```
https://github.com/adventworks/dotnetcore-k8s-sample

```


## Define your CI build process

You'll need a continuous integration (CI) build process that publishes a container image to any container registry (for example: Azure Container Registry) and a Helm chart.
To set up a CI build process, see:

* [Build Docker image and publish Helm chart](../../languages/docker.md).

## Prerequisites

You'll need an Azure subscription. You can get one free through [Visual Studio Dev Essentials](https://visualstudio.microsoft.com/dev-essentials/).

## Create an Azure Kubernetes cluster to host a container

1. Sign into Azure at [https://portal.azure.com](https://portal.azure.com).

2. In the Azure Portal, choose **Create a resource**, **New**, **Containers**, then choose **Kubernetes Service**.    

3. Select or create a new Resource Group, enter name for your new Kubernetes cluster and DNS name prefix

   ![Creating the Web App for Containers](_img/create-aks-cluster.png)

4. Click on **Review + Create** and then once validation passes, click on **Create** button.

5. Wait until the new AKS cluster has been created. Then you can create a release pipeline as shown in the next section.

The **Docker** tasks you used in the build pipeline when you created the
build artifacts push the Docker image back into your Azure Container Registry.
The AKS cluster you created here will host an instance of that image and expose it as a website.

## Create a release pipeline

1. In the **Build &amp; Release** hub, open the build summary for your build.

2. In the build summary page, choose the **Release** icon to start a new release pipeline.

   If you have previously created a release pipeline that uses these build artifacts, you will
   be prompted to create a new release instead. In that case, go to the **Releases** tab page and
   start a new release pipeline from there by choosing the **+** icon.

3. Select the **Empty Process**.

   ![Adding the App Service Deployment task](_img/add-empty-process.png)

4. Go to **Environment 1** and click on **+** icon to add new task
5. Add **Helm tool installer** task to ensure that the agent which runs the subsequent tasks has Helm and Kubernetes installed on it.
6. Click on **+** icon again to add new **Package and deploy Helm charts** task
   Configure the properties as follows:
   
   - **Azure Subscription**: Select a connection from the list under **Available Azure Service Connections** or create a more restricted permissions connection to your Azure subscription.
     If you are using VSTS and if you see an **Authorize** button next to the input, click on it to authorize VSTS to connect to your Azure subscription. If you are using TFS or if you do not see
     the desired Azure subscription in the list of subscriptions, see [Azure Resource Manager service connection](../../library/connect-to-azure.md) to manually set up the connection.

   - **Resource Group**: Enter or select the resource group of your **AKS cluster**.  
   
   - **Kubernetes cluster**: Enter or select the **AKS cluster** you have created.  
   
   - **Command**: Select **init** as Helm command. This will install Tiller to your running Kubernetes cluster. It will also set up any necessary local configuration. Tick **Use canary image version** to install the latest pre-release version of Tiller. You could also choose to upgrade tiller if it is pre-installed by ticking on **Upgrade Tiller**
   
7. Again click on **+** icon to add another **Package Helm charts** task
   Configure the properties as follows:
   
   - **Azure Subscription**: Select a connection from the list under **Available Azure Service Connections** or create a more restricted permissions connection to your Azure subscription.
     If you are using VSTS and if you see an **Authorize** button next to the input, click on it to authorize VSTS to connect to your Azure subscription. If you are using TFS or if you do not see
     the desired Azure subscription in the list of subscriptions, see [Azure Resource Manager service connection](../../library/connect-to-azure.md) to manually set up the connection.

   - **Resource Group**: Enter or select the resource group of your **AKS cluster**.  
   
   - **Kubernetes cluster**: Enter or select the **AKS cluster** you have created.  
   
   - **Namespace**: Enter your Kubernetes cluster namespace where you want to deploy. If you don't have one, enter **dev**. This might help a CI/CD scenario where there is a segregation between environments and we can have a limited access tillers to each namespace allowing to segregate staging deploy from production deploy. Tiller namespace can be specified in the advanced section of the task or by passing the --tiller-namespace option as argument.

   - **Command**: Select **upgrade** as Helm command.

   When you select the **upgrade** as helm command, the task recognizes it and shows some additional fields.

   - **Chart Type**: Select **File Path** as Chart type.

   - **Chart Path**: Enter the path to your Helm chart. If you are publishing it using CI build, you can pick the file package by clicking on file picker.
   Or simply enter $(System.DefaultWorkingDirectory)/**/*.tgz

   - **Release Name**: Give any name to your release. For example **azuredevops**
   
   - **Arguments**: Enter the arguments and their values here. You could also specify the same in **Set Values** field as comma separated key-value pairs or provide a **Value File** which can be a YAML file or a URL . You could ensure traceabilty for the Helm chart by tagging it with source repository and Build ID as below:
   
    ```
    --set image.repository=$(imageRepoName) --set image.tag=$(Build.BuildId) 
   
    ```
   > Either set the values of $(imageRepoName) in the variable section or replace it with your image repository name, which is typically of format `name.azurecr.io/coderepository`
 
- **Reset Values**: Tick this checkbox if you need the values built into the chart to override all values provided from the task.
- **Recreate Pods**: Tick this checkbox if there is a configuration change during the release and you want to replace a running pod with new configuration
- **Force**: Tick this checkbox if you want to upgrade and rollback to delete and recreate the resurce and re-install the full release when there any any conflicts. This is useful in scenarios where applying patches can fail (e.g., for services, because clusterIp is immutable). 

8. Save the release pipeline.

## Create a release to deploy your app

You're now ready to create a release, which means to start the process of running the release pipeline with the artifacts produced by a specific build. This will result in deploying the build:

1. Choose **+ Release** and select **Create Release**.

2. In the **Create new release** panel, check that the artifact version you want to use is selected and choose **Create**.

3. Choose the release link in the information bar message. For example: "Release **Release-1** has been created".

4. Open the **Logs** tab to watch the release console output.

5. After the release is complete, navigate to your site running in Azure using the Web App URL `http://{web_app_name}.azurewebsites.net`, and verify its contents.


## Next steps

* [Set up multi-stage release](../../release/define-multistage-release-process.md)
