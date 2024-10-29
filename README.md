# Spring PetClinic sample application for Kubernetes

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

The famous [Spring PetClinic sample application](https://github.com/spring-projects/spring-petclinic)
is now available as a Kubernetes native application leveraging microservices.

All dependencies / code which are not required to run this application with Kubernetes have been removed:
for example you don't need to use service discovery with 
[Spring Cloud Netflix](https://spring.io/projects/spring-cloud-netflix) since the Kubernetes platform
already provides one such implementation.

Moreover, container images for the Spring PetClinic are now built using [Cloud Native Buildpacks](https://buildpacks.io).
You will not find any `Dockerfile` in this repository: the `pack` CLI is used to build secure and
optimized container images for you.

## Building this application

You need a JDK 8+ to build this application:

```bash
$ ./mvnw clean package
```

Pre-built container images are available, so that you can start deploying this app to your favorite Kubernetes cluster. In case you'd like to build your own images, please follow these instructions.

To use the pre-built containers set `REPOSITORY_PREFIX=alexandreroman` and go to the [K8S deployment section](#deploying-this-application-to-kubernetes)

[Read this guide](https://buildpacks.io/docs/install-pack/) to deploy the `pack` CLI to your workstation.

Many buildpack implementations are available: for best results, use [Paketo buildpacks](https://paketo.io):

```bash
$ pack set-default-builder gcr.io/paketo-buildpacks/builder:base
```

You're ready to build container images with no Dockerfile!

Use the provided `Makefile` to build container images:

Set `DOCKER_PREFIX` to your docker hub username

```bash
$ make all DOCKER_PREFIX=myrepo
```

## Running this application locally

There is no need to run an Eureka server or anything else: this application is ready to run on your workstation.

Start the gateway:

```bash
$ java -jar spring-petclinic-api-gateway/target/spring-petclinic-api-gateway-VERSION.jar
```

Start the `customers` service:

```bash
java -jar spring-petclinic-customers-service/target/spring-petclinic-customers-service-VERSION.jar
```

Start the `vets` service:

```bash
$ java -jar spring-petclinic-vets-service/target/spring-petclinic-vets-service-VERSION.jar
```

Start the `visits` service:

```bash
$ java -jar spring-petclinic-visits-service/target/spring-petclinic-visits-service-VERSION.jar
```

Using your browser, go to http://localhost:8080 to access the application.

## Deploying this application to Kubernetes

This application relies on a MySQL database to persist data: you first need to deploy a MySQL instance.

Run these commands to create a MySQL database:

```bash
$ kubectl create ns spring-petclinic
$ helm upgrade pdb bitnami/mysql -n spring-petclinic -f k8s/services/mysql/values.yml --version 9.1.4 --install
```

You're almost there!

Deploy the application to your cluster:

Our deployment YAMLs have a placeholder called `REPOSITORY_PREFIX` so we'll be able to deploy the images from any Docker registry. Sadly, Kubernetes doesn't support environment variables in the YAML descriptors. We have a small script to do it for us and run our deployments:

Set `REPOSITORY_PREFIX` to your docker hub username.

OR set to `REPOSITORY_PREFIX=alexandreroman` to use pre-built containers

```bash
./scripts/deployToKubernetes.sh
```

The application is not publicly accessible: you need to create a Kubernetes service. Depending on your cluster configuration, you may have to use an `Ingress` route or a `LoadBalancer` to expose your application.

Run this command to use a Kubernetes managed load balancer:
```bash
$ kubectl apply -f k8s/loadbalancer
```

Run `minikube tunnel` to create a tunnel to the application load balancer.

Running the following to see the external IP
```bash
kubectl get services -n spring-petclinic 
NAME                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
api-gateway          ClusterIP      10.105.118.131   <none>        8080/TCP       13m
app-lb               LoadBalancer   10.102.105.180   127.0.0.1     8081:32456/TCP   2m53s
customers            ClusterIP      10.101.9.241     <none>        8080/TCP       13m
pdb-mysql            ClusterIP      10.96.197.73     <none>        3306/TCP       9m26s
pdb-mysql-headless   ClusterIP      None             <none>        3306/TCP       9m26s
vets                 ClusterIP      10.99.197.155    <none>        8080/TCP       13m
visits               ClusterIP      10.100.130.182   <none>        8080/TCP       13m
```

Go to http://localhost:8081 to access the application

Congratulations: you're done!

![Spring Petclinic Microservices screenshot](docs/application-screenshot.png)
