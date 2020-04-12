
# Helidon Quickstart MP Example

This example implements a simple Hello World REST service using MicroProfile

## Prerequisites

1. Maven 3.5 or newer
2. Java SE 8 or newer
3. Docker 17 or newer (if you want to build and run docker images)
4. Kubernetes minikube v0.24 or newer (if you want to deploy to Kubernetes)
   or access to a Kubernetes 1.7.4 or newer cluster
5. Kubectl 1.7.4 or newer for deploying to Kubernetes

Verify prerequisites
```
java -version
mvn --version
docker --version
minikube version
kubectl version --short
```

## Build

```
mvn package
```

## Start the application

```
java -jar target/helloworld.jar
```

## Exercise the application

```
curl -X GET http://localhost:8080/greet
{"message":"Hello World!"}

curl -X GET http://localhost:8080/greet/Joe
{"message":"Hello Joe!"}

curl -X PUT -H "Content-Type: application/json" -d '{"greeting" : "Hola"}' http://localhost:8080/greet/greeting

curl -X GET http://localhost:8080/greet/Jose
{"message":"Hola Jose!"}
```

## Try health and metrics

```
curl -s -X GET http://localhost:8080/health
{"outcome":"UP",...
. . .

# Prometheus Format
curl -s -X GET http://localhost:8080/metrics
# TYPE base:gc_g1_young_generation_count gauge
. . .

# JSON Format
curl -H 'Accept: application/json' -X GET http://localhost:8080/metrics
{"base":...
. . .

```

## Build the Docker Image

```
docker build -t helloworld .
```

## Start the application with Docker

```
docker run --rm -p 8080:8080 helloworld:latest
```

Exercise the application as described above

## Deploy the application to Kubernetes

1. Generate helidon helloworld application boilerplate using this command:
    ```
    mvn archetype:generate 
        -DinteractiveMode=false     
        -DarchetypeGroupId=io.helidon.archetypes    
        -DarchetypeArtifactId=helidon-quickstart-mp     
        -DarchetypeVersion=1.1.1     
        -DgroupId=codes.recursive     
        -DartifactId=helloworld         
        -Dpackage=helloworld
    ```
    
2. Build the docker image for the helidon hello world application 
    ```
    docker build -t arjuns93/helidon-helloworld .
    ```

3. Push this docker image to docker hub
    ```
    docker push arjuns93/helidon-helloworld:latest
    ```

4. Make changes to the app yaml to support the appropriate deployment
    ```
    kind: Service
    apiVersion: v1
    metadata:
      name: helloworld
      labels:
        app: helloworld
    spec:
      type: NodePort
      selector:
        app: helloworld
      ports:
      - port: 8080
        targetPort: 8080
        nodePort: 30303
        name: http
    ---
    kind: Deployment
    apiVersion: apps/v1
    metadata:
      name: helloworld
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: helloworld
      template:
        metadata:
          labels:
            app: helloworld
            version: v1
        spec:
          containers:
          - name: helloworld
            image: arjuns93/helidon-helloworld:latest
            imagePullPolicy: Always
    ---
    ```
5. Deploy the containers onto Kubernetes
    ```
    kubectl apply -f app.yaml
    ```
6. Fetch the endpoint for the application deployed onto K8s
    ```
    minikube service helloworld —url
    ```

