# Lab 1. Title

Let's investigate how Helm can help us focus on other things by letting a chart do the work for us. We'll first deploy an application to a Kubernetes cluster by using `kubectl` and then show how we can offload the work to a chart by deploying the same app with Helm.

The application is the [Guestbook App](https://github.com/IBM/guestbook), which is a sample multi-tier web application.

## Scenario 1: Deploy the application using `kubectl`

```bash
git clone https://github.com/IBM/workshop-template
cd workshop-template
```
