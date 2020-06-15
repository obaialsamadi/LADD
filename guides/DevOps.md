---
tags: [LA]
title: DevOps
created: '2020-05-23T21:56:28.873Z'
modified: '2020-06-06T00:17:11.896Z'
---

# DevOps
## Tools

### Build Automation

Definition: Automated processing of code in preparation for deployment. This is based on what framework and programming languages.

- Java: ant, maven, gradle
- Javascript: npm, grunt, gulp
- Make: for Unix
- Packer: build machine images and containers (not so much for code)

### Continuous Integration

Definition: Continuous merging of code into a single branch or mainline. (Server integrates with source control)

- Jenkins
- TravisCI: built around Github Integration

### Configuration Management

Definition: Managing and changing the state of pieces of infrastructure in a consisstent and maintanable way. Good way to support infrastructure as code.

- Ansible: uses YAML files. No control server needed. Ansible Tower can be used to manage automation. No agents (just python and ssh to connect to host and make changes)
- Puppet: managed through UI. Uses control server agent architecture.
- Chef: procedural. Agent/server. Uses Chef DSL
- Salt: can respond to events in real time.

### Virtualization(1) and Containerization(2)

(1) Definition: Managing resources by creating virtual machines, not physical ones.

- VMWare ESX and ESCi
- Microsoft Hyper-V

(2) Definition: lightweight, isolated packages containing what you need to run a piece of software. Requires fewer resources than VM (VMs contain entire OS plus virtual versions of the hardware). Containers have bare minimum needed to run the software. It simulates the OS in a lightweight fasion. 

- Docker

### Monitoring

Definition: collecting and presenting data about the state and performance of applications.

- Sensu: infrastructure monitoring
- NewRelic: infrastucture & application monitoring
- AppDynamics: application monitoring 

Aggregation and Analytics: do something with the data you collected

- Elastic Stack

### Orchestration

Definition: Automation that supports processes and workflows, such as provisioning resources. Scale applications on request, auto scale on usage, self healing systems in regard to nodes.

- Docker Swarm : docker-native, made specifically for Docker containers. 
- Kubernetes: manage containerized apps across multiple hosts
- Zookeeper: works alongside Kubernetes
- Terraform: combines orchestration and infrastucture as code. Works well with Ansible, AWS, and Kubernetes

## Cloud

DevOps is not the cloud. It is a culture. The cloud is remove servers on the net offering services instead of locally hosted solutions. The practices of DevOps are useful in cloud. They support each other.

**IaaS:** infrastucture as a service, provides low level infrastructure, a bare OS. You configure the rest.

- AWS EC2
- Azure
- GCE

**PaaS:** platform as a service, standardized OS, you just manage application and the data.

- AWS Elastic Beanstalk
- Heroku
- Google App Engine

**SaaS:** Software as a service. Everything is managed for you, you just use. 

- Gmail
- MS Office

**FaaS:** also called serverless is known as function as a service. Everything is abstracted. you deploy small single purpose functions. You don't deploy applications, just functions you need. 

- AWS Lambda
- Azure Functions
- GC Functions


















