---
layout: post
title:  "Jekyll website on an Azure Blob Storage using Azure DevOps"
excerpt: "Discover how run a jekyll website CI/CD and host it without a web server"
---

This repo is an example on how to build a jekyll website on an azure blob storage using Azure DevOps

## Introduction

Few months ago, Microsoft released the possibility to host a website on an [Azure Blob Storage](https://azure.microsoft.com/en-us/services/storage/blobs/) and I was interested in a modern way to build a blog site. Jekyll is a static website generator, written in Ruby which allows to simply write your content as markdown and click on the "publish" button. On the other side, a blob storage on Azure is an efficient way of storing and stream files, mainly to make them available for your applications. If the blob storage has a lot of intersting features, that the [static website](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website) which interests us right now. The static website allow to serve a website...without hosting it on a web server. The main advantages here are:

- the price: you only pay the storage, around 0,02cts/mo/Gb
- the performance: a blob storage can serve up to 50 Gbps (even more using the premium tier)
- replication: all data can be automatically replicated in different datacenters arround the world
- CDN: you can just enable a CDN above it so serve it with the nearest data

Of course, the can be overpowered for a personal website/blog but even just for the price, that the best way to host a static website in 2019.

> Static website does not mean pure HTML. It could contain Javascript and fully dynamic UI (Angular, ReactJS, Vue.js, etc.). Most of the new websites are built using fully Javascript and not being server-side anymore.

## Prepare a jekyll website

Whatever will be your operating system (it could be Windows 10 with the Ubuntu subsystem), you'll need to install ruby, jekyll and bundler (a ruby gem package manager). Then you just need to type the following command in an empty folder, in order to generate the scaffold of the website

``` bash
jekyll new .
```

You may have to generate a .gitignore file for ignoring the "jekyll-compiled" files.

``` gitattributes
_site/
.sass-cache/
.jekyll-cache/
.jekyll-metadata
```

If the Gemfile is missing, you can try to run a bundler init command

``` bash
bundle init
```

If the test is OK, then it's time to commit and push your folder and content on your Git repository.

## Set up a build pipeline

The idea here is to use jekyll to "build" our website and then, we take the output and package it for deployment. In Azure DevOps, create a build pipeline (you can build a Yaml pipeline but I will show it using the designer to be more didactic)

First, select your Git repository. It will also later to trigger continuous building.
![Fetch source code](/assets/images/p/jekyll-static-website/jek-source.png)

Then let's add some tasks:

- one to install Ruby on the agent
- one to install jekyll
- one to install bundler
- one to run jekyll
- one to adjust some files to package
- one to package it

![Build tasks](./images/jek-tasks.png)

To look into details, the three bash tasks could be merged into one but I keep it like it for more clarity. But at the end, it will contain the following commands:

The upgrading of ruby to be sure to have the last version of rubygems as there were some breaking changes at some point. Plus the install of jekyll "runtime"

![Install jekyll](./images/jek-bash1.png)

The installation of Jekyll

![Install gems](./images/jek-bash2.png)

Then we build the website. Jekyll will takes the different assets and outputs the result in the _site folder
![Building the jekyll website](./images/jek-bash3.png)

Sadly, that's not finished, because the next step is to take the generated output and publish it as a build artifact. The reason for that, is that I want to deploy from a release pipeline and for that, I need to transmit the "website" and the artifact is the way to do it.
> Note that we copy the _site folder containing the website but also the deploy folder which contains the scripts for the instructure as code (seen later in this article)

![Copy files](./images/jek-copy.png)

And to finish, we package it as an artifact
![Building the artifact](./images/jek-publish.png)

## Provision a blob storage (Infra as code)

Like I said, for a static website, it could be very useful to host our site on a blob storage with the static website feature enabled. We could do that directly in the [Azure Portal](https://portal.azure.com), but I prefer to use Infrastructure as Code, since it's an obvious best practice.

The following ARM (Azure Resource Manager) file contains three parts:

- one for variables because I like to put dynamicity in the script. I could inject parameters here directly in case of multi-environments
- one middle part creating the storage account (the blob storage)
- one last part to export the name of the blob storage because I'll use it in the release pipeline. I want it to be fully dynamic in case of changing the name of the storage.

``` json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "variables": {
        "appPrefix": "devopsjourney",
        "storageName": "[concat('sto',variables('appPrefix'))]"
    },
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2018-07-01",
            "name": "[variables('storageName')]",
            "location": "[resourceGroup().location]",
            "tags": {
                "displayName": "[variables('storageName')]"
            },
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "StorageV2"
        }
    ],
    "outputs": {
        "outputStorageName": {
            "type": "string",
            "value": "[variables('storageName')]"
        }
    }
}
```

This json file should be stored besides the source code in Git, this way, whenever you'll deploy your application, you'll deploy the underlying architecture.

Note that we create a standard storage account. The fact is that for the moment, it is not possible so enable the static website feature with Azure. We have to enable it through the Azure API and the easiest way to do it is using the Azure CLI. Keep this for later.

``` powershell
az storage blob service-properties update --account-name MY_STORAGE_ACCOUNT_NAME --static-website  --index-document index.html
```

## Set up of the release pipeline

It's time to build our release pipeline. Go on the Releases menu in Azure DevOps and create a new pipeline and select the relevant artefact by selecting the build pipeline we just created. Then create an empty first step.
Within this step, let's add few tasks (it could be done with only two but I want a fully dynamical pipeline):

- one "Azure Resource Group Deployment" task to generate the infrastructure (our storage account)
- one "ARM Outputs" to extract the data from the generated infrastructure (we want to retrieve the name of the storage account)
- one task "Azure Cli" to execute our command to enable the static website feature
- one task "Azure File Copy" to publish the files on the blob storage

![Release](./images/jek-rel1.png)

For the first task, you just need to fill the fields and select the ARM template file

On the second task (ARM output), don't do anything. And on the Azure CLI task, copy paste the following command:

``` powershell
az storage blob service-properties update --account-name $(OUTPUTSTORAGENAME) --static-website  --index-document index.html
```

![Release](./images/jek-rel2.png)

$(OUTPUTSTORAGENAME) is the format to say "inject a variable from the pipeline". And the pipeline knows this variable because in the ARM template file, at the end, we create the "ouput" variable and then we added the task to take the output and map it to a pipeline. As simple as it is. In our case, it would be the name dynamically generated by our infra as code. Thus, if one day, we change the name of the storage account, the pipeline will still be able to set up the feature static website and to upload the file on the right storage.

On the very last task, just specify the blob name and the container which MUST be $web for a static website.
![Release](./images/jek-rel3.png)

## Conclusion

That's all. You just learned how to build a full CI/CD pipeline for a jekyll webste (could be any techno) and host it on a blob storage, reducing the price of the hosting of few cents per year ! Oh yeah :)