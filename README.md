# Infrastructure Interview Test Solution

<h4 align="center">A production-minded Kubernetes deployment for the provided sample API, built as a practical MVP for the take-home assignment.</h4>

<p align="center">
  <a href="#overview">Overview</a> •
  <a href="#mvp-goal">MVP Goal</a> •
  <a href="#scope">Scope</a> •
  <a href="#target-architecture">Target Architecture</a> •
  <a href="#repository-structure">Repository Structure</a> •
  <a href="#mvp-deliverables">MVP Deliverables</a> •
  <a href="#local-workflow">Local Workflow</a> •
  <a href="#success-criteria">Success Criteria</a> •
  <a href="#roadmap">Roadmap</a>
</p>

---

## Overview

This repository contains my solution approach for an infrastructure take-home assignment.

The company provided a reference application in [`infrastructure-interview-test-main/`](./infrastructure-interview-test-main). That folder contains a small TypeScript + Express + TypeORM API backed by MySQL/MariaDB. It is intentionally simple and development-oriented, which makes it a good base for demonstrating containerization, Kubernetes deployment patterns, and production-minded operational decisions.

This root `README.md` defines the target MVP for the solution built around that reference application.

---

## MVP Goal

The MVP for this project is to:

* containerize the provided application
* deploy it to a local Kubernetes cluster
* expose it externally
* run it in a highly available configuration
* document the setup clearly enough that another engineer can reproduce it

The intent is to deliver a clean, credible baseline that satisfies the assignment requirements without overcomplicating the implementation. The focus is on good infrastructure judgment, clear tradeoffs, and a solution that another engineer could understand and run with minimal friction.

---

## Provided Reference Application

The provided application lives in [`infrastructure-interview-test-main/`](./infrastructure-interview-test-main).

It currently provides:

* `GET /posts`
* `GET /posts/:id`
* `POST /posts`

It currently assumes:

* a locally reachable MySQL/MariaDB database
* TypeORM configuration through `ormconfig.env`
* basic local development execution

This folder is kept as the original application reference and adaptation source. Preserving it makes the starting point explicit and keeps the infrastructure work easy to explain during review.

---

## Scope

The assignment requires the following baseline capabilities:

* a public container image for the application
* Kubernetes deployment manifests or equivalent deployment code
* external access to the application
* high availability across three availability zones

Because this MVP is designed to be tested locally, it cannot reproduce real cloud availability zones. It can still represent the intent by:

* running multiple replicas
* using anti-affinity or topology spread constraints
* documenting how the same deployment pattern maps to a real multi-AZ cluster

---

## Target Architecture

The target MVP architecture is:

1. The provided Node.js API is packaged into a Docker image.
2. A MySQL/MariaDB database runs as a separate Kubernetes workload for local testing.
3. The API runs as a Kubernetes `Deployment` with multiple replicas.
4. A Kubernetes `Service` exposes the API internally.
5. An `Ingress` or `LoadBalancer`-style exposure mechanism publishes the API externally in the local cluster.
6. Configuration is injected through environment variables and Kubernetes objects such as `ConfigMap` and `Secret`.
7. Health checks, resource requests, and basic scheduling constraints are added to make the deployment more production-like.

---

## MVP Characteristics

This MVP is designed around a few practical qualities:

* reproducible local setup
* minimal manual steps
* clean separation between application code and infrastructure code
* clear configuration management
* baseline security hygiene
* basic operational readiness
* concise documentation

For this submission, production-minded does not mean implementing every platform feature. It means choosing a small number of strong practices and applying them consistently.

---

## Repository Structure

Current repository structure:

```text
liferay_project/
├── README.md
└── infrastructure-interview-test-main/
    ├── README.md
    ├── package.json
    ├── ormconfig.env
    ├── tsconfig.json
    ├── yarn.lock
    └── src/
```

Target repository structure for the MVP:

```text
liferay_project/
├── README.md
├── infrastructure-interview-test-main/
├── docker/
├── k8s/
├── scripts/
└── docs/
```

Purpose of each top-level area:

* `infrastructure-interview-test-main/` - provided reference application and adapted app source
* `docker/` - container build assets if we separate them from the app directory
* `k8s/` - Kubernetes manifests, overlays, or Helm chart
* `scripts/` - helper scripts for build, deploy, and local testing
* `docs/` - architecture notes, deployment notes, and operational guidance

---

## MVP Deliverables

The minimum deliverables for this repository are:

* a Dockerfile for the application
* `.dockerignore`
* Kubernetes manifests for:
  * namespace
  * deployment
  * service
  * ingress or alternative external exposure
  * database deployment/stateful workload for local testing
  * config and secret objects
* readiness and liveness probes
* resource requests and limits
* replica configuration for high availability
* a short deployment script or command sequence
* updated documentation explaining how to build, deploy, test, and clean up

Optional but valuable additions:

* Helm packaging
* local cluster bootstrap script
* smoke tests
* CI workflow
* image publishing automation

---

## Local Workflow

The local development and validation workflow is expected to look like this:

1. Build the application image.
2. Load or pull the image into the local Kubernetes cluster.
3. Deploy database and application resources.
4. Wait for the workloads to become healthy.
5. Expose the service and test the API endpoints.
6. Verify replica distribution and recovery behavior.

The preferred local target for testing can be one of:

* `kind`
* `minikube`
* `k3d`

`kind` is the preferred MVP target because it is lightweight, scriptable, and widely used for local Kubernetes testing.

---

## Success Criteria

The MVP is successful when all of the following are true:

* the application image can be built successfully
* the app runs inside Kubernetes instead of only on a local machine
* the app can reach its database through Kubernetes networking
* the API is reachable from outside the cluster
* at least two application replicas can run concurrently
* the deployment includes health checks and sensible restart behavior
* another engineer can follow the README and reproduce the setup

---

## Constraints and Assumptions

Current assumptions:

* the provided app may need minor changes for container and Kubernetes compatibility
* the MVP is local-first, not cloud-first
* multi-AZ availability will be represented through Kubernetes scheduling strategy and documentation, since a local cluster does not provide real cloud AZs
* simplicity and clarity are better than implementing every optional feature

These assumptions may be refined as implementation details become clearer.

---

## Roadmap

Planned implementation order:

1. Understand and minimally harden the provided application.
2. Add containerization.
3. Make configuration environment-driven.
4. Add Kubernetes manifests for app and database.
5. Expose the service externally.
6. Add health checks, replicas, and scheduling constraints.
7. Document end-to-end usage.
8. Add optional polish such as Helm, CI, or automation scripts.

---

## Notes

This README is intentionally written as a delivery guide for the assignment rather than as a fully polished product README. Its purpose is to make the target state explicit, keep the implementation focused on a realistic MVP, and provide a clear narrative for technical review.
