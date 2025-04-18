# Best Buy Cloud-Native Application

## Updated Application Architecture

![image](https://github.com/user-attachments/assets/13462c16-9b85-417f-b2c8-92c28c39b57a)

## Application and Architecture Explanation

The Best Buy application is a demo cloud-native application for Best Buy's online store, designed using a microservices architecture. It allows customers to browse products, place orders, and enables Best Buy employees to manage products and orders. The architecture is based on the Algonquin Pet Store (On Steroids) design, with the key difference being the use of a managed backing service (Azure Service Bus) for the order queue, instead of RabbitMQ.

Here's a breakdown of the application's functionality and architecture:

* **Store-Front:** This is the customer-facing web application. Customers can use it to browse products, add them to their cart, and place orders. It interacts with the Product Service to get product information and the Order Service to create orders.
* **Store-Admin:** This is the employee-facing web application. Best Buy employees can use it to manage product information (add, update, delete products) through the Product Service, and view order information through the Order Service.
* **Order-Service:** This service handles the creation of orders. When a customer places an order, the Store-Front sends a request to the Order Service. The Order Service then sends the order data to Azure Service Bus (the managed message queue).  It also persists order data to MongoDB.
* **Product-Service:** This service manages product data. It provides CRUD (Create, Read, Update, Delete) operations for product information, which are used by both the Store-Front and Store-Admin applications. Product data is stored in MongoDB.
* **Makeline-Service:** This service processes orders. It reads order data from the Azure Service Bus queue, simulates order fulfillment, and marks orders as completed.
* **AI-Service:** This service generates product descriptions and images. It uses GPT-4 to generate text descriptions and DALL-E 3 to generate product images.
* **Database:** MongoDB is used as the database for storing both product and order data.
* **Message Queue:** Azure Service Bus is used as the managed message queue for handling order messages.  The Order Service sends messages to the queue, and the Makeline-Service consumes messages from the queue.

## Deployment Instructions

Here are the step-by-step instructions to deploy the Best Buy application in a Kubernetes cluster:

**Prerequisites:**

1.  A Kubernetes cluster (e.g., Azure Kubernetes Service (AKS)).
2.  kubectl command-line tool configured to connect to your Kubernetes cluster.
3.  Azure CLI installed and configured (if using Azure resources like Service Bus).
4.  An Azure Service Bus Namespace, Queue, and appropriate authorization rules.
5.  Azure OpenAI Service deployed, with GPT-4 and DALL-E 3 models.
6.  Azure AKS cluster with a master node and a worker node

**Deployment Steps:**

1.  **Clone the Microservice Repositories:**
    * Clone this repository listed to your local machine.

2.  **Create an AKS Cluster:**
    * If you don't have an existing AKS cluster, create one using the Azure CLI or the Azure portal.  Here's an example using the Azure CLI:

        ```bash
        az group create --name myResourceGroup --location eastus
        az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 3 --generate-ssh-keys
        ```

        * This will create a resource group named "myResourceGroup" and an AKS cluster named "myAKSCluster" with 3 nodes in the "eastus" region.  You can customize the resource group name, cluster name, node count, and region as needed.
    
    * **Connect to the AKS Cluster:**
        * After the cluster is created, you need to configure `kubectl` to connect to it.  The Azure CLI provides a convenient command to do this:
            ```bash
            az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --overwrite-existing
            ```
            * This command retrieves the cluster's credentials and adds them to your local `kubeconfig` file.  The `--overwrite-existing` flag ensures that any existing configuration for the cluster is overwritten, which is useful if you've connected to other AKS clusters before.

3.  **Set up Azure Service Bus:**
    * If you haven't already, create an Azure Service Bus Namespace.
    * Create a Queue within the Namespace (e.g., "orders").
    * Create Shared Access Policies for the Queue with `Send` and `Listen` permissions.  Note the connection key for both policies.
    * Encode the keys using base64 with the command `echo -n <order-queue-key> | base64`
     * Add the Encoded keys in the `secrets.yaml` file

4.  **Set up Azure OpenAI:**
     * Create an Azure OpenAI Service.
     * Deploy GPT-4 and DALL-E 3 models.
     * Note the API key and endpoint.
     * Add the endpoint and model deployment names to the best-buy-ai-service deployment in `best-buy-all-in-one.yaml` file
     * Encode the key using base64 with the command `echo -n <open-ai-key> | base64`
     * Add the Encoded key in the `secrets.yaml` file

5.  **Deploy the Application:**
    * Use the provided Kubernetes yaml files to deploy the application:
        * `kubectl apply -f secrets.yaml`
        * `kubectl apply -f best-buy-all-in-one`

6.  **Verify the Deployment:**
    * Use `kubectl get pods` to check the status of the deployed pods.  Make sure all pods are running and ready.
    * Use `kubectl get services` to check the services.
    * Use `kubectl logs <pod-name>` to check the logs of individual pods for any errors.

7.  **Access the Application:**
    * Access the Store-Front and Store-Admin applications by visiting the exposed EXTERNAL-IPS.
    * Use the command `kubectl get services` to retrieve the IPs.

## Table of Microservice Repositories

| Service                  | Repository Link                                                              |
| ------------------------ | ---------------------------------------------------------------------------- |
| Best-Buy-Store-Front     | https://github.com/yvanniyonzima-ac/best_buy_store_front                     |
| Best-Buy-Store-Admin     | https://github.com/yvanniyonzima-ac/best_buy_store_admin                     |
| Best-Buy-Order-Service     | https://github.com/yvanniyonzima-ac/best_buy_order_service                     |
| Best-Buy-Product-Service   | https://github.com/yvanniyonzima-ac/best_buy_product_service                   |
| Best-Buy-Makeline-Service  | https://github.com/yvanniyonzima-ac/best_buy_makeline_service                   |
| Best-Buy-AI-Service        | https://github.com/yvanniyonzima-ac/best_buy_ai_service                         |

## Table of Docker Images

| Service                  | Docker Image Link                                                              |
| ------------------------ | ---------------------------------------------------------------------------- |
| best_buy_store_front     | https://hub.docker.com/repository/docker/yvanniyonzima/best_buy_store_front/general     |
| best_buy_store_admin     | https://hub.docker.com/repository/docker/yvanniyonzima/best_buy_store_admin/general     |
| best_buy_order_service     | https://hub.docker.com/repository/docker/yvanniyonzima/best_buy_order_service/general     |
| best_buy_product_service   | https://hub.docker.com/repository/docker/yvanniyonzima/best_buy_product_service/general   |
| best_buy_makeline_service  | https://hub.docker.com/repository/docker/yvanniyonzima/best_buy_makeline_service/general  |
| best_buy_ai_service        | https://hub.docker.com/repository/docker/yvanniyonzima/best_buy_ai_service/general        |

## Issues and Limitations

* **Azure Service Bus Integration:**
    * The application was unable to send messages to the Azure Service Bus queue.  Although the Service Bus namespace received the request, the orders queue did not receive any incoming messages.  No errors were observed in the application logs.  Further investigation into the connection string, queue configuration, and potential network issues is needed.

* **GPT Model Integration:**
    * Initially, a `CrashLoopFeedback` error occurred, possibly due to an incompatibility between the OpenAI version and the Semantic Kernel version.  Updating the OpenAI version resulted in the pod status showing as "Not Running" with "0/1 Ready".  The root cause was not identified.  Using a different Docker image (`ramymohamed/ai-service-l8:latest`) allowed image generation to work, but the model appeared to be trained on pet store products instead of Best Buy products.  This indicates a potential issue with the model configuration or training data.

* **AI Foundry Deployment Issues:**
    * Deleting the Azure resource group containing the OpenAI resource before deleting the associated models and project in AI Foundry resulted in an inaccessible project in AI Foundry.  The project could not be deleted, leading to quota exhaustion.  This highlights a dependency issue in the resource deletion process.

## Video Recording Link

https://www.youtube.com/watch?v=n8yGDRoxNtY&ab_channel=YvanNiyonzima
