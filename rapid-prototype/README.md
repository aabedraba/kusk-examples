# Kusk Rapid Prototyping Example

This is the code used in the blog article _[Rapidly prototype your APIs on Kubernetes with Kusk Gateway](kubeshop.io)_

## How to run

### 1- Install Kusk in your cluster

Download the cli and run

```sh
kusk install
```

Get the gateway External IP. If you're running on a local cluster, you can port-forward to the service.

```sh
kubectl get -n kusk-system svc/kusk-gateway-envoy-fleet
```

### 2- Mock the API

This will take the `example` field from the `openapi.yaml` and will provide mock responses.

Apply the OpenAPI API and configuration to the gateway:

```sh
kusk api generate -i openapi.yaml --envoyfleet.name kusk-gateway-envoy-fleet | kubectl apply -f -
```

Now you can test the mocked API:

```sh
curl EXTERNAL_IP/hello

{"message": "Hello from a mocked service!"}
```

### 3- Connect a service



#### Build the `hello-world-service` container and push it to your container repository of choice

```sh
cd hello-world-container
docker build -t $DOCKER_ORG/hello-world-service:latest
docker push $DOCKER_ORG/hello-world-service:latest
```

#### Update your image in the `deployment.yaml` file

#### Apply the Deployment

```sh
kubectl apply -f deployment.yaml
```

#### Disable mocking and connect the gateway to the new upstream

In the `openapi.yaml` file

```diff
x-kusk:
  cors:
    origins:
      - "*"
    methods:
      - GET
      - POST
-  mocking:
-    enabled: true
+  upstream:
+    service:
+      name: hello-world-svc
+      namespace: default
+      port: 8080
```

Apply the new changes to the Gateway with

```sh
kusk api generate -i openapi.yaml --envoyfleet.name kusk-gateway-envoy-fleet | kubectl apply -f -
```

And now test the connected service

```sh
curl EXTERNAL_IP/hello

{"message": "Hello from an implemented service!"}
```
