---
layout: single-with-toc
weight: 1
title: Introduction
preamble: >
  Welcome to the Nelson documentation - firstly, thanks for taking the time to look into the project. This section contains information about what Nelson is, and the kinds of problems it can solve. If you're looking for practical user documentation, please [look at the detailed documentation](/documentation/) instead.
menu:
  main:
    identifier: intro
    url: /introduction/
    weight: 1
---

# What is Nelson?

Nelson is a system for continuously deploying containers, and automatically managing their lifecycle. An application lifecycle refers to starting an application and then later wanting to replace that same application with say, a newer version. Nelson provides the plumbing to make that happen automatically: you simply focus on shipping changes to your repository, and Nelson orchestrates the launching, running, and eventual sunset of a given application revision.

## Git-centric

Everything must be versioned. Nelson adopts a hardline position on configuration and changes made to systems at runtime: with two explicit exceptions, no application shall change its operable configuration. This means that your configuration is code and should be checked in along with your application. This includes the declarative manifest file that Nelson uses to receive its instructions. If you need configuration changes, modify it, check it in and redeploy.

Adopting this approach has several discrete advantages:

+ Reduced deployment risk: by deploying smaller sets of changes more frequently, you can know exactly what change set went into a particular version. There are several development workflows available with Nelson; you can choose your deployment cadence and the amount of code that goes into a specific release.

+ Container images used at runtime are essentially inert and the container registry / image store does not need to be tightly secured, because no private information or system secrets are in the container a-priori.

+ Every single change to the system has an audit trail, thanks to Git.

## Safe and Secure

Runtime containers does not hold any credentials or other secret material until the very moment they are launched by the scheduler. The credentials given to the container dictate the access permissions the container has, and Nelson automatically provisions access policies based on what information it was told about the deployment. This has some nice properties:

+ You cannot interact with a system or resource that you did not tell Nelson about.

+ System auditing is a breeze: query the graph to see what applications use which secure resources.

+ In the event of a compormised system, simply instruct nelson to redeploy the affected system(s) and the new deployment will have an entirely new set of credentials and the old permissions and policies will all be revoked and deleted.

Practically speaking, credentials are sourced from [Vault](https://www.vaultproject.io/) (or a KMS of your choosing), and then mounted to a `tempfs` attached to the container (an in-memory filesystem). You can only source credentials from Vault, for which you have a valid Vault Policy. These policies are dynamically generated by Nelson, and provisioned for you automatically at deployment time, and deleted automatically when the deployment is later torn down.

## Consistent

By virtue of the fact that *Nelson* is orchestrating application deployments over potentially many target datacenters, it is important to realize that there is a subsequent lack of transactionality in the operations *Nelson* takes. This is most apparent when deploying a new revision of a particular system: application code can never assume it is a "singleton" or in some way special, as the minimum number of versions running - even if it's for a very short time - will be greater than one. *Nelson* will *eventually* deliver on its promise and make one revision the primary revision of a system by cleaning up the others, depending on the particular circumstances, that process might not be immediate, and is most certainly not atomic.

This means that application creators have the following constraints:

1. Applications which require singleton behavior can either choose to implement application-layer leader election, or use convergent data structures to make sure that all overlapping changes will always commute.

2. Data corruption can - and will - eventually happen and applications need to be able to recover from this. Typically this means checkpointing data writes to limit the blast radius of any potential corruption (more appropriate for batch-style processes), engineering teams should properly evaluate the possibility for corruption and recovery in their particular use case.

The authors of *Nelson* full appreciate that these constraints require more engineering work. However, by applying these constraints it means *Nelson* can provide a guarantee around several critical behaviors, and set the right expectation from the start about application lifecycle. Moreover, these constraints represent best practice from a distributed systems perspective, and acknowledge the reality of discrete, interactive systems.

<hr />

# What Nelson Is Not

Nelson is not a monolithic solution that solves all problems. Nelson follows the classic unix philosphy: it is highly composable with other tools, which allows the system to be non-perscriptive, whilst still itself delivering value. There are a few scenarios which were very deliberitly ignored:

+ Nelson does not support ad-hoc deployments: users cannot deploy random, unversioned containers from their desktop; this is a very deliberate design feature.

+ Nelson works best in a poly-repo environment, where builds focus on small, atomic units. Nelson can work with explicit tagging and releasing for those with a mono-repo style of source control, but it will almost certianly require some propietary tooling - this will always be out of scope for Nelson and its ecosystem.
