Modificar service-account to add all secrets credentials. pipelines
Modificar imagem maven task
Modificar SecurityContextConstraints para permitir executar como sudo.
Modificar o nome do secret credentials-application no pipelie
Modificar service-account argo to add all secrets credentials.
oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller

# Quarkus CI/CD with OpenShift Pipelines, OpenShift GitOps, Kustomize, and Quay.io

Welcome to the ultimate blueprint for integrating OpenShift Pipelines (Tekton), OpenShift GitOps (Argo CD), and Kustomize to seamlessly deploy your Quarkus applications! This repository serves as a comprehensive example showcasing how these powerful tools work together, leveraging GitOps principles to streamline your cloud-native CI/CD workflows.

## Why Use This Repository?

- **OpenShift Pipelines (Tekton):** Automate building, testing, and pushing container images seamlessly.
- **OpenShift GitOps (Argo CD):** Effortlessly manage application deployments, ensuring continuous delivery through GitOps practices.
- **Kustomize:** Customize and manage Kubernetes manifests efficiently and declaratively for different environments.
- **Quay.io Integration:** Securely store, manage, and deploy your container images using Quay.io.
- **Quarkus Framework:** Benefit from rapid boot times, low memory footprint, and a superior developer experience optimized for cloud environments.

## What You'll Achieve

By exploring this repository, you'll learn how to:

- Create a robust CI/CD pipeline leveraging OpenShift Pipelines (Tekton) to build and push Quarkus application images to Quay.io.
- Automatically synchronize and customize deployments to your OpenShift clusters using OpenShift GitOps (Argo CD) and Kustomize.
- Implement GitOps best practices for secure, reliable, and reproducible deployments.

## Get Started

Dive in and discover how to elevate your cloud-native development practices today!

## Operators Instalation

### Openshift Pipelines

### Openshift GitOps

## Configuration

### GitOps

In order to allow argocd to create the project in openshift is necessary add the "cluster-role" to the service-account, it is possible to achieve this using the command bellow

```bash
oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller
```
