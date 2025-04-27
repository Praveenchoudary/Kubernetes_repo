Kubernetes - Ephemeral Containers
📘 Introduction

Ephemeral containers differ from other containers because:

    They lack guarantees for resources or execution.

    They are never automatically restarted.

    They are not appropriate for building applications.

Ephemeral containers are described using the same ContainerSpec as regular containers, but many fields are incompatible and disallowed for ephemeral containers.
❗ Important Points

    Ephemeral containers may not have ports, so fields like ports, livenessProbe, and readinessProbe are disallowed.

    Pod resource allocations are immutable, so setting resources is disallowed.

🛠️ Purpose of Ephemeral Containers

Ephemeral containers are useful for interactive troubleshooting when kubectl exec is insufficient because:

    A container has crashed.

    A container image doesn't include debugging utilities.

🛠️ Methods to Debug a Pod
a. Debug Using Ephemeral Containers

If your pod is running but you are unable to exec into it, you can add an ephemeral container using:

kubectl debug -it <RUNNING-POD> --image=<DEBUG-IMAGE>:<DEBUG-IMAGE-TAG> --target=<RUNNING-CONTAINER>

    <RUNNING-POD>: Name of the existing running pod.

    <RUNNING-CONTAINER>: Name of the container you want to debug.

    This command adds a new debug container (like busybox) into the existing pod and attaches you to it.

b. Debug Using a Copy of the Pod

If your pod has crashed and you cannot exec into it, create a new copy of the pod with a debug container:

kubectl debug <ORIGINAL-POD> -it --image=<DEBUG-IMAGE>:<DEBUG-IMAGE-TAG> --share-processes --copy-to=<NEW-DEBUG-POD>

    <ORIGINAL-POD>: Name of the original crashed pod.

    <NEW-DEBUG-POD>: Name of the new debug pod (copy of the original).

This method creates a new pod including the debug container and the containers from the original pod.
🧪 Example: Debugging with an Ephemeral Container
Step 1: Create a Pod

Create a file named ephemeral-pod.yaml:

apiVersion: v1
kind: Pod
metadata:
  name: ephemeral-pod
spec:
  containers:
  - image: registry.k8s.io/pause:3.1
    name: ephemeral-container
  restartPolicy: Never

Apply the YAML:

kubectl apply -f ephemeral-pod.yaml

Check the pod status:

kubectl get po

Output:

NAME             READY   STATUS    RESTARTS   AGE
ephemeral-pod    1/1     Running   0          3m41s

Step 2: Try to Exec into the Pod

kubectl exec -it ephemeral-pod -- sh

Expected Error:

OCI runtime exec failed: exec failed: unable to start container process: exec: "sh": executable file not found in $PATH: unknown
command terminated with exit code 126

This happens because the pause container doesn't include a shell.
Step 3: Add an Ephemeral Debug Container

Use kubectl debug to attach a new debug container:

kubectl debug -it ephemeral-pod --image=busybox --target=ephemeral-container

Expected Output:

Targeting container "ephemeral-container". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
Defaulting debug container name to debugger-xxxx.
If you don't see a command prompt, try pressing enter.
/ # 
/ # 

Now, you have access to the ephemeral container and can debug!
