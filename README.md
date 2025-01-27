<!--
    Copyright (c) 2021-2022 Robert Bosch Manufacturing Solutions GmbH
    Copyright (c) 2021-2022 Contributors to the Eclipse Foundation

    See the NOTICE file(s) distributed with this work for additional 
    information regarding copyright ownership.
    
    This program and the accompanying materials are made available under the
    terms of the Apache License, Version 2.0 which is available at
    https://www.apache.org/licenses/LICENSE-2.0.
     
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
    WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
    License for the specific language governing permissions and limitations
    under the License.
    
    SPDX-License-Identifier: Apache-2.0
-->

# Semantic Hub
The Semantic Hub is a logical and architectural component of Tractus-X.
The source code under this folder contains reference implementations of the SLDT Semantic Hub

## Build Packages

Run `mvn install` to run unit tests, build and install the package.

## Run Package Locally
To check whether the build was successful, you can start the resulting JAR file from the build process by running `java -jar target/semantic-hub-backend-{current-version}.jar`.

## Build Docker
Run `docker build -t semantic-hub .`

In case you want to publish your image into a remote container registry, apply the tag accordingly and `docker push` the image.

## Deploy using Helm and K8s
If you have a running Kubernetes cluster available, you can deploy the Semantic Hub using our Helm Chart, which is located under `./charts/semantic-hub`.
In case you don't have a running cluster, you can set up one by yourself locally, using [minikube](https://minikube.sigs.k8s.io/docs/start/).
In the following, we will use a minikube cluster for reference.

Before deploying the Semantic Hub, enable a few add-ons in your minikube cluster by running the following commands:

`minikube addons enable storage-provisioner`

`minikube addons enable default-storageclass`

`minikube addons enable ingress`

If you want to use the in-memory triple store that is not persistent (useful for local deployments) set `embeddedTripleStore: true`. 

In order to deploy the helm chart, first create a new namespace "semantics": `kubectl create namespace semantics`.

Then run `helm install hub -n semantics ./charts/semantic-hub`. This will set up a new helm deployment in the semantics namespace. By default, the deployment contains the Semantic Hub instance itself, and a Fuseki Triplestore.

Check that the two containers are running by calling `kubectl get pod -n semantics`.

To access the Semantic Hub API from the host, you need to configure the `Ingress` resource.
By default, the Semantic Hub includes an `Ingress` that exposes the API on https://minikube/semantics/hub

For that to work, you need to append `/etc/hosts` by running `echo "$(minikube ip) minikube" | sudo tee -a /etc/hosts`.

For automated certificate generation, use and configure [cert-manager](https://cert-manager.io/).
By default, authentication is deactivated, please adjust `hub.authentication` if needed

## Parameters
The Helm Chart can be configured using the following parameters (incomplete list). For a full overview, please see the [values.yaml](backend/deployment/semantic-hub/values.yaml).

### Semantic Hub
| Parameter       | Description | Default value       |
| ---             | ---         | ---                 |
| `hub.image`     | The registry and image of the Semantic Hub   | `semantic-hub:latest` |
| `hub.embeddedTripleStore`     | Configures whether the Semantic Hub uses its non-persistent embedded in-memory Triplestore. If set to `true`, no separate Fuseki instance is deployed. | `false` |
| `hub.host`     | This value is used by the `Ingress` object (if enabled) to route traffic.   | `minikube` |
| `hub.authentication`     | Enables OAuth2 based authentication/authorization.   | `false` |
| `hub.idpIssuerUri`     | The issuer URI of the OAuth2 identity provider.   | `http://localhost:8080/auth/realms/catenax` |
| `hub.graphdbBaseUrl`     | The URL of an external GraphDB. Ignored if `graphdb.enabled` is set to `false`   | `http://graphdb:3030` |
| `hub.ingress.enabled`     | Configures if an `Ingress` resource is created.   | `true` |
| `hub.ingress.tls`     | Configures whether the `Ingress` should include TLS configuration. In that case, a separate `Secret` (as defined by `hub.ingress.tlsSecretName`) needs to be provided manually or by using [cert-manager](https://cert-manager.io/)   | `true` |
| `hub.ingress.tlsSecretName`     | The `Secret` name that contains a `tls.crt` and `tls.key` entry. Subject Alternative Name must match the `hub.host`    | `hub-certificate-secret` |
| `hub.ingress.urlPrefix`     | The url prefix that is used by the `Ingress` resource to route traffic  | `/semantics/hub` |
| `hub.ingress.className`     | The `Ingress` class name   | `nginx` |
| `hub.ingress.annotations`     | Annotations to further configure the `Ingress` resource, e.g. for using with `cert-manager`.  |  |

### GraphDB
| Parameter       | Description | Default value       |
| ---             | ---         | ---                 |
| `graphdb.enabled`     | Configures, whether a separate Fuseki Triplestore should be deployed in the cluster.   | `true` |
| `graphdb.storageClassName`     | Defines the storage class name of the `PersistentVolumeClaim` that is used to persist the GraphDB data.  | `standard` |
| `graphdb.storageSize`     | Size of the `PersistentVolumeClaim`  | `50Gi` |

## Notice for Docker image

This application provides container images for demonstration purposes.

GitHub Container Registry (GHCR): https://github.com/catenax-ng/product-semantics/pkgs/container/sldt-semantic-hub

Eclipse Tractus-X product(s) installed within the image:

- GitHub: https://github.com/eclipse-tractusx/sldt-semantic-hub
- Project home: https://projects.eclipse.org/projects/automotive.tractusx
- Dockerfile: https://github.com/eclipse-tractusx/sldt-semantic-hub/blob/main/backend/Dockerfile
- Project license: [Apache License, Version 2.0](https://github.com/eclipse-tractusx/sldt-semantic-hub/blob/main/LICENSE)

**Used base image**
- [eclipse-temurin:17-jre-alpine](https://github.com/adoptium/containers)
- Official Eclipse Temurin DockerHub page: https://hub.docker.com/_/eclipse-temurin  
- Eclipse Temurin Project: https://projects.eclipse.org/projects/adoptium.temurin  
- Additional information about the Eclipse Temurin images: https://github.com/docker-library/repo-info/tree/master/repos/eclipse-temurin

As with all Docker images, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.