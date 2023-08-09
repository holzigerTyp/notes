# Transform deployment using helmfile to ArgoCD

> How can I transform my helmfile deployment into an ArgoCD deployment? What changes are required in order to deploy an application using ArgoCD? What are best practises?

# Table of Contents

- [Base idea](#base-idea)
- [Current folder structure (using helmfile)](#current-folder-structure-using-helmfile)
- [Target folder structure](#target-folder-structure)
- [Approaching ArgoCD](#approaching-argocd)
  - [Creating applications](#creating-applications)
  - [Separate helm chart](#separate-helm-chart)
  - [Setup ArgoCD project](#setup-argocd-project)
    - [Creation](#creation)
    - [Destinations](#destinations)
    - [Cluster Resource Allow List](#cluster-resource-allow-list)
  - [Add repository to ArgoCD](#add-repository-to-argocd)
  - [Setup ArgoCD root application (using UI)](#setup-argocd-root-application-using-ui)
  - [Finish up](#finish-up)


# Base idea

ArgoCD is a *declarative continuous deployment for Kubernetes* as the [argoproj](https://github.com/argoproj) calls [their repository](https://github.com/argoproj/argo-cd). With ArgoCD you are able to automate your deployments and take a step forward into GitOps. It uses some custom resource definitions (CRDs) but the main part is the *Application* (applications.argoproj.io). The application defines the root of your deployment including the repository, target revision, helm values and vice versa.

To install ArgoCD in your environment please take a look at the [official documentation](https://argo-cd.readthedocs.io/en/stable/getting_started/).

This documentation assumes the following:
- ArgoCD instance, installed and configured, located at `argocd.example.com`
- Repository located at `https://git.example.com/example/kubernetes.git`
- One or more helm charts containing the resource definitions for your application

# Current folder structure (using helmfile)

```
├── helmfile.yaml
├── example
│   ├── Chart.yaml
│   ├── templates
│   │   ├── deployment-frontend.yaml
│   │   ├── deployment-backend.yaml
│   │   ├── ingress.yaml
│   │   ├── service-frontend.yaml
│   │   └── service-backend.yaml
│   └── values.yaml
├── values-dev.yaml
└── values-prod.yaml
```

# Target folder structure

```
├── apps
│   ├── Chart.yaml
│   ├── templates
│   │   ├── frontend.yaml
│   │   └── backend.yaml
│   └── values.yaml
├── frontend
│   ├── Chart.yaml
│   ├── templates
│   │   ├── deployment-frontend.yaml
│   │   ├── ingress.yaml
│   │   └── service-frontend.yaml
│   └── values.yaml
├── backend
│   ├── Chart.yaml
│   ├── templates
│   │   ├── deployment-backend.yaml
│   │   ├── ingress.yaml
│   │   └── service-backend.yaml
│   └── values.yaml
├── values-dev.yaml
└── values-prod.yaml
```

# Approaching ArgoCD

## Creating applications

Create a new chart called `apps`. This directory will contain the resource definitions of your ArgoCD applications.
In this example we have two parts of our application, `frontend` and `backend`. So our chart would look like this:

<details>
<summary>apps/Chart.yaml</summary>

```yml
apiVersion: v2
name: example
description: Example application with backend and frontend

version: 0.0.0
type: application
```

</details>

<details>
<summary>apps/values.yaml</summary>

```yml
spec:
  environment:
  project: example
  destination:
    server: https://kubernetes.default.svc
    # Replace this with the namespace where you application should be deployed in
    namespace: example
  source:
    # Replace the repoURL to point to your repository
    repoURL: https://git.example.com/example/kubernetes.git
    # If you are currently working on a branch, provide your branch name here
    # Make sure to change this back to 'HEAD' if you like to always get the newest changes.
    targetRevision: HEAD
```

</details>

<details>
<summary>apps/templates/backend.yaml</summary>

```yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-backend
  namespace: argocd
spec:
  destination:
    namespace: {{ .Values.spec.destination.namespace }}
    server: {{ .Values.spec.destination.server }}
  project: {{ .Values.spec.project }}
  source:
    path: backend
    repoURL: {{ .Values.spec.source.repoURL }}
    targetRevision: {{ .Values.spec.source.targetRevision }}
    helm:
      releaseName: example-backend
      valueFiles:
      - {{ printf "../values-%s.yaml" .Values.spec.environment }}
```

</details>

<details>
<summary>apps/templates/frontend.yaml</summary>

```yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-frontend
  namespace: argocd
spec:
  destination:
    namespace: {{ .Values.spec.destination.namespace }}
    server: {{ .Values.spec.destination.server }}
  project: {{ .Values.spec.project }}
  source:
    path: frontend
    repoURL: {{ .Values.spec.source.repoURL }}
    targetRevision: {{ .Values.spec.source.targetRevision }}
    helm:
      releaseName: example-frontend
      valueFiles:
      - {{ printf "../values-%s.yaml" .Values.spec.environment }}
```

</details>

## Separate helm chart

If you have started this with one single helm chart you now have to separate your helm chart into charts for the different parts. Your result should look like the [backend and frontend of the target folder structure](#target-folder-structure).

> ⚠️ While refactoring be aware of that we will be using one single values file for the entire deployment so make sure to use explicit / unique paths for your values.

## Setup ArgoCD project

### Creation

1. Go to your ArgoCD instance and sign in.
2. Navigate to `Settings` > `Projects`
3. Click on `+ NEW PROJECT`
4. Enter a proper name for the project e.g. your application name
5. Add a description if you like to
6. Create your application by hitting the `CREATE` button

Now you should see the summary of your created project.

### Destinations

The destinations restrict the possible clusters and namespaces your project is able to deploy to.

In my case I want to be able to deploy the application to whereever I like to. Of course you can restrict the access here if you want to.

1. Scroll down to `DESTINATIONS`
2. Hit `EDIT`
3. Add a new destination by clicking on `ADD DESTINATION`
4. For now I will remove the asterisk in the `Name` column
5. Click on `SAVE`

### Cluster Resource Allow List

The information button right next to the title says: *Cluster-scoped K8s API Groups and Kinds which are permitted to be deployed*

To make sure that all our resources can be deployed in the cluster follow these steps:

1. Scroll down to `CLUSTER RESOURCE ALLOW LIST`
2. Click `EDIT`
3. Then add a resource to the `CLUSTER RESOURCE ALLOW LIST` by pressing `ADD RESOURCE`
4. Because I do not want to restrict the resources that can be deployed in any way I will leave it like this. Change this if you like to. 
5. Hit `SAVE`

## Add repository to ArgoCD

> ⚠️ In order to complete this step you have to make sure that your cluster is able to reach your Git instance.

1. Navigate to `Settings` > `Repositories`
2. Then press `+ CONNECT REPO`
3. Select your preferred connection method
4. Fill out the form

> ℹ️ Make sure to select the project you have created in the steps above.

While using `HTTPS` as a connection method, authentication will very likely be required. 
Using GitLab you can create an access token for the project and use it as the password following these steps:

1. Navigate to your repository
2. Go to `Settings` > `Access Tokens` in your navigation
3. Add a name and expiration date for your token
4. Select `Maintainer` as the role
5. Mark the checkbox for the `read_repository` scope
6. Click on `Create project access token`

At the top the access token is now displayed. Make sure to copy it and store it in a secure way.
In ArgoCD you can use the name of the repository as a username and the access token as the password.

> ⚠️ This is your only chance of copying / viewing this token.

After you have configured your repository make sure that after you have clicked `CONNECT` your repository should have the connection status `✅ Successful`.

## Setup ArgoCD root application (using UI)

1. In the `Applications` view click on `+ NEW APP`
2. As a name choose the name of your application or something like `example-root`
3. Select your created project from the previous steps
4. Scroll down to `SOURCE`
5. Select your repository URL
6. Change the revision if you are working on a branch
7. Set `apps` as the target path
8. Choose your destination including the cluster and namespace. The namespace should be the namespace of your entire deployment.
9. In the `HELM` section under `PARAMETERS` set the value of `spec.environments` to your target environment (either `dev` or `prod` depending on the name of your values files)
10. Click on `CREATE`

## Finish up

After you have successfully created the root application, take a look at the `APP DIFF`. Make sure that it includes your applications in the way you want them. Also check if the path of the values files are correct.

If everything is okay, apply the changes by clicking `SYNC` and `SYNCHRONIZE`. After your application reached the `Healthy` state, navigate to the other applications and check their states. Also sync each of your subapps.

> 🎉 Congrats, you have transformed your helmfile deployment to an ArgoCD deployment.
