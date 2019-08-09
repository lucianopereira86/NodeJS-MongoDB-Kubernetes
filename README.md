![titulo](/docs/titulo.JPG)

# NodeJS-MongoDB-Kubernetes

Upload a NodeJS API connected with MongoDB to Kubernetes on Google Cloud Platform.

## Technologies:

- NodeJS Express
- MongoDB
- Docker
- Google Cloud Platform
- Kubernetes

## Before Start

First of all, download or clone this repository. It has the same project from [CRUD-NodeJS-Swagger-MongoDB](https://github.com/lucianopereira86/CRUD-NodeJS-Swagger-MongoDB), but with some new files:
- Dockerfile: Used by Docker to build a NodeJS API image.
- deployment.yml: Exposes the NodeJS API to internet on Kubernetes.
- googlecloud_ssd.yaml: Creates a SSD storage on Kubernetes.
- mongo-statefulset.yaml: Creates a MongoDB volume on Kubernetes.

Also, the "server.js" was edited to receive a new connection string with MongoDB hosted in the cloud.
To follow this guide you must have a basic understanding about Docker and containers.
For any doubts, access the [docs](https://docs.docker.com/).

## Google Cloud Platform

Why to use [Google Cloud Platform](https://console.cloud.google.com)?
Accordingly to [ZDNet](https://www.zdnet.com/article/what-google-cloud-platform-is-and-why-youd-use-it/):

> You use a cloud platform such as GCP when you want to build and run an application that can leverage the power of hyperscale data centers in some way: to reach users worldwide, or to borrow sophisticated analytics and AI functions, or to utilize massive data storage, or to take advantage of cost efficiencies. You pay not for the machine but for the resources the machine uses.

Basically, if you want to host your apps in the web and do not want to worry about scaling, volume, security and performance, GCP is a great choice.

## Kubernetes

What is Kubernetes?
As the [Kubernetes documentation](https://cloud.google.com/kubernetes-engine/?&utm_source=google&utm_medium=cpc&utm_campaign=latam-BR-all-pt-dr-skws-all-all-trial-e-dr-1007178-LUAC0008679&utm_content=text-ad-none-none-DEV_c-CRE_325592090332-ADGP_SKWS+%7C+Multi+~+Compute+%7C+Kubernetes-KWID_43700040482195549-kwd-299675360776-userloc_1031424&utm_term=KW_kubernetes-ST_Kubernetes&gclid=Cj0KCQjwkK_qBRD8ARIsAOteukAzxctsrV53P5iaRmVcNWgVXarKph5jZQNsKNoHB7xp4U7I63liiT8aAp3aEALw_wcB&gclsrc=aw.ds) says:

> Kubernetes Engine (GKE) is a managed, production-ready environment for deploying containerized applications (...) enables rapid application development and iteration by making it easy to deploy, update, and manage your applications and services (...) Use routine health checks to detect and replace hung, or crashed, applications inside your deployments (...) autoscaling allows you to handle increased user demand for your services, keeping them available when it matters most.

These are some basic characteristics about Kubernetes. 
In this guide, you will be able to create a cluster, nodes, pods and a persistent storage to make your services running on Kubernetes. Let's go!

## Creating a cluster

Go to [Google Cloud Platform](https://console.cloud.google.com/projectcreate?previousPage=folder%3D&organizationId=0) and create a new project.

![gcp01](/docs/gcp01.JPG)

Access *Kubernetes Engine > Clusters* and click on "Create Cluster".

![gcp02](/docs/gcp02.JPG)

Rename the cluster and press on "Create". 3 nodes will be created by default with 3.75 GB each.

![gcp03](/docs/gcp03.JPG)

Wait for the cluster activation and this will be the result:

![gcp04](/docs/gcp04.JPG)

## MongoDB

Now let's create a MongoDB storage in our cluster.
First, install the GCloud CLI in your machine by following these [docs](https://cloud.google.com/sdk/gcloud/).
Kubernetes also has its own CLI, so install it [here](https://kubernetes.io/docs/reference/kubectl/).
At the root folder of this project, run the commands below:

Set your project for GCloud.

```batch
gcloud config set project nodejs-mongodb-kubernetes
```

Update GCloud components.

```batch
 gcloud components update
```

Set your cluster for GCloud.

```batch
gcloud container clusters get-credentials my-cluster --zone us-central1-a
```

Create a SSD storage on Kubernetes.

```batch
kubectl apply -f googlecloud_ssd.yaml
```

At *Storage > Storage classes*, a SSD type storage named "fast" will be created.

![gcp05](/docs/gcp05.JPG)

Create a MongoDB volume inside the "fast" storage.

```batch
kubectl apply -f mongo-statefulset.yaml
```

Check if the stateful set is ready.

```batch
kubectl get statefulset
```

There must be 1 Desired and 1 Current.

![gcp09](/docs/gcp09.JPG)

Check if the pods are ready.

```batch
kubectl get pods
```

Both of them must be running.

![gcp10](/docs/gcp10.JPG)

The MongoDB volume will appear at *Persistant Volume Claims* tab.

![gcp06](/docs/gcp06.JPG)

A service pod will be created at *Services & Ingress*.

![gcp07](/docs/gcp07.JPG)

And at *Workloads*, a Stateful Set will be created as well.

![gcp08](/docs/gcp08.JPG)

Connect with the MongoDB instance.

```batch
kubectl exec -ti mongo-0 mongo
```

Inside the prompt, initialize the database.

```batch
rs.initiate()
```

Open the database configuration.

```batch
rs.conf()
```

Notice the host as "mongo-0:27017". It's similar to the connection string inside the "server.js" file.

![gcp11](/docs/gcp11.JPG)

## NodeJS

Now, let's upload our NodeJS API to the cloud.

Build an image from this project with the name "gcr.io/nodejs-mongodb-kubernetes/nodejs" and tag "v1".
Everytime you need to update the image, change only the tag.

```batch
docker build -t gcr.io/nodejs-mongodb-kubernetes/nodejs:v1 .
```

Upload the image to Google Cloud Platform.

```batch
gcloud docker -- push gcr.io/nodejs-mongodb-kubernetes/nodejs:v1
```

Run the image on Kubernetes.

```batch
kubectl run nodejs --image=gcr.io/nodejs-mongodb-kubernetes/nodejs:v1 --port 3000
```

Check if there is a pod running the API.

```batch
kubectl get pods
```

![gcp12](/docs/gcp12.JPG)

Expose the API to internet on port 3000.

```batch
kubectl expose deployment nodejs --type=LoadBalancer --port 3000 --target-port 3000
```

Finally, a public IP will be created for the NodeJS API on Google Cloud Platform.

![gcp13](/docs/gcp13.JPG)

Access it and the Swagger will be available.

![swagger](/docs/swagger.JPG)

Now, have fun with your CRUD running in the cloud \^\_\^.

## References

For more informations about deployment of projects to Kubernetes and Google Cloud Platform, read these articles:

https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app

https://www.sitepoint.com/kubernetes-deploy-node-js-docker-app/

https://codelabs.developers.google.com/codelabs/cloud-mongodb-statefulset/index.html?index=..%2F..index#0

