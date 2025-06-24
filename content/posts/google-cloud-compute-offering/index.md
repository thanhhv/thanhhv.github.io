+++
title = "Google Cloud Compute Offering"
date = "2025-06-23T13:00:00+07:00"
draft = false
tags = ["cloud", "GCP"]
+++

In Google Cloud, **Compute Offering** is a set of services that help you run workloads (like applications, APIs, batch jobs, etc.) on Google's host infrastructure. Depending on your workload and requirements, you can choose from the options below:


![gg-cloud-comute-offering](gg-compute-offering.png)

## 1. Compute Engine (IaaS - Infrastructure as a Service)
- A virtual machine where you have full permission to configure the environment.
- High level of control, similar to traditional hosting on the cloud.
- **Use case:** Configure OS in detail, install custom software, need full system control.
- **Example:** Run a Node.js/Golang server, cron job, or backend AI model training.
- **AWS equivalent:** EC2

## 2. App Engine (PaaS - Platform as a Service)
- A platform for running applications where you only need to deploy code — Google handles everything else (scaling, patching, infrastructure, etc.).
- Low level of control — just push your code, no need to manage servers.
- **Use case:** Rapid development, no infrastructure management needed.
- **Example:** Run a small API server, REST API, or MVP web app.
- **AWS equivalent:** Elastic Beanstalk

## 3. Cloud Run (Serverless Containers)
- Run containers (e.g., Docker) in a serverless model. Billing is based on request duration.
- Medium level of control — you manage the container image, while Google handles scaling and infrastructure.
- **Use case:** Leverage the benefits of containers + serverless for lightweight backends or microservices.
- **Example:** Run an API service, webhook receiver, or AI inference container.
- **AWS equivalent:** App Runner / Fargate

## 4. Google Kubernetes Engine (GKE) (CaaS - Container as a Service)
- Run containerized applications with Kubernetes. Google manages the control plane.
- High level of control — you can fully configure your cluster, while Google manages some parts of it.
- **Use case:** Need orchestration, CI/CD pipelines, or complex multi-service applications.
- **Example:** Run a microservices system, CI job runners, or real-time data pipelines.
- **AWS equivalent:** EKS

## 5. Cloud Functions (FaaS - Function as a Service)
- Write and deploy small functions that are triggered by events (HTTP, Pub/Sub, Cloud Storage, etc.).
- Extremely easy to use — just write a function, no need to manage infrastructure.
- **Use case:** Handle small event-driven logic, time-based triggers, or system integration tasks.
- **Example:** Auto-resize images, handle webhooks from Telegram/Stripe, send emails on user registration.
- **AWS equivalent:** Lambda
