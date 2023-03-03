---
title: 'Azure Container Registry Caching - why is it great'
date: 2023-03-10 00:00:00
description: This new feature for ACR will improve your image management
featured_image: '/images/blog/acr-caching/new-rule.png'
---

## ACR Caching

Few days ago, Azure released a new [preview feature for Azure Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/tutorial-registry-cache). This first is more critical for your governance than it seems.

When you are deploying containers, they come either from your internal registry with custom container images or from a public docker registry. When your development teams use these public containers it forces your network to allow the connection to the Internet but also allows any container image to be deployed in your environment. You have no way to filter them nor to control their security.

This feature transforms your Azure Container Registry as a caching proxy. Instead of pulling from an external registry, you query your own ACR and it will be responsible to download and to cache the image for you. But it will only do it for declared images (named `caching rules`).

![ACR Caching](../images/blog/acr-caching/acr-caching.png)

Adding to the notion of caching and performance, it will allow different scenarios around governance and security (spoiler: but not yet).

## How to use it?

For the moment, you can only enable it and configure it through the Azure Portal but it should quickly come with CLI and infra as code too.
Open your container registry and open the `Caching Rules` blade

![Caching rules](../images/blog/acr-caching/caching-rule.png)

From there, you can create up to 50 caching rules, one per container image you'd like to cache. Give a name to your rule, the name of the container image (its URL) and the alias you want to give in your ACR.

![Create a new rule](../images/blog/acr-caching/new-rule.png)




> Note: Caching for ACR does not automatically pull new versions of images when a new version is available. For every new image available, a new pull request must be complete. It may change in the future.

## Governance

The real value of this feature is that it allows you to put a policy that only allows trusted registries in your clusters. You can use OPA/Gatekeeper and [Azure policies for Kubernetes](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/policy-for-kubernetes).

[One policy in particular](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Kubernetes/ContainerAllowedImages.json) only allows image pulls for whitelisted registries.

![Unique container registry](../images/blog/acr-caching/policy-acr.png)

Until now, when this policy was enabled, when a project team wanted to use a public image, they add to pull the image locally, push it in the registry and then pull it in the Kubernetes cluster. This new feature will simplify this part.

## Next steps

Of course, this preview is quite incomplete but it paves the way to something great. For instance, we could have only one place for all container images and we could enable security scanning on it (which is in the roadmap).

## Conclusion

It cost almost nothing (only storage used by imported images) but brings more control, best performance and is the first step for your governance. It should be your default choice.

ps: be careful with [the current limitations](https://learn.microsoft.com/en-us/azure/container-registry/tutorial-registry-cache#preview-limitations) especially the number of caching rules.
