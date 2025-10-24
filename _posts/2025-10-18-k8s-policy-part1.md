---
title: "Kubernetes Policy-as-Code using Kyverno and OPA Gatekeeper (Part 1)"
date: 2025-10-18 10:00:00 +0000
categories: [DevSecOps, Cloud Security]
tags: [cloud security, kubernetes security, kubernetes policies, Pod security standards, kyverno, devsecosps, cloud native, policy-as-code, container-security]
layout: post
publish: true
comments: true # <-- THIS LINE ENABLES GISCUS COMMENTS
image:
  path: /assets/headers/k8s-security.jpg
---

This project involves using a deliberately vulnerable bank application, **VULN-BANK** by Commando-X ([GitHub](https://github.com/Commando-X)) and securing its Kubernetes deployment using Kyverno and Open Policy Agent (OPA) Gatekeeper as Policy-as-Code tools. The goal is to deploy the VULN-BANK application and enforce organisational and compliance-aligned security policies directly within the cluster to ensure all application containers run as non-root users and automatically inject memory and CPU limits if not defined and prevent applications from running in privileged mode due to compliance with regulatory frameworks (e.g., PCI DSS or ISO 27001).

## Two Key Policy Use-Cases Implemented

- **Kyverno**: Enforce that all containers run as non-root users and automatically inject CPU and memory limits if not defined.
- **OPA Gatekeeper**: Prevent Pods from using `hostNetwork: true` or running in privileged mode, protecting the host and meeting compliance standards like PCI DSS and ISO 27001.

This guide is broken down into three phases:

1. Setting up the Environment
2. Deploying the Insecure VULN-BANK App in Kubernetes
3. Enforcing a Security Policy

## Setting up the Environment

### Prerequisites

Install these tools:

- **Git**
- **Docker / Docker Desktop**
- **Kubectl**: command-line tool for interacting with a Kubernetes cluster
- **kind** (Kubernetes in Docker, for a local Kubernetes cluster)
- **Helm**: package manager for Kubernetes. Think of it like apt or brew, but for Kubernetes applications. It bundles all the necessary YAML files and configurations into a package called a chart

Getting started, spin up the app following the documentation ([GitHub](https://github.com/Commando-X/vuln-bank)) and run the Docker Container locally to make sure the app is working.

![App running locally](/assets/img/kyverno/1.png)
![App running locally2](/assets/img/kyverno/2.png)
![App running locally3](/assets/img/kyverno/3.png)

**N.B**: Stop the docker Container to avoid too much computational workload on the system as we are only checking the app runs.

```bash
docker ps
docker stop <container_id>
```

### Create a Local Kubernetes Cluster with kind and Prepare the Image

We will use kind to create a local Kubernetes cluster inside Docker containers so we can run everything locally without cloud costs.

#### 1) Create a kind Cluster

In project root folder, create `kind-config.yaml`, copy and paste the code below into it:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 3000
    hostPort: 3000
    protocol: TCP
```

Run the commands below in terminal:

```bash
kind create cluster --name vulnbank-demo --config kind-config.yaml
kubectl cluster-info --context kind-vulnbank-demo
```

This creates a cluster where port 3000 in the cluster is reachable on the host at localhost:3000.

![Kind cluster created](/assets/img/kyverno/4.png)

#### 2) Build the VulnBank Image and Load it into kind

By default, kind cannot pull local Docker images from the host; either push to a registry or load with kind load. From project root, run the cmd below:

```bash
docker build -t vuln-bank:local .
```

![Docker build](/assets/img/kyverno/5.png)

```bash
# load the image into kind
kind load docker-image vuln-bank:local --name vulnbank-demo

# check the status of kind clusters
kind get clusters

# check the status of Kubernetes nodes
kubectl get nodes

# run a temporary container for testing and attach an interactive bash shell to it
kubectl run test-vulnbank --image=vuln-bank:local --rm -it --bash
```

![Kind load image](/assets/img/kyverno/6.png)

Each KIND cluster node is a Docker Container. To check images inside it:

```bash
docker ps
```

Then list images inside vulnbank control plane:

```bash
docker exec -it vulnbank-demo-control-plane crictl images
```

![Images in kind](/assets/img/kyverno/7.png)

## Deploying the Insecure VULN-BANK App

Using Kubernetes, we will deploy the application in its default vulnerable configuration, which often means running the Container processes as root.

### Create Kubernetes Manifests for VulnBank

Create a directory `k8s/` in project and add the following files:

1. `k8s/deployment.yaml`
2. `k8s/service.yaml`
3. `k8s/db.yaml`

The files content can be found here: [GitHub Repository](https://github.com/zainaboladiti/cloud-security-k8s-policy)

This simple configuration deploys the application with minimal settings, and importantly, it does not explicitly set a non-root user, which defaults to the insecure root user based on the Container image's configuration.

Apply the manifests created:

```bash
kubectl apply -f k8s/deployment.yaml 
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/db.yaml
```

Check and confirm the Pods are running:

```bash
kubectl get Pods -l app=vuln-bank -o wide
```

You should see a Pod with the name pattern `vulnbank-XXXXX` in the Running status. At this point, the VULN-BANK application is running in an insecure configuration (as root).

![Pods running](/assets/img/kyverno/8.png)

Forward the port to match port in `kind-config.yaml`:

```bash
kubectl port-forward svc/vuln-bank-svc 3000:3000
```

![Port forward](/assets/img/kyverno/9.png)

Then open [http://localhost:3000](http://localhost:3000) to view the VulnBank app.

![VulnBank app](/assets/img/kyverno/10.png)

## Security Policy Enforcement

Now, we will use Kyverno to enforce a rule that prevents the Pod from running as the root user (UID 0), aligning with the Kubernetes security best practice of "Run as Non-Root."

**Use Case**: Enforce Non-Root Containers and Add Resource Limits Automatically.

**Scenario**: As a security professional, we want to ensure all application containers deployed by the development team in the organisation run as non-root users and automatically inject memory and CPU limits.

### Kyverno – Installation, Policies Enforcement and Testing

Kyverno is the policy engine for Kubernetes-native that allows validation, mutation, generation of resources and defining security rules using Kubernetes YAML.

There are two common ways for installing Kyverno: Helm (recommended for production) or raw YAML for quick demo. For the purpose of this documentation, we will install via Helm.

#### Step 1: Install Kyverno with Helm

```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
```

Next, update the Helm repositories to make sure chart information is up to date:

```bash
helm repo update
```

![Helm repo update](/assets/img/kyverno/11.png)

```bash
helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace
```

![Kyverno install](/assets/img/kyverno/12.png)

```bash
# optional: install kyverno-policies (prebuilt policy sets)
helm install kyverno-policies kyverno/kyverno-policies --namespace kyverno
```

![Kyverno policies](/assets/img/kyverno/13.png)

**Verify Kyverno Installation**

To watch the status of the Pods:

```bash
kubectl get Pods -n kyverno -w
```

![Kyverno Pods](/assets/img/kyverno/14.png)

#### Step 2: Create Kyverno Policies

##### 1. Policy to Enforce Non-Root Containers

This is a validation policy to deny any resource (like a Pod or Deployment) that tries to run a Container as the root user (`runAsUser: 0`) or does not explicitly set `runAsNonRoot: true`. In project folder, create `k8s-policies/kyverno/require-non-root.yaml`:

The file content can be found here: [GitHub Repository](https://github.com/zainaboladiti/cloud-security-k8s-policy)

`background: true` used so the policy will scan and report on all pre-existing resources in the cluster that violate the rule. It runs as a background process after we have created the policy.

> **Note**: Kyverno provides many ready-made examples and best-practice policies (require non-root, require limits, etc.). Use `validationFailureAction: audit` first if you want to observe violations without blocking.
{: .prompt-tip }

##### 2. Policy to Inject Default Resource Limits

This is a mutation policy that automatically adds requests and limits if they are missing. In project folder, create `k8s-policies/kyverno/inject-default-resources.yaml`:

The file content can be found here: [GitHub Repository](https://github.com/zainaboladiti/cloud-security-k8s-policy)

> **Note**: `patchStrategicMerge` is Kyverno's supported way to inject fields; it will not overwrite existing limits but adds missing fields.
{: .prompt-tip }

#### Step 3: Apply the Kyverno Policies to the Cluster

```bash
kubectl apply -f k8s-policies/kyverno/
```

![Apply policies](/assets/img/kyverno/15.png)

#### Step 4: Test Kyverno Policy Enforcement with VulnBank Manifest

##### 1. Validation

Now we will re-deploy the application by deleting the previous deployment. Kyverno will check `runAsNonRoot` with `validationFailureAction: enforce`, deployment will be denied.

```bash
kubectl delete deployment/vuln-bank
kubectl apply -f k8s/deployment.yaml
```

![Policy denial](/assets/img/kyverno/16.png)

Since the policy is now in place, the Kubernetes API server checked the policy before applying the manifest and we're presented with an error message above, confirming the policy is blocking the insecure configuration. Try accessing the web app url, it will no longer be accessible.

![App not accessible](/assets/img/kyverno/17.png)

##### 2. Mutation

We will check the Pod for resource limits and observe injection. The Container in the Pod now has `resources.requests` and `resources.limits` injected, per the mutation policy.

To view the Pod, run the cmd below:

```bash
kubectl get Pod -l app=mutate-test -o yaml | sed -n '1,200p'
```

Inspect the 'resources' block inside the Container: Kyverno should have injected the default requests/limits.

![Resources injected](/assets/img/kyverno/18.png)

## Why These Policies Matter

- Running as root inside a Container significantly increases the blast radius of any compromise, if an attacker escapes the Container or exploits a vulnerability, they could gain host-level access.
- Enforcing non-root execution ensures applications run with the least privilege principle, one of the cornerstones of Container security and compliance frameworks like CIS Benchmarks and ISO 27001.
- Developers could forget to define CPU and memory limits, which can lead to resource exhaustion or denial of service in multi-tenant clusters. By automatically injecting safe defaults, we ensure fair resource allocation, stability and predictable scheduling.

Together, these Kyverno policies not only protect the cluster from insecure configurations but also reduce human error by automatically enforcing standards during deployment.

---

Continue to [Part 2] where we implement OPA Gatekeeper policies and CI/CD integration.

