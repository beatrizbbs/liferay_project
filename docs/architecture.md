# Architecture

## Overview

This project packages the provided sample API into a containerized application and deploys it to Kubernetes using Helm.

The reference application is a small Node.js service built with Express and TypeORM, backed by MySQL/MariaDB. The application itself is intentionally simple. The main purpose of this solution is to demonstrate practical infrastructure and platform engineering decisions around packaging, configuration, deployment, external exposure, and operational readiness.

The MVP is designed for local Kubernetes execution, while still aligning with the expectations of a more production-like deployment model.

---

## Architecture Goals

This architecture is designed around the following goals:

1. **Keep the application easy to deploy**
   - The application should run with a small number of clear commands
   - Deployment artifacts should be organized and reproducible

2. **Use Kubernetes-native deployment patterns**
   - Application runtime managed by a `Deployment`
   - Stable internal networking through a `Service`
   - External traffic handled through `Ingress`
   - Configuration managed through `ConfigMap` and `Secret`

3. **Support high availability expectations**
   - Multiple application replicas
   - Health checks for restart and routing decisions
   - Scheduling guidance that reflects multi-failure-domain thinking


---

## High-Level Architecture

At a high level, the system is expected to work like this:

- A client sends an HTTP request to the application
- Traffic enters the local Kubernetes cluster through an Ingress controller
- The Ingress routes the request to a Kubernetes `Service`
- The `Service` forwards traffic to one of the healthy application Pods
- The application Pod processes the request and connects to MariaDB through cluster networking
- Helm manages the Kubernetes resources used to install and configure the stack

This keeps network entry, service discovery, runtime scheduling, and configuration management separated into well-defined Kubernetes concerns.

---

## Main Components

### API Client

The client represents any consumer of the sample API.

This could be:

- a browser
- `curl`
- Postman
- another service

Clients do not connect directly to Pods. They access the application through the external entry point configured for the cluster.

---

### Local Kubernetes Cluster

The local Kubernetes cluster is the runtime environment for the MVP.

This project is intended to be tested on a local distribution such as:

- `kind`
- `minikube`
- `k3d`

For this MVP, a local cluster provides a practical way to validate containerization, workload definitions, service exposure, and rollout behavior without requiring a cloud account.

---

### Helm Chart

Helm is the packaging and deployment mechanism for the Kubernetes resources in this solution.

Its role is to:

- template Kubernetes manifests
- centralize configuration values
- support repeatable installs and upgrades
- make the deployment easier to reason about and explain

**Why Helm was chosen**

- It was explicitly preferred for the assignment
- It provides a cleaner delivery artifact than manually managing many raw manifests
- It keeps the deployment configurable without duplicating YAML

---

### Ingress Controller

The Ingress controller is responsible for implementing external HTTP routing inside the cluster.

It receives incoming traffic and applies the rules defined by the application `Ingress` resource.

In a local Kubernetes environment, this is the component that turns cluster-internal services into something reachable from outside the cluster.

---

### Kubernetes Ingress

The `Ingress` resource defines how external HTTP traffic reaches the application.

Its responsibilities include:

- mapping a host or path to the application service
- acting as the public entry point for the application
- keeping request routing separate from application runtime definitions

This makes external exposure explicit and keeps it decoupled from the Pod-level deployment.

---

### Kubernetes Service

The Kubernetes `Service` provides stable internal networking for the application Pods.

Its role is to:

- give the application a stable DNS name inside the cluster
- load balance traffic across healthy Pods
- decouple clients from individual Pod IPs

This is an important abstraction because Pods are ephemeral and can be rescheduled at any time.

---

### Application Deployment

The application `Deployment` manages the desired state of the API Pods.

Its responsibilities include:

- defining the container image to run
- controlling the replica count
- restarting failed Pods
- rolling out updated versions
- applying resource settings and health checks

This is the main runtime controller for the API workload.

---

### Application Pods

The application Pods run the containerized Node.js API.

Their responsibilities include:

- receiving requests routed through the Service
- executing the `/posts` API endpoints
- connecting to the database
- reading runtime configuration from environment variables
- returning responses to clients

This is the business logic layer of the system.

---

### MariaDB Workload

MariaDB provides the relational database required by the reference application.

For the local MVP, it is deployed as a Kubernetes workload within the cluster so that the application can be tested end-to-end without depending on a separately managed external database.

Its responsibilities include:

- storing application data
- serving SQL queries from the application
- providing a stable in-cluster dependency for local validation

For a local MVP, this keeps the setup self-contained. In a production environment, this database would likely be replaced by a managed database service or a more robust persistence strategy.

---

### ConfigMap

The `ConfigMap` stores non-sensitive runtime configuration.

Typical values may include:

- application port settings
- database host name
- database name
- environment-level non-secret settings

This keeps configuration separate from the container image and makes the deployment easier to adapt across environments.

---

### Secret

