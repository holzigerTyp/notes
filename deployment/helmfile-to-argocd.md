# Transform deployment using helmfile to ArgoCD

> How can I transform my helmfile deployment into an ArgoCD deployment? What changes are required in order to deploy an application using ArgoCD? What are best practises?

# Table of Contents


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

