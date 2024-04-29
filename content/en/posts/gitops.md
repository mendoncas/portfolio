+++
title = "The peace of mind of GitOps"
date = "2024-03-03T11:37:01-03:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["devops", "gitops", "kubernetes"]
+++

Recently, we have migrated all of our services from a DC/OS Marathon cluster to an Openshift one. 
This was quite a big step for our organization, as we had to adapt to a completely new platform. 
This included finding more modern and efficient ways to architect our CI/CD pipelines. 
As i researched the many different continuous delivery mechanisms that can be implemented, 
i stumbled upon the pull-based pipelines and the GitOps paradigm, and it was love at first sight. 

Here, i´m going to share my experience implementing the GitOps paradigm and using ArgoCD as the tool of choice for  continuous delivery.

## GitOps, compared to the traditional approach

First, it is important see the difference and main advantages of GitOps compared to the traditional approach, the push-based pipelines.
Push-based pipelines, simply put, are pipelines that *push* to your deployment at the delivery step of your CI/CD. As an example, 
a traditional pipeline that deploys to kubernetes might end with the command:

```bash
kubectl apply -f deployment.yaml
```

In contrast, a pull-based pipeline relies on a tool that *polls* a certain resource for the desired state and applies it to your cluster.
In our case, the tool of choice is ArgoCD, and the polled resource is a git repository. 
The practice of utilizing of a git repository as the single source of truth for deployment
configuration is what we call GitOps. This guarantees a very important property: 
every deployment configuration of each application is subject to version control. This allows us to easily track, manage and, 
if necessary, rollback to previous states.

## Adoption and implementation
After setting up an ArgoCD proof of concept pipeline, i presented it to my colleagues. 
At first, the delivery step was counter-intuitive, however, my colleagues quickly noticed the value of version-controlling our 
deployments and we collectively agreed that this approach would be the one. 
This was an important step, considering that DevOps is more about culture than anything else, it is important to have the whole team on the same page.
Once our team decided to go with ArgoCD, we had to decide how to actually implement our new pipelines. Here´s how we did it:

### Monorepo
Our single source of truth is a monorepo with files for every application. 
For each branch of each application, we have a helm chart for it´s corresponding deployment,
following this format:

```bash
git/path/to/application
├── branch-name
│   ├── Chart.yaml
│   ├── templates
│   └── values.yaml
```
This allows us to easily find every app, considering that the path to it inside the monorepo is equal to the application´s path on our git server.
At first, i was worried that the monorepo would get too big and end up slowing down our pipelines, but it turned out to just not be an issue.

### Pipelines
The "delivery" steps of our pipelines got a lot simpler, and we ended up with something similar to this:

```bash
git clone monorepo
yq -i '.image = "new-image-name"' values.yaml # update image name in values.yaml
git commit . -m "commit message"
git push
```
As ArgoCD is only responsible for watching over a git repository and deploy changes, we were able to keep our CI steps exactly the same.

## The results
Migrating to a whole new platform and deployment process was quite a lot of work, but the process was relatively simple.
The benefits were exactly as expected: no more stress over mysterious config files on mysterious locations; or lost, non-recorded changes!
There is a peace of mind in DevOps that only version-controlled deployments and simple deliveries can bring!

## Related readings and references
[Security benefits of pull-based pipelines, by alxk](https://alex.kaskaso.li/post/pull-based-pipelines)
