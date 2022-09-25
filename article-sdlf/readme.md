# Restarting With AWS Serverless Data Lake Framework

## Introduction

In this article we will discuss about AWS Serverless Data Lake Framework [link here], features, pitfalls and how to get started and install it really fast. SDLF, according to the readme on GitHub is an AWS Professional Services open source initiative. Citing the source:

"_The Serverless Data Lake Framework (SDLF) is a collection of reusable artifacts aimed at accelerating the delivery of enterprise data lakes on AWS, shortening the deployment time to production from several months to a few weeks. It can be used by AWS teams, partners and customers to implement the foundational structure of a data lake following best practices._"

This is exactly what we were looking for to adopt, as a company wide framework, to get started with several clients in need of a new data-lake. SDLF proved to be an excellent starting point for our pipelines. Instead of just writing some glue jobs here and there and, when done trying to harden them and bring to a well-architected standard, we choose SDLF which firstly install a well-architected and hardened scaffolding, and then writing our glue jobs the right way to fit into this framework.

In terms of documentation, we found plenty to get us started. If you are not familiar with SDLF, here are every thing there is so far:

- website based on docs: <https://sdlf.readthedocs.io/>
- workshop: <https://catalog.us-east-1.prod.workshops.aws/workshops/501cb14c-91b3-455c-a2a9-d0a21ce68114/en-US>
- source code: <https://github.com/awslabs/aws-serverless-data-lake-framework>

However, here is a brief of how it works in our own words: SDLF comprises in a set of repositories that deploys a CICD ready (with CodeCommit, CodeBuild, CodePipeline and all) data lake pipeline scafolding and examples. Pipelines can contain multiple stages (default 2). The deployment considers multiple environments: shared environment, dev, test, prod environments. For a quick evaluation and get-started, shared and dev environments can be deployed on the same account.

![sdlf architecture](./pictures/2022-09-25_13-09-05-architecture.png)
[image copied from <https://sdlf.readthedocs.io/en/latest/architecture.html>]

The greatest advantage we saw in using SDLF was that it contains all things that we need in a data-lake other than the actual transformations. Here is the list of these things, or in other words, things we dont have to care about if using SDLF:

- highly secure design
- projects or repositories separated by functionality
- CICD pipelines ready made, that triggers by pull-request approval
- all data at rest between stages, and final stake encrypted
- tightly integrated with AWS Lake Formation. SDLF actually looks like a natural addition on top of Lake Formation. Final stage in data lake is added a Lake Formation database.

## Install SDLF

One of the first things we created around SDLF, is an ansible set of scripts for deploying it in one shot, and then completly destroing it in another shot. This would be the first step in having an automated testing procedure.

We published these changes as a new branch as part of forked SDLF repository on our GitHub account. This is available here:

<https://github.com/PredictIf/aws-serverless-data-lake-framework>

The install script deploys SDLF in a AWS account that plays SHARED and DEV environment roles. It will create CodeCommit replicas of several folders from the github original repo.

### Requirements

- A linux type of development environment where you will clone our branch. Cloud9 will work fine. I will not work on Windows due to limitations of the max number of characters a file path can have.
- Git
- Ansible
- Pyhon 3.8+
- Library: git-remote-codecommit
- AWS credentials profile pointing to an account to be used as SHARED and DEV environments for SDLF. It's important to mention that the region must be also configured as part of the profile. Another important detail is that there should not be other copy of SDLF installed or attempted installed in another region or same region. This is caused by the name colision in global names of IAM roles and policies. However, if you really want to install it where SDLF is installed on another region, there is a work around that I will mention in another post. Or just post a note on the article.
- Your user, or the entity that will execute the ansible scripts, should be added as admin in Lake Formation. (For details, see step 5 in the [workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/501cb14c-91b3-455c-a2a9-d0a21ce68114/en-US/10-deployment/100-setup)).

### 1. Clone the repo

```bash
#!/bin/bash
git clone https://github.com/PredictIf/aws-serverless-data-lake-framework.git
```

### 2. Edit specific variables

First, you would need to edit config file correctly, before launching the install. With your editor of choice, open and edit following files:

- `./ansible/environments/vars_common.yaml`
- `./ansible/environments/dev/vars-app.yaml`

![sdlf deploy](./pictures/2022-09-25_13-02-11.-config.png)

<!-- add permissions for lake formation -->

### 3 Launch the Ansible Playbook for Install

After you are sure you edited config files correctly, launch the ansible playboook:

```bash
cd ./ansible
ansible-playbook deploy.yaml
```

You should see something like this:

![sdlf deploy](./pictures/2022-09-25_10-10-50-deploy.png)

After 50 to 60 min, you should have SDLF foundations with an organization that you specified and a team that you specified and a simple 2 stages pipeline - deployed.

## Verify SDLF Installation

If you uncheck `View nested` option, the list of installed cloudformation installed should look like this:

![sdlf cfn list](./pictures/2022-09-25_12-10-50-cfn-list.png)

Last step of installation will drop legislators json files in the raw bucket and trigger the out-of-the-box pipeline. Here is how step functions should look like after a successful install. Please note that Step A (sdlf-engineering-main-sm-a) has a failed execution. That is by design, in order to test the error triggering branch of the step function, and is triggered by `./sdlf-utils/pipeline-examples/legislators/data/regions.json` file.

![sdlf step function list](./pictures/2022-09-25_12-20-26-step-functions.png)

Everything is handled through CICD pipelines in SDLF. Here is the list of pipelines after a successful deployment.

![sdlf pipelines list](./pictures/2022-09-25_12-22-24-pipelines.png)

## Uninstall SDLF

Just deleting all installed cloudformation templates would not completly erase any trace of SDLF. Following items needs special attention:

- S3 buckets not empty
- encryption keys
- repositories in CodeCommit
- CloudTrail groups
- lambda layers
- System Manager parameters

Our ansible script `./ansible/destroy.yaml` will take care of all of them. You can execute it like this:

```bash
cd ./ansible
ansible-playbook destroy.yaml
```

It should take about 15 min.

The script will tries to delete cloudformation templates in parallel in order to finish the businessfaster. And is trying in subsequent steps, because some of templates may fail in the fist step. So you may see some messages like the one blow, but this should not be a cause of concern.

![sdlf destroy](./pictures/2022-09-25_12-46-52-destroy.png)

In the end everything should be wipped out. (everything related to SDLF)

## Conclusion

Overall and all in all, SDLF is a great framework, in our opinion. The team did an excellent job producing what we would usually do in the last phase of a rushed data-lake project, when a client asks to build a data-lake by Wednesday.
It deffinetly deserve more attention from devops. ....
