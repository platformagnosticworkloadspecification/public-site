+++
title="PAWS - Platform-Agnostic Workload Specification"
weight=0
+++

# PAWS - Platform-Agnostic Workload Specification

## What is “PAWS”?

{{< hint info >}}
PAWS _is currently in alpha if you’re interested to join and provide feedback go [here.](https://forms.gle/z4MHQvdp4hYjv6eHA)_
{{< /hint >}}

PAWS is a specification for defining environment agnostic configuration for cloud based workloads. The PAWS configuration itself is saved alongside the code of the workload in source control. This configuration can then be combined with environment specific parameters to run the workload in the target environment. The same workload can then be run on completely different technology stacks without the developer needing to be an expert in any one of them.

For example, the same PAWS configuration can be used to generate a docker-compose file for local development, Kubernetes manifests for deployment to a shared development environment and to a serverless platform such as Google Cloud Run for integration tests.

## Why PAWS?

Cloud-native developers often struggle with config drift between environments. This is made even more complicated when the technology stack in each environment is different. Maybe you use Docker Compose for local development but Helm Charts to deploy to the Kubernetes based development environment? Not only do you have to figure out Docker Compose and Helm, but you need to keep them in sync!

PAWS is a single, easy to understand configuration for each workload. This configuration can then be used to generate the docker compose file or Kubernetes manifests that you need to get the workload up and running.

## How it works

PAWS is expressed in YAML and sits alongside the code for the workload in source control. It describes the environment agnostic configuration needed to run the workload. This can include:

- Templated configuration in the form of Environment Variables or Files
- Dependencies on resources such as Databases, Volumes or DNS names
- Dependencies on other workloads and services
- Routes that the workload should respond to
- Container overrides such as for CMD and ARGS

A tool known as a PAWS Implementation is then run against the PAWS configuration. It is up to the implementation how it deals with environment specific parameters. The result of the Implementation is either to deploy the workload in some environment (e.g. deploying to a K8s cluster or Google Cloud Run) or output files which can be used to run the workload in an environment e.g. a `docker-compose.yaml` file for local development or Kubernetes manifests for a K8s deployment)

### Example

Consider a product catalogue service that manages products. It’s API is available under the `/products` and it stores its products in a PostgreSQL database.

The PAWS configuration might look as follows:

```bash
apiVersion: spec.paws.sh/v1alpha1
kind: Workload
metadata:
  name: products-catalog
spec:
  resources:
    db:
      type: postgres
    api-dns:
      type: dns
  container:
    variables:
      CONNECTION_STRING: postgresql://${resources.db.user}:${resources.db.password}@${resources.db.host}:${resources.db.port}/${resources.db.name}
  routes:
    resources.api-dns.host:
	    /products:
	      match: prefix
```

For the local environment, the docker-compose PAWS implementation might be used. This might produce the following docker-compose file:

```bash
products-catalog:
  build: .

  depends_on:
    - products-catalog--db

  environment:
    CONNECTION_STRING: postgresql://postgres:p455w0rd@products-catalog--db:5432/postgres

products-catalog--db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: p455w0rd
```

While for the K8s implementation might create Kubernetes Deployment, Service, Ingress, ConfigMap and Secret manifests.

{{< figure link="/_assets/images/paws-overview.png" src="/_assets/images/paws-overview.png" caption="PAWS Configuration and Implementation" alt="paws-overview.png" >}}

## Development Status

PAWS is currently in alpha. PAWS is developed by an open consortium around Lee Ditiangkin, Guy Sayer, Chris Stephenson, Kaspar von Grünberg, and many others.
