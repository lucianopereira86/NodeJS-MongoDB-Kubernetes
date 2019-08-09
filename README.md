![titulo](/docs/titulo.JPG)

# NodeJS-MongoDB-Kubernetes

Upload a NodeJS API and a MongoDB to Kubernetes at Google Cloud Platform by using Docker.

## Technologies:

- NodeJS Express
- MongoDB
- Docker
- Google Cloud Platform
- Kubernetes

## Before Start

To follow this guide you must have a basic understanding about Docker and containers.
For any doubts, access the [docs](https://docs.docker.com/).

What is Kubernetes and why to use it?
First of all you have to know about [Google Cloud Platform](https://console.cloud.google.com).
Accordingly to [ZDNet](https://www.zdnet.com/article/what-google-cloud-platform-is-and-why-youd-use-it/):

> You use a cloud platform such as GCP when you want to build and run an application that can leverage the power of hyperscale data centers in some way: to reach users worldwide, or to borrow sophisticated analytics and AI functions, or to utilize massive data storage, or to take advantage of cost efficiencies. You pay not for the machine but for the resources the machine uses.

Basically, if you want to host your apps in the web and do not want to worry about scaling, volume, security and performance, GCP is a great choice and Kubernetes is one of the best services it provides,

Now, let's go to the [Kubernetes documentation](https://cloud.google.com/kubernetes-engine/?&utm_source=google&utm_medium=cpc&utm_campaign=latam-BR-all-pt-dr-skws-all-all-trial-e-dr-1007178-LUAC0008679&utm_content=text-ad-none-none-DEV_c-CRE_325592090332-ADGP_SKWS+%7C+Multi+~+Compute+%7C+Kubernetes-KWID_43700040482195549-kwd-299675360776-userloc_1031424&utm_term=KW_kubernetes-ST_Kubernetes&gclid=Cj0KCQjwkK_qBRD8ARIsAOteukAzxctsrV53P5iaRmVcNWgVXarKph5jZQNsKNoHB7xp4U7I63liiT8aAp3aEALw_wcB&gclsrc=aw.ds) to learn about it:

> Kubernetes Engine (GKE) is a managed, production-ready environment for deploying containerized applications (...) enables rapid application development and iteration by making it easy to deploy, update, and manage your applications and services (...) Use routine health checks to detect and replace hung, or crashed, applications inside your deployments (...) autoscaling allows you to handle increased user demand for your services, keeping them available when it matters most.

These are some basic characteristics about Kubernetes and, in this guide, you will be able to create a cluster, nodes, pods and a persistent storage to make your services running in the cloud. Let's go!

## Google Cloud Platform

Go to [Google Cloud Platform](https://console.cloud.google.com/projectcreate?previousPage=folder%3D&organizationId=0) and create a new project.

![gcp01](/docs/gcp01.JPG)

Access Kubernetes Engine > Clusters and click on "Create Cluster".

![gcp02](/docs/gcp02.JPG)

Rename the cluster and press on "Create". 3 nodes will be created by default with 3.75 GB each.
![gcp03](/docs/gcp03.JPG)

Wait for the cluster activation and this will be the result:

![gcp04](/docs/gcp04.JPG)

## MongoDB

Now let's create a MongoDB storage in our cluster.
First, install the GCloud CLI in your machine by following this [documentation](https://cloud.google.com/sdk/gcloud/).
The Kubernetes also has its own CLI, so install it [here](https://kubernetes.io/docs/reference/kubectl/).
Then, download or clone this repository, open the terminal at the root folder and run the commands below.

Set your project for GCloud

```batch
gcloud config set project nodejs-mongodb-kubernetes
```

Update GCloud components

```batch
 gcloud components update
```

Set your cluster for GCloud

```batch
gcloud container clusters get-credentials my-cluster --zone us-central1-a
```

Create a SSD storage on Kubernetes

```batch
kubectl apply -f googlecloud_ssd.yaml
```

At Storage > Storage classes, a SSD type storage named "fast" will be created

![gcp05](/docs/gcp05.JPG)

Create a MongoDB volume inside the "fast" storage.

```batch
kubectl apply -f mongo-statefulset.yaml
```

It will appear at Persistant Volume Claims tab

![gcp06](/docs/gcp06.JPG)

A service pod will be created at Services & Ingress

![gcp07](/docs/gcp07.JPG)

And at Workloads, a Stateful Set will be created as well

![gcp08](/docs/gcp08.JPG)

Check if the stateful set is ready

```batch
kubectl get statefulset
```

There must be 1 Desired and 1 Current

![gcp09](/docs/gcp09.JPG)

Check if the pods are ready

```batch
kubectl get pods
```

Both of them must be running

![gcp10](/docs/gcp10.JPG)

Connect with the MongoDB instance

```batch
kubectl exec -ti mongo-0 mongo
```

Inside the prompt, initialize the database

```batch
rs.initiate()
```

Open the database configuration

```batch
rs.conf()
```

Notice the host as "mongo-0:27017".
![gcp11](/docs/gcp11.JPG)

## NodeJS

Now, let's upload our NodeJS API. The connection string inside the "server.js" is "mongodb://mongo-0.mongo:27017/crud\_?".

Build an image from this project

```batch
docker build -t gcr.io/nodejs-mongodb-kubernetes/nodejs:v1 .
```

Upload the image to Google Cloud Platform

```batch
gcloud docker -- push gcr.io/nodejs-mongodb-kubernetes/nodejs:v1
```

Run the image on Kubernetes

```batch
kubectl run nodejs --image=gcr.io/nodejs-mongodb-kubernetes/nodejs:v1 --port 3000
```

Check if there is a pod running the API

```batch
kubectl get pods
```

![gcp12](/docs/gcp12.JPG)

Expose the API to internet on port 3000

```batch
kubectl expose deployment nodejs --type=LoadBalancer --port 3000 --target-port 3000
```

Finally, a public IP will be created for the NodeJS API.

![gcp13](/docs/gcp13.JPG)

Access it and the Swagger will be available.

![swagger](/docs/swagger.JPG)

Now, have fun with your CRUD running in the cloud \^\_\^.

## References

For more informations about deployment of projects to Kubernetes and Google Cloud Platform, read these articles:
https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app
https://www.sitepoint.com/kubernetes-deploy-node-js-docker-app/
https://codelabs.developers.google.com/codelabs/cloud-mongodb-statefulset/index.html?index=..%2F..index#0
