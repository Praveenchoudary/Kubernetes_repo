# Kubernetes Pods 

In this section, we will explore **Kubernetes Pods**, which are the smallest deployable units in Kubernetes.

## What is a Pod?

A **Pod** is a group of one or more containers that share the same network namespace, storage, and configuration options. Pods are the smallest unit of deployment in Kubernetes and are the foundation of the Kubernetes application model.

### Key Characteristics of a Pod:

- **Single or Multiple Containers**: A Pod can contain one or multiple containers. These containers can communicate with each other using `localhost`.
- **Shared Storage**: Containers in a Pod share storage volumes that can be mounted to their file systems.
- **Network Namespace**: All containers in a Pod share the same IP address and port space.

## Why Pods?

Pods are useful for:

1. **Co-located Containers**: Containers that need to run together, like a web server and a helper program that pushes logs, can be managed as a Pod.
2. **Container Networking**: Pods simplify networking and expose only a single IP address for all containers inside the Pod.
3. **Scaling**: Pods can be scaled easily in Kubernetes, ensuring optimal resource utilization and efficient container management.

   ![image](https://github.com/user-attachments/assets/f572f682-ef40-420f-a733-46a0408551a2)

   ![image](https://github.com/user-attachments/assets/f4509280-87ec-4881-89e4-45e30dc0633e)



## How to Create a Pod?
Creating a Pod - Imperative Way

Besides using YAML files (declarative approach), Kubernetes also allows us to create resources like Pods imperatively using the kubectl command directly from the terminal.
Syntax:

kubectl run <pod-name> --image=<container-image> --port=<port>

Example:

To create a Pod running an nginx container using the imperative approach, run:

kubectl run my-pod --image=nginx:latest --port=80

What happens here?

    kubectl run: The command to create a Pod.

    my-pod: The name of the Pod.

    --image=nginx:latest: The container image used.

    --port=80: The port exposed by the container.

Verify the Pod:

kubectl get pods


In Kubernetes, Pods are created using YAML files, which define the desired state of the application. A basic Pod specification in a YAML file looks like this:

### Example YAML for Creating a Pod

We will create a simple Pod that runs an `nginx` container.

You can create a YAML file (`pod.yaml`) to define the Pod.

## Pod YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80

Explanation of the YAML:

    apiVersion: Specifies the Kubernetes API version.

    kind: The resource type. Here, it is a Pod.

    metadata: Contains metadata like the name of the Pod.

    spec: Specifies the Pod configuration. Here, we define the container's name, image, and exposed port.

In the above YAML:

    A single container (nginx) is defined.

    The container runs the nginx:latest image.

    The container exposes port 80 for web traffic.
In the above YAML:

    A single container (nginx) is defined.

    The container runs the nginx:latest image.

    The container exposes port 80 for web traffic.

Creating the Pod

    Save the YAML file as pod.yaml.

    Run the following command to create the Pod:

kubectl apply -f pod.yaml

    To verify that the Pod is created, run:

kubectl get pods

    You can check the logs of the Pod by using:

kubectl logs my-pod
