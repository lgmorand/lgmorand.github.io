---
title: 'Azure DevOps - pipeline decorator - part 1'
date: 2022-12-12 00:00:00
description: How to use pipeline decorators, an unknown super powerful feature of Azure DevOps
featured_image: '/images/blog/cover.jpg'
---

## Discover the pipeline decorators in Azure DevOps

It's been few years (10+) that I'm helping customers implement DevOps process and tooling, including Azure DevOps. In 2022, I'm still surprised how few people know this powerful feature which are the pipeline decorators. These decorators are on way to customize workflows at scale in the whole organization without having to edit each workflow.

This suject will be covered in four parts:

- Part 1: What are the pipeline decorators and create our first one
- Part 2: Deploy our decorator and enhance it
- Part 3: Create a more advanced decorator: a docker linter
- Part 4: Create another advanced decorator: a cred scanner

### What are the pipeline decorators ?

The pipeline decorators are pipeline tasks which can be injected automatically in all workflows on an Azure DevOps organization




Our pipeline decorator is ready, it's no time to publish it to our organization. It will be covered in the second part on this article.


### Create a publisher account

Since Azure DevOps Services is a SaaS offering, the only way to customizing your own organization is install extensions but these extensions have to come from the [Azure DevOps Marketplace](https://marketplace.visualstudio.com/azuredevops). In our case, we need to publish our pipeline decorator (which is a extension) but do so, we need do it under a publisher name even if, as we'll see later, nobody except us will be able to see/install our extension.

Go to the [management portal](https://marketplace.visualstudio.com/manage) which should offer you to create your publisher account. Be aware that the account which who you create the publisher account is important, as this account would also need to be an administrator of your Azure DevOps organization

> During the creation of my publisher, I discovered that my ID was used already. The existing publisher was created using another account when I was younger. I tried to delete the existing publisher without success until I discovered a clean solution which I [documented here](https://lgmorand.github.io/blog/delete-publisher).






### Upload your extension

It's not time to upload our extension on the marketplace. For that you need:

- the generated extension (*.vsix)
- ensuring the ID contained in its manifest (vss-extension.json) matches exactly the name of the publisher account you just created




![Extension uploaded](../images/blog/azure-devops-pipeline-decorator/upload%20extension.png)

Once the extension is uploaded and validated, you can check the result by opening the contextual menu (...) and choose "*View extension*" to see the final page of your extension. That's what a user would see if your extension was public:

![Extension home page](../images/blog/azure-devops-pipeline-decorator/extension-page.png)

We still need to make it available from our org. To do so, from the context menu, click on "*Share/Unshare*" to open a side panel when you can type the name of organizations you want your extension be available:

![Share the extension](../images/blog/azure-devops-pipeline-decorator/share-with-org.png)

Now, go back in your Azure DevOps organization. Once there, open the *Organization settings* and the *extensions* tab. This screen shows the **Installed** extensions, the **Requested** extensions which are the extensions users without enough right tried to add to the organization and the **Shared** extensions. That's in this last screen that you should now see your custom decorator.

![The extension is shared with the org](../images/blog/azure-devops-pipeline-decorator/shared-within-org.png)

It only means that the extension is available but it does not mean it is installed. You must now click on the extension and select "*Install*" to fully install it on your organization.

### Time to check that the decorator is working

This part is the easiest one. We added no specific conditions to our manifest expect that it should run for any pipeline (classic or YAML) by using the property *ms.azure-pipelines-agent-job.pre-job-tasks*.

Go and run any of your pipeline and check for the result.
