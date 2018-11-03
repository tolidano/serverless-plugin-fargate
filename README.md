# TL;DR: 

Build a docker container using CodeBuild, save to ECR, deploy to Fargate, and optionally create an ALB as target for API Gateway

# Introduction

Let's be honest - the term "serverless" is a misnomer. There are still servers all over the place. Your code in a Lambda is just packed up into a very small docker container that runs as required on some EC2 instance somewhere. This is the (current) edge of where we can abstract away the concept of software engineering - you write the code, and someone else handles everything else. 

There are 3 common issues with Lambda though:
1. 15-minute maximum runtime
2. CPU is defined as a function of memory
3. Prewarming is hacked by periodically executing your Lambda in a no-op fashion at the appropriate rate

Enter Fargate, which removes some of the abstraction in exchange for more control. Fargate uses Amazon's own EC2 hardware fleet to run containers for you. In this way, you still don't need to provision EC2 hardware or worry about auto-scaling or Kubernetes configurations. However, you resolve all of the issues above in a sane and stable way. The container can remain running as long as you like, can be running adequately before an anticipated traffic spike, and the CPU and memory are defined independently, so your CPU intensive jobs can run in 256MB of RAM and your memory hogs can still run on a single thread of a CPU.

# Methodology

This plugin is implemented to fire after the deploy:deploy phase of the core serverless framework. At that point, you have a valid code package uploaded to Lambda and an API Gateway pointing to it.
