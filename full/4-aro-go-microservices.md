---
Title: 4 - Deploy a Ratings application in the ARO cluster
Description: Follow these instructions to You will be deploying a ratings application in the ARO cluster
Author: David Okeyode
---
# Lesson 4: Deploy a Ratings application in the ARO cluster
In this workshop lesson, you will be deploying a ratings application in your ARO cluster. 
![Application](../img/4-app-overview.png)
* **A MongoDB with pre-loaded data**
   * [Link to the pre-loaded data](https://github.com/microsoft/rating-api/raw/master/data.tar.gz)

* **A public facing API `rating-api`**
   * [Code GitHub repo](https://github.com/microsoft/rating-api)
   * A NodeJS application that connects to mongoDB container app to retrieve and rate items
   * The container exposes port 8080
   * MongoDB connection is configured using an environment variable called MONGODB_URI
   * We’ll build the container app using the [source-to-image (S2I) build strategy](https://aroworkshop.io/#source-to-image-s2i)
   
* **A public facing web frontend `rating-web`**
   * [Code GitHub repo](https://github.com/microsoft/rating-web)  
   * A NodeJS application that connects to the rating-api container app 
   * The web app connects to the API over the internal cluster DNS, using a proxy through an environment variable named API
   * The container exposes port 8080
   * ratings-api connection is configured using an environment variable called `API` 
   * We'll build the container app using the [source-to-image (S2I) build strategy](https://aroworkshop.io/#source-to-image-s2i)

Once you're done, you'll have an experience similar to the below.

![Application](../img/4-app-overview-1.png)
![Application](../img/4-app-overview-2.png)
![Application](../img/4-app-overview-3.png)


Here's what we'll be completing in this lesson:

> * Creating a [project](https://docs.openshift.com/aro/4/applications/projects/working-with-projects.html) on the Azure Red Hat OpenShift Web Console
> * Deploying a MongoDB container that uses Azure Disks for [persistent storage](https://docs.openshift.com/aro/4/storage/understanding-persistent-storage.html)
> * Deploying a Node JS API and frontend app from Git Hub using [Source-To-Image (S2I)](https://docs.openshift.com/aro/4/openshift_images/create-images.html)
> * Exposing the web application frontend using [Routes](https://docs.openshift.com/aro/4/networking/routes/route-configuration.html)
> * Creating a [network policy](https://docs.openshift.com/aro/4/networking/network_policy/about-network-policy.html) to control communication between the different tiers in the application


## Create an OpenShift Project
1. **Obtain the kubeadmin credentials using the command below**
   ```
   LOCATION=uksouth       # the location of your cluster
   RESOURCEGROUP=aro-workshop-rg   # the resource group of your cluster that you created in the last lesson           
   CLUSTER=arocluster        # the name of your cluster
   ```
```
    az aro list-credentials \
      --name $CLUSTER \
      --resource-group $RESOURCEGROUP
```

2. **Connect to the OpenShift cluster web console**
* Obtain the URL of the web console using the command
```
 consoleURL=$(az aro show \
    --name $CLUSTER \
    --resource-group $RESOURCEGROUP \
    --query "consoleProfile.url" -o tsv)
```
```
echo $consoleURL
```

3. **Create an OpenShift project**
* Open [Azure Cloud Shell](https://shell.azure.com) with OpenShift CLI (oc) installed from the previous task
* Retrieve the `kubeadmin` user password
   * We will store this value in a variable called **kubeadminpass**
```
kubeadminpass=$(az aro list-credentials \
  --name $CLUSTER \
  --resource-group $RESOURCEGROUP \
  --query kubeadminPassword -o tsv)
```
```
echo $kubeadminpass
```
* Retrieve the ARO cluster API server's address
```
apiServer=$(az aro show -g $RESOURCEGROUP -n $CLUSTER --query apiserverProfile.url -o tsv)
```
* Login to the OpenShift cluster's API server using the kubeadmin credentials
```
oc login $apiServer -u kubeadmin -p $kubeadminpass
```
   * You should receive a **Login successful** response after running the command
* Create a new openshift project
```
    oc new-project workshop \
``` 
* Set your current project to the newly created one
```
    oc project workshop
```

4. **Extra Information. Services will be available at the following address:**
    - [service name].[project name].svc.cluster.local
    - E.g. mongodb.ratings-project.svc.cluster.local


## Create the MongoDB container app
1. **Create mongoDB from template**
* ARO provides a container image and template to make creating a new MongoDB database service easy. We will use the `mongodb-persistent` template to define both a deployment configuration and a service.

* View available templates using the command below. The templates are preinstalled in the openshift namespace.
```
    oc get templates -n openshift
```
* Deploy
    oc process openshift//mongodb-persistent \
        -p MONGODB_USER=ratingsuser \
        -p MONGODB_PASSWORD=ratingspassword \
        -p MONGODB_DATABASE=ratingsdb \
        -p MONGODB_ADMIN_PASSWORD=ratingspassword | oc create -f -
    
    # Verify if the mongoDB pod was created successfully
    oc get all
    
    # Retrieve mongoDB service name
    oc get svc mongodb



## **Exercise 3 - Deploy Ratings API App**
- The `rating-api` is a NodeJS application that connects to mongoDB to retrieve and rate items. 
- Below are some of the details that you’ll need to deploy this.
    - `rating-api` on GitHub: [https://github.com/microsoft/rating-api](https://github.com/microsoft/rating-api)
    - The container exposes port 8080
    - MongoDB connection is configured using an environment variable called `MONGODB_URI`


1. **Fork the application to your own GitHub repository**
    - To be able to setup CI/CD webhooks, you’ll need to fork the application into your personal GitHub repository
    - https://github.com/microsoft/rating-api/fork → Fork


2. **Use the OpenShift CLI to deploy the** `**rating-api**`
    - **Note** You’re going to be using [source-to-image (S2I)](https://aroworkshop.io/#source-to-image-s2i) as a build strategy.
    # Build and deploy the container image as a new app
    oc new-app https://github.com/davidokeyode/rating-api --strategy=source
    
    # Verify
    oc status


3. **Configure the required environment variables**
    - The ***MONGODB_URI*** environment variable is required by the ***ratings-api*** app to connect to the mongodb database
    # ratingsuser
    # ratingspassword
    oc set env deploy/rating-api MONGODB_URI=mongodb://ratingsuser:ratingspassword@mongodb:27017/ratingsdb
    
    OR
    
    oc set env deploy/rating-api MONGODB_URI=mongodb://ratingsuser:ratingspassword@mongodb.ratings-project.svc.cluster.local:27017/ratingsdb
    
    OR
    
    Web console (Developer view) --> Project Details --> Overview --> 1 Deployment --> ratings-api --> Environment



4. **Verify that the service is running**
    - OpenShift web console (Developer view) → Project Details → Overview → 1 Deployment → ratings-api → Pods → Select rating-api Pod → Logs
        - Verify “CONNECTED TO mongodb://ratingsuser:ratingspassword@mongodb:27017/ratingsdb”


5. **Retrieve** `**rating-api**` **service hostname**
    - Obtain the rating-api service hostname using the following command:
    - [service name].[project name].svc.cluster.local
    oc get svc rating-api



6. **Setup GitHub webhook**
    - To trigger S2I builds when you push code into your GitHib repo, you’ll need to setup the GitHub webhook.
    # Retrieve the GitHub webhook trigger secret
    # You’ll need use this secret in the GitHub webhook URL
    # Make a note of the output
    oc get bc/rating-api -o=jsonpath='{.spec.triggers..github.secret}'
    
    # Retrieve the GitHub webhook trigger URL from the build configuration
    # Make a note of the "Webhook GitHub URL" section
    oc describe bc/rating-api
    
    # Replace the <secret> placeholder with the secret you retrieved in the previous step to have a URL similar to https://api.otyvsnz3.eastus.aroapp.io:6443/apis/build.openshift.io/v1/namespaces/workshop/buildconfigs/rating-api/webhooks/SECRETSTRING/github.
    # You’ll use this URL to setup the webhook on your GitHub repository
    
    - Go to your GitHub repository and add a webhook
        - https://github.com/davidokeyode/rating-api →  **Settings** → **Webhooks → Add Webhook**
            - **Payload URL**: The URL composition from above
            - **Content type**: application/json
            - Leave other settings
            - Add webhook
    - After adding the webhook, whenever you push a change to your GitHub repository, a new build will automatically start, and upon a successful build a new deployment will start.



## **Exercise 3 - Deploy Ratings frontend App**
- The `rating-web` is a NodeJS application that connects to the `rating-api`. Below are some of the details that you’ll need to deploy this.
- `rating-web` on GitHub: [https://github.com/microsoft/rating-web](https://github.com/microsoft/rating-web)
    - The container exposes port 8080
    - The web app connects to the API over the internal cluster DNS, using a proxy through an environment variable named `API`


1. **Fork the application to your own GitHub repository**
    - To be able to setup CI/CD webhooks, you’ll need to fork the application into your personal GitHub repository.
    - https://github.com/microsoft/rating-web/fork → Fork


2. **Use the OpenShift CLI to deploy the** `**rating-web**`
    - We’ll use the [source-to-image (S2I)](https://aroworkshop.io/#source-to-image-s2i) build strategy
    # Build and deploy the container image as a new app
    oc new-app https://github.com/davidokeyode/rating-web --strategy=source
    
    # Verify
    oc status


3. **Configure the required environment variables**
    - The ***API*** environment variable is required by the ***ratings-web*** app to connect to the ***ratings-api*** service
    
    oc set env deploy rating-web API=http://rating-api:8080
    
    OR
    
    Web console (Developer view) --> Project Details --> Overview --> 2 Deployments --> ratings-web --> Environment --> Add "Single values (env)"


4. **Expose the** `**rating-web**` **service using a Route**
    # Expose the service.
    oc expose svc/rating-web
    
    # Find out the created route hostname
    oc get route rating-web


5. **Test the service**
- Copy the route hostname and browse to it in a browser
- http://<route_hostname>


6. **Setup GitHub webhook**
    - To trigger S2I builds when you push code into your GitHib repo, you’ll need to setup the GitHub webhook.
    # Retrieve the GitHub webhook trigger secret
    # You’ll need use this secret in the GitHub webhook URL
    # Make a note of the output
    oc get bc/rating-web -o=jsonpath='{.spec.triggers..github.secret}'
    
    # Retrieve the GitHub webhook trigger URL from the build configuration
    # Make a note of the "Webhook GitHub URL" section
    oc describe bc/rating-web
    
    # Replace the <secret> placeholder with the secret you retrieved in the previous step to have a URL similar to https://api.otyvsnz3.eastus.aroapp.io:6443/apis/build.openshift.io/v1/namespaces/workshop/buildconfigs/rating-web/webhooks/SECRETSTRING/github.
    # You’ll use this URL to setup the webhook on your GitHub repository


    - Go to your GitHub repository and add a webhook
        - https://github.com/davidokeyode/rating-web →  **Settings** → **Webhooks → Add Webhook**
            - **Payload URL**: The URL composition from above
            - **Content type**: application/json
            - Leave other settings
            - Add webhook
    - After adding the webhook, whenever you push a change to your GitHub repository, a new build will automatically start, and upon a successful build a new deployment will start.


7. **Make a change to the website app and see the rolling update**
    - Go to the https://github.com/davidokeyode/rating-web/blob/master/src/App.vue file in your repository on GitHub
    - Edit the file, and change the `background-color: #999;` line to be `background-color: #0071c5`.
    - Commit the changes to the file into the `master` branch.
    - **Verify build**
        - OpenShift Web Console (Developer view) → Builds → rating-web → Builds
        - You’ll see a new build queued up which was triggered by the push. 
        - Once this is done, it will trigger a new deployment and you should see the new website color updated.
    - **Verify deployment**
        - OpenShift Web Console (Developer view) → More → Project Details → 2 Deployments → rating-web → Events
        - Refresh site once completed



## **Exercise 4 - Create Network Policy**
- Now that you have the application working, it is time to apply some security hardening. You’ll use [network policies](https://docs.openshift.com/aro/4/networking/network_policy/about-network-policy.html) to restrict communication to the `rating-api`.


1. **Create network policy**
- You will create a policy that applies to any pod matching the `app=rating-api` label. 
    - The policy will allow ingress only from pods matching the `app=rating-web` label.
- OpenShift Web Console (Administrator view) → Networking → Network Policies → Make sure “ratings-project” is selected → Create Network Policy
    - Copy and paste the following policy:
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: api-allow-from-web
      namespace: ratings-project
    spec:
      podSelector:
        matchLabels:
          app: rating-api
      ingress:
        - from:
            - podSelector:
                matchLabels:
                  app: rating-web


    - Click “**Create**”







## Next steps

In this lesson, you completed the following:
> * Setup the prerequisites required for your Azure environment
> * Create the required virtual network and subnets for the ARO cluster
> * Create the ARO cluster

Proceed to the next lesson:
> [Connect to the ARO cluster](2-connect-aro-cluster.md)
