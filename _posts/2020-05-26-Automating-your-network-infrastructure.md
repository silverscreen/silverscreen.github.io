---
layout: post
title: Automating your network infrastructure
description: Network automation does not an automated network make.
summary: How Not to Automate
comments: false
tags: [typography]
---

I will be exploring some of the open-source automation frameworks available, this case in particular Ansible and Nornir, and the popular version-control system Gitlab, to build a CI/CD (Continuous Integration and Continuous Delivery) pipeline for the deployment and tracking of network changes.

---- 

#### What Is Network DevOps

For quite some time now, DevOps engineers have worked around a set of principles, that sees the merging of development, QA, and the operational tasks of deployment and integraton, into a set of continued processes.

These processes not only provide a continous workflow to follow, but rather, they represent a cultural change. Where previously Development and Operations teams worked independently of each other, they are now contingent on one another.

This cultural shift in the way Development and Operations teams work, has been inherently slow to adopt within the world of network engineering. And whilst adoption of this way of thinking has steadily increased, there is still a struggle and sense of confusion as to how programming, through the use of development and software development techniques, can be used to support network automation along the path of software-defined networking.

#### Challenges Faced When Working With Networks

Networks are oftentimes complex, and you'll seldom find two different enterprises running the network in the same way.

Due to the aforementioned, this makes them generally slow to adapt. And rarely will you find a solution that can cover all your needs without some customisation, which in itself comes with a plethora of challenges.

No solution, both commerical and open source, is exempt from being superseded and or being discontinued.

Networks are usually 100 percent production.

It can be very hard, if not sometimes impossible, to test small portions of network configuration, unless of course you're lucky enough to own a test environment that isn't production.

### Test Environments

Traditionally, engineers tasked with building their own test environments have either purchased, or loaned, physical equipment to construct their own in-house labs. 

The problems with this are:

* A physical lab can require a lot of equipment, and cabling (not to mention space).
* The costs associated with acquiring kit can be high.
* There is an increased potential for errors due to complexity.
* Often not a one-to-one reproduction.
* Building a physical network, whether production or a lab, is time consuming.

The alternative, has seen virtual test environments gaining in popularity. With free options such as GNS3, an open source network simulator with scriptable APIs to stand up and tear down virtualised environments as part of a CI/CD pipeline. To vendors such as Cisco's VIRL and CML, as well as other offerings from Juniper and VMware, all providing extensive virtual test suites for creating, testing, troubleshooting, and modelling. 

#### The challenges with a virtual environment are: 

- Depending on what solution you choose, things can get expensive. 
- Regardless of what you choose, there are some things you still can't test. 
- Virtual hardware will never accurately simulate data plane traffic such as that provided by real hardware/ASICs.
- A virtual environment can be a copy, but it will never be a one-to-one reproduction.

Reproducing the complexity of an Enterprise network in a lab is often not feasible for a lot of businesses. Therefore the remaining option most are faced with is to rely on incremental changes deployed to a production environment with sufficient monitoring, along with the ability to rollback upon the first sign of danger. And thus the importance of having a framework that can effectively catch, deploy and rollback these changes, within a controlled environment, is the reason why so many are looking towards implementing the DevOps style of working. 
