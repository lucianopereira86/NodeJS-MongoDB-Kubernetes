# NodeJS-MongoDB-Kubernetes

Uploading a NodeJS API and a MongoDB to Google Cloud Platform's Kubernetes with Docker.

## Technologies:

- NodeJS Express
- MongoDB
- Docker
- Google Cloud Platform
- Kubernetes

## Docker

Build image from this project

```batch
docker build -t gcr.io/nodejs-mongodb-kubernetes/nodejs:v1 .
```

Run the image to check if it's ok.

```batch
docker run --rm gcr.io/nodejs-mongodb-kubernetes/nodejs:v1 -p 3000:3000
```

Upload the image to google cloud platform

```batch
gcloud docker -- push gcr.io/nodejs-mongodb-kubernetes/nodejs:v1
```

Create a cluster endpoint

```batch
gcloud container clusters get-credentials standard-cluster-1 --zone us-central1-a
```

```batch
kubectl run nodejs --image=gcr.io/nodejs-mongodb-kubernetes/nodejs:v1 --port 3000
```

To check the pods running

```batch
kubectl get pods
```

Expose the service to Internet

```batch
kubectl expose deployment nodejs --type=LoadBalancer --port 3000 --target-port 3000
```
