# Overview

## Why Kubernetes

Today users expect applications to be available 24/7, while developers expect to deploy new versions of those applications several times a day. The way we build software is moving in this direction, enabling applications to be released and updated in an easy and fast way without downtime. We also need to be able to scale application in line with the user demand and we expect them to make intelligent use of the available resources. Kubernetes is a platform designed to meet those requirements, using the experience accumulated by Google in this area, combined with best-of-breed ideas from the community.

## What you will learn

By the end of this bootcamp you will understand what Kubernetes does. You will also learn how to deploy, scale, update and debug containerized applications on a Kubernetes cluster.

## How you will learn

This practices are based on usual use cases. The idea is to solve it by applying some techniques, describe in the main part of the course or performing nano projects like managing the same application across all exercises to understand the workflow on a Kubernetes cluster.

To make it successful, participants need to be prepared in advance, by going through the recommended pre class content to clear the concepts as well as by making sure their systems are prepared with all the software required, before the class. The pre class checklist given below can be handy for this purpose.

## Who is this for

* This course is for someone who has already taken docker fundamentals course/has equivalent knowledge, and would like to learn how to extend it to orchestrate container deployments at production scale with Kubernetes.
* If you are a Operations/Systems personnel and would like to learn how scalable, fault tolerant and high available infrastructures are built on or off cloud to orchestrate container based deployments, this course is for you.
* If you are a developer and would like to learn how to deploy your application stacks in production, on top of scalable, highly available and leverage features provided by Kubernetes, this course is for you.
* If you are a QA, and if your organization has a staging/QA environment built on Kubernetes, and you would like to understand how to leverage it for setting up automated test workflow and learn the primitives offered by kubernetes, this course is for you.
* You could be developer/operations personnel and be in charge of securing application infrastructure and setup auxiliary services such as monitoring, centralized logging etc. this course is for you.

## What will you do as part of this course

As part of this course you will :

* Design a scalable, resilient, secure solution for deploying microservices application stack in production using primitives offered by Kubernetes.
* Learn how applications from a micro services stack interconnect with services as well as expose public facing services with various options including nodeport, externalIP, ingress, ...
* Achieve Continuous Deployment with different release strategies such as Zero Downtime, Blue/Green, Canary, ...
* Learn how to manage persistent storage in a Kubernetes environment.
* Deploy an application which is auto scalable, high available, and resilient to failures.

## What is not covered

Even though this course covers many concepts related to Kubernetes, it still has the following areas uncovered :

* Cloud specific provisioning and integration
* HA deployment of a Kubernetes cluster with multi masters
* In depth Kubernetes administration
* Writing Micro Services Applications
* Alternate container runtimes e.g. rocker/rkt, runc, CRI-O

## Which prerequisites are needed

Following are the prerequisite skills to attend this course. Since its a beginner level course, no prior experience with Linux containers is assumed.

* Docker Basics
  * Running Containers
  * Building Images and writing Dockerfiles
  * Docker Compose
  * Docker Networking and Storage
* Linux/Unix Systems Fundamentals
* Familiarity with Command Line Interface \(CLI\)
* Fundamental knowledge of code editors
* Understanding of YAML syntax and familiarity with reading/writing basic YAML specifications

## Where to train

There is a lot of way to trained you to manage a Kubernetes cluster and the resources associated :

* Online with:
  * The labs [Play With Kubernetes \(PWK\)](https://labs.play-with-k8s.com/)
  * The labs [Katacoda](https://www.katacoda.com/courses/kubernetes)
* Locally on a dedicated cluster deployed on[ Vagrant instances](https://github.com/wikitops/ansible_kubernetes)
* On a cloud dedicated cluster deployed thanks to [Kubespray](https://github.com/kubernetes-incubator/kubespray)
* On a single local instance thanks to [Minikube](https://github.com/kubernetes/minikube)

## Where to document

To go further in the documentation of Kubernetes objects and principles, please refer to those documentation platform :

* [Kubernetes Official documentation](https://kubernetes.io/docs/home/)
* [Kubernetes Official blog](https://kubernetes.io/blog/)



