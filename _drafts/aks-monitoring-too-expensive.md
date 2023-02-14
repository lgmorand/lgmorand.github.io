---
title: 'Demo blog post'
date: 2022-12-12 00:00:00
description: AKS monitoring can be expensive. Let's see how you can cut the costs efficiently
featured_image: '/images/about/about.jpg'
---

Let's be straight, monitoring is not cheap. The more information you need, the **more you'll have to pay either for the processing or the storage of all this data**. If you implemented monitoring of your different clusters, you quickly discovered that it can really become expensive.

If you can't make all the costs disappear there are different ways to optimize the costs and that's what we are going to cover. We still talk about:

- Agent configuration
- Basic Logs
- Table transformation
- Data Collection rules
- Diagnostics settings


## Agent configuration


## Basic Logs


## Table transformation


## Data collection rules

## Diagnostics settings

As explained in this article, *diagnostics settings* are platform logs & metrics which are grabbed automatically by Azure but not stored by default (but still viewable in the "Metrics" panel, even if monitoring is disabled). They bring informations which are not present in ContainerInsights or prometheus, including the logs of the differents APIs of Kubernetes.

```` sql
Usage 
| where TimeGenerated > startofday(ago(31d))
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(TimeGenerated, 1d), DataType | render barchart

````



## Conclusion

All these tweaks won't allow to get a free monitoring solution but they should help to fine tune your usage and to pay only for what you need.
