---
tags: [LA]
title: Containers & Orchestration
created: '2020-06-02T21:13:37.889Z'
modified: '2020-06-02T23:09:56.146Z'
---

# Containers & Orchestration

Containers: portable software. Run on a variety of systems reliably and consistently regardless of what system it is on. Speeds up development and automation and can run everywhere. 

Similar to VM but uses fewer resources than a VM, less overhead, can fit more containers on same server as VM.

VM has host OS, VM has a full copy of another OS on top of it (overhead), containers sit on host OS but containers run on the same OS. Faster, smaller, lightweight, efficient usage of available resources.

Orchestration: processes and tools to process and manage containers.

You could do it manually, or use a tool to spin up 5 containers. Good idea to spin them up on 5 different hosts incase one of them goes down. O-Tools make it easier to do that as well. 

Zero downtime deployment: no need to take server down for maintenance. A ZDD spins up new containers running new code, so the old containers are still up running old code that users can use. Once new containers are done, direct user traffic to new containers, after that remove old containers. ZERO DOWN TIME.

- run software consistently on different machines
- isolation: keep pieces of software separate from one another
- scaling: deploy and destroy as users need
- automation: save time and money
- containers uses resources more efficiently than vms


## Advantages and Limitations

- benefits of VMs while being lightweight
- boots software components needed rather than whole OS, so faster boot up
- smaller in size than VMs
All this adds to faster and simpler automation

Limitations:
- less flexible, cant run windows container on linux container, no full copy of kernel
- can be challenging and complex but tools can help with that

## Orchestration Tools
- Kubernetes
  - container orchestration tool. Build and manage container infrastructure and automation. Self healing apps, automated scaling, easy deployment
- Docker Swarm
  - docker's native orchestration functionality. built right into docker.
- Nomad
  - by hashicorp, simple and lightweight for basic orchestration functionality

## Use Cases
- Microservices: splitting application into a series of small independent services. Flexible and stable because you can separate functionality and manage it separately, less coupling. Can do it automatically and in real time with O-Tools
- Cloud Transformation: migrating existing infrastructure into the cloud. Grab existing app and wrap containers around these pieces of software then run in the cloud. Works for many cases but not all. Running on cloud you pay only for what you use.
- Automated Scaling: automatically provisioning resources in real time data metrics. So based on metrics as to when your service is being used, O tools can spin up and down instances based on user usage in seconds (minutes for VMs!)
- CD: deploy code automatically and frequently instead of big deployments every now and then. New functionality to customers faster. Easy to test code inside container in a deployment environment. What works in testing works in production with containers. Same image in test and production.
- Self-Healing: automatically detect when problems occur without involving a human. No need to wake up in the middle of the night to reboot a server. Containers can be automated to do that, but also can just replace the server or restart it just replaces it entirely. 
- Developer Visibility: "well it works on my machine" is no longer an issue and can find out what is actually wrong. The container **IS** the production environment.

## Containers in the Cloud

The cloud is "someone else's computer". Instead of physical data centers, you pay a cloud provider to run remote data centers over the internet and pay based on use. 

You can run containers on servers hosted on cloud providers. All the benefits we discussed so far are available in the cloud. You save money especially in comparison to VMs. Clouds offer container support out of the box. 
