# Deployment

- [Containers](#containers)
- [CI/CD](#ci-cd)
- [Capacity Management](#capacity-management)

## Containers
- Allow you to run different applications on different systems
- Allow us to run our code in a consistent environment during dev and prod
- **Docker Image**: a pre-built artifact which contains all necessary dependencies for running an application. The package of everything.
- **Docker Container**: a running instance of a docker image


## CI/CD
### Continuous integration
- Practice of frequently integrating all code changes into a single, authoritative branch, from which production environments are deployed.
- **Tingle**: centrally managed CI/CD system. Builds, Tests, and Deploys.
  - `build-info.yaml`: contains constructions for Tingle to build, test and deploy a system
  - Templates for common services
  - Action containers: the way Tingle runs each build step. A docker image with common languages and tools (e.g. maven, sbt, etc.) installed in it.

### Contiunous Delivery
- A discipline where you build software in such a way tht the software can be released to production at any time. Easy to rollback for example.
- **Tugboat**: performs Production and Canary Deployments.
  - Rollbacks to previous deployed versions
  - Interfaces with multiple backends
  - `deployment-info.yaml`: contains instructions for Tugboat to deploy
  - Canary: instances of the new version of your application. Approve before rolling the new version to other old instances. Limit the blast radius of the new versions we make.

## Capacity Management
### Kubernetes
- Running docker containers more declaratively
- Make sure running the containers on a suitable host
- One extra extraction layer on top of Docker
- Open-source
- Automated deployments, auto-scaling, easier maintenance

### GKE
_Google Kubernetes Engine_

<img src="https://cloud.google.com/static/kubernetes-engine/images/cluster-architecture.svg" height="500px">

- Use Kubernetes in Google Infrastructure
- Cluster: multiple machines grouped together. In GCP, are created at zone level
- Node: worker machines. In GCP, nodes are Google Compute Engines
- Pod: represents a unit of deployment with one or more containers. Like a VM.
- Namespace: a virtual cluster of things you care about. A namespace belongs to a particular team. Makes it easier to find things.
<img src="https://cloud.google.com/static/kubernetes-engine/images/workload-identity-sameness.svg" height="500px">

- Kubernetes configuration files
  - deployment.yaml: Creates Pods and works to ensure that the necessary are available on the node when creating a pod
    - Readiness Probe: Health check URL. For kubernetes to know if your application is working or not. If get a health response.
      ```yaml
      readinessProbe:
        exec:
          command:
            - grpc_health_probe
            - -addr
            - localhost:5990
      ```
    - Resource Requests: for capacity management. CPU and memory.
      - resources.requests: amount to be allocated
      - resources.limits: if there's more capacities available, limit the container to use them. Shouldn't be too much higher than the requests.
      ```yaml
      resources:
        requests:
          cpu: 200m
          memory: 4G
        limits:
          cpu: 2
          memory: 4Gi
      ```
    - Docker Container: the name of the container that needs to be deployed. Usually comes from an env var.
      ```yaml
      - containers
        ...
        image: $DEPLOYMENT_IMAGE
      ```
  - service.yaml: responsible enabling network access to a set of pods. Notify the network that this deployment is a service i want others to be able to use.
  - horizontalpodautoscaler.yaml
    - Horizontal auto-scaling: increase the number of instances
    - Vertial auto-scaling: increase the size of instances
    - Horizontal auto-scaling: Auto scaling the number of replicas of your Pod based on resource utilization or custom metrics
      ```yaml
      maxReplicas: 10
      minReplicas: 2
      ```
      - For example: auto-scale when my CPU reaches 80% of the capacity
      ```yaml
      metrics:
      - type: Resource
        resource:
          name: cpu
          targetAverageUtilization: 80
      ```