The `Secret` stores sensitive runtime values used by the application and database.

Typical values may include:

- database username
- database password
- root password for local MariaDB setup

Using Kubernetes `Secret` objects is a baseline improvement over hardcoding sensitive values directly in source files or deployment manifests.

---

## Request Flow

The main request flow is:

1. A client sends a request to the application endpoint.
2. The request enters the cluster through the Ingress controller.
3. The Ingress routes the request to the application `Service`.
4. The `Service` forwards traffic to a healthy application Pod.
5. The application processes the request.
6. If data is required, the application connects to MariaDB using the internal Kubernetes service name.
7. The response is returned back through the same path to the client.

This flow ensures:

- external traffic enters through a controlled entry point
- internal Pod addresses remain abstracted behind a Service
- multiple replicas can serve requests transparently

---

## Deployment Flow

The deployment flow for this architecture is:

1. The application source is prepared for container execution.
2. A Docker image is built for the Node.js API.
3. The image is made available to the local cluster or a public registry.
4. Helm installs or upgrades the release.
5. Kubernetes creates or updates the database and application workloads.
6. Readiness and liveness checks help determine when Pods can receive traffic and when they should be restarted.
7. The Ingress exposes the application for local testing.

This creates a clear separation between:

- application source code
- container packaging
- deployment configuration
- runtime orchestration

---

## Configuration and Secret Flow

The application should be configured through Kubernetes-native objects rather than local-only files.

Typical configuration flow:

1. Helm renders the deployment using environment-specific values.
2. Kubernetes creates the required `ConfigMap` and `Secret` objects.
3. The application Pod starts and receives its runtime configuration through environment variables.
4. The application uses those values to connect to MariaDB and run normally.

**Why this matters**

- it removes environment assumptions from the container image
- it avoids depending on a local-only setup like `localhost`
- it makes the deployment more portable and production-minded

---

## High Availability Approach

The assignment expects the application to be deployable in a highly available setup across three availability zones.

Because this implementation is local-first, it cannot create real cloud availability zones. Instead, the MVP represents that intent through Kubernetes scheduling and replica strategy:

- multiple application replicas
- readiness and liveness probes
- rolling update behavior
- anti-affinity or topology spread constraints where practical
- documentation explaining how the same pattern maps to a real multi-AZ cluster

In a cloud-hosted Kubernetes environment, the same design would place nodes across multiple availability zones and use Kubernetes scheduling features to reduce concentration of replicas in a single failure domain.

---

## Why These Design Choices Were Made

### Why Kubernetes?

Kubernetes is the core requirement of the assignment, so the solution is designed around native Kubernetes concepts instead of using a simpler container runtime alone.

This demonstrates:

- workload orchestration
- service discovery
- external exposure
- health-aware deployments
- scalable runtime patterns

### Why Helm instead of only raw manifests?

Helm keeps the deployment cleaner and easier to operate once there are multiple related resources.

It helps by:

- grouping resources into one installable unit
- reducing duplicated YAML
- centralizing tunable values
- making upgrades and overrides easier

### Why MariaDB inside the cluster for the MVP?

For a local take-home submission, running MariaDB inside the cluster keeps the environment self-contained and reproducible.

This avoids requiring:

- an externally managed database
- additional local setup outside Kubernetes
- environment-specific host assumptions

---

## Security Considerations

This architecture includes several baseline security practices:

- configuration separated from application code
- sensitive values stored in Kubernetes `Secret` objects
- external access routed through Ingress rather than directly to Pods
- multiple layers of separation between client traffic and application runtime

Additional improvements that could be added later include:

- network policies
- stricter container security contexts
- image scanning
- secret externalization through a dedicated secret manager
- TLS termination for local or remote environments

---

## Reliability and Scaling

The MVP is designed with baseline resilience in mind.

### Reliability

- multiple application replicas reduce the chance of a single Pod outage taking down the service
- Kubernetes can restart failed containers automatically
- readiness checks help keep unhealthy Pods out of the traffic path

### Scaling

- the application can scale horizontally by increasing the replica count
- the Service can distribute traffic across running Pods
- the deployment structure can be adapted later for autoscaling

Possible future improvements include:

- Horizontal Pod Autoscaler support
- PodDisruptionBudget configuration
- stronger probe tuning
- dedicated production-grade database backing services

---

## Future Improvements

Possible next improvements for this architecture include:

- CI validation for the application image and Helm chart
- automated local cluster bootstrap
- smoke tests after deployment
- separate values files for different environments
- ingress TLS support
- metrics, dashboards, and alerting
- stronger scheduling rules for failure-domain distribution

---

## Summary

This architecture uses a practical Kubernetes deployment model built around Helm, Ingress, Services, Deployments, application Pods, and an in-cluster MariaDB dependency for local testing. The design is intentionally simple enough to be reproducible locally while still reflecting the core ideas expected from a production-minded container platform submission.
