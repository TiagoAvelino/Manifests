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

## Get Started

Dive in and discover how to elevate your cloud-native development practices today!

## What You'll Achieve

By exploring this repository, you'll learn how to:

- Create a robust CI/CD pipeline leveraging OpenShift Pipelines (Tekton) to build and push Quarkus application images to Quay.io.
- Automatically synchronize and customize deployments to your OpenShift clusters using OpenShift GitOps (Argo CD) and Kustomize.
- Implement GitOps best practices for secure, reliable, and reproducible deployments.

## Operators Instalation

### Openshift Pipelines

### Openshift GitOps

## Configuration

### Openshift GitOps

To enable Argo CD to automatically create and manage projects in OpenShift, you need to grant the appropriate permissions to the service account. You can do this by assigning the `cluster-admin` role:

```bash
oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller
```

Next, apply your GitOps configuration using the provided [Application Manifest](CD/application-kustomize.yaml):

```bash
oc apply -f CD/application-kustomize.yaml
```

Once applied, Argo CD will automatically create and manage the application deployment within your OpenShift project. It is best practice to maintain infrastructure manifests separately from your application manifests; however, this repository provides an example of integrating both.

![Application Deployment Example](/Images/ArgoApp.png)

## OpenShift Pipelines

To develop a pipeline, we first define the individual tasks that make up our pipeline. Let's explore each one individually. The pipeline diagram below illustrates our simple [example pipeline](#example-pipeline).

<a name="example-pipeline"></a>

![Example Pipeline](/Images/Pipeline.png)

### Pipeline Task Breakdown

Some tasks in this pipeline require access to private resources, such as container registries or GitHub APIs. These tasks rely on Kubernetes secrets that are mounted via workspaces. For example:

- The `buildah` task uses a secret (`my-quay-secret`) to authenticate and push images to Quay.io.
- The `create-pr` task accesses GitHub credentials stored in the `git-credentials` secret to open pull requests automatically.

#### fetch-repository

This task uses the `git-clone` cluster task to clone the application source code from the provided Git repository and branch. It ensures a clean state by deleting any existing content in the workspace before checking out the latest code. This is the foundation of the pipeline, as all other steps depend on having access to the application code.

#### build-jar

This task compiles the Java application using Maven. It executes `mvn clean package`, which removes previous build artifacts and generates a new JAR file. The Maven settings can be optionally provided via a workspace to include custom configurations, such as private repositories or mirrors.

#### convert-app-name

This task is responsible for standardizing the application name by converting it to lowercase. This is particularly useful when using the application name as part of image tags or Kubernetes resource names, which often follow lowercase naming conventions. It takes `APP_NAME` as input and outputs a `lowerAppName` result used in the next step.

#### buildah

This task uses the `buildah-com` cluster task to build a container image from the source code and push it to a remote image registry, typically Quay.io. It uses the lowercase app name as part of the image tag and references credentials provided via a Docker config workspace. The task returns metadata such as the image URL and digest for downstream use.

#### update-kustomize

This task modifies the Kubernetes deployment manifests using Kustomize. It updates the image reference with the newly built container image URL and digest, ensuring that the correct image is deployed. These updates are committed and pushed to the Git repository that holds the platform manifests, following GitOps practices.

#### create-pr

This task automates the process of opening a pull request in the platform manifests repository. It uses GitHub API credentials stored in a Kubernetes secret to authenticate. The PR merges changes from the development branch into the destination branch (e.g., `hml`), enabling review and promotion of deployment updates.

Before running this task, make sure to apply the necessary secrets:

```bash
oc apply -f CI/Secrets/quay-secret.yaml && \
oc apply -f CI/Secrets/secret-application.yaml
```

After appling the secrets, you will need to apply the secrets into the service account, to do it its necessary to apply the SA manifest:

```bash
oc apply -f CI/SA/roles.yaml
```

Also, make sure you have the custom tasks needed by the pipeline applied to your cluster:

```bash
oc apply -f CI/Tasks/commit-task.yaml && \
oc apply -f CI/Tasks/lower-case.yaml && \
oc apply -f CI/Tasks/pull-request.yaml
```

Finally, apply your Pipeline configuration using the provided [Pipeline Manifest](CI/Pipeline/pipeline.yaml):

```bash
oc apply -f CI/Pipeline/pipeline.yaml
```

## Running the Pipeline

After defining the pipeline and its tasks, the next step is to trigger a `PipelineRun` that executes the defined workflow.

The [PipelineRun Manifest](CI/Pipeline/pipeline-run.yaml) is an example of `PipelineRun` that links to the `java-pipeline` and provides all the necessary parameters and workspace bindings:

### Security Context

The `securityContext` defined in the `podTemplate` sets the file system group (`fsGroup`) to `653`. This is often required in OpenShift clusters to ensure that mounted volumes (like PVCs) have the correct group ownership, allowing the containers in the pod to write to them. The `runAsUser` is commented out but can be set if stricter UID control is needed.

### Workspaces

This `PipelineRun` uses three workspaces:

- **workspace**: Backed by a PersistentVolumeClaim (`workspace-pvc`) and shared between tasks for transferring data.
- **dockerconfig**: Mounts a Kubernetes secret containing credentials to push the image to Quay.io.
- **maven_settings**: Optionally mounts a Maven settings volume via `maven-settings-pvc` to include custom configurations during build.

To create apply the pipeline-run you should first apply the PVCs that the tasks in the pipeline will use to run the pipeline.

```bash
oc apply -f CI/PVCs/pvc-maven.yaml && oc apply -f CI/PVCs/pvc-workspace.yaml
```

After that just need to create the pipeline-run

```bash
oc apply -f CI/Pipeline/pipeline-run.yaml
```

## References

- [Kustomize Documentation](https://kubectl.docs.kubernetes.io/references/kustomize/)
- [Argo CD Documentation](https://argo-cd.readthedocs.io/)
- [Tekton Pipelines Documentation](https://tekton.dev/docs/pipelines/)
- [OpenShift Pipelines Documentation](https://docs.openshift.com/container-platform/latest/cicd/pipelines/understanding-openshift-pipelines.html)
- [OpenShift GitOps Documentation](https://docs.openshift.com/container-platform/latest/cicd/gitops/understanding-openshift-gitops.html)

       oc adm policy add-role-to-user system:image-builder -z pipeline
       oc adm policy add-role-to-user system:image-puller -z pipeline
