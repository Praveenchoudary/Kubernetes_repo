Kubernetes Container PatternsIn Kubernetes, containers are deployed and managed within pods. A pod is the smallest and simplest unit in the Kubernetes object model that can be created, deployed, and managed. Here, you can use different container patterns within a single pod to achieve specific functionalities. Here are some common container patterns used in Kubernetes:Init ContainerSidecar ContainerEphemeral ContainerMulti Container1. Init ContainerA Pod can have multiple containers running apps within it, but it can also have one or more init containers, which are run before the app containers are started. Init containers are designed to run initialization tasks before the main application container starts. They can be used for tasks like setting up configuration files, initializing databases, or waiting for external services to be ready. Init containers are exactly like regular containers, except:Init containers always run to completion.Each init container must complete successfully before the next one starts.Example of using init containers:Create nginx-init.yaml:apiVersion: v1
kind: Pod
metadata:
  name: nginx-init
  labels:
    app: nginx
spec:
  initContainers:
    # Initializing container
    - name: init-container
      image: alpine
      command: ['sh', '-c', 'echo "<h1>This is from INIT container</h1>" >> /usr/share/nginx/html/index.html']
      volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
  containers:
    # application container i.e., main container
    - name: app
      image: nginx
      volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
  volumes:
    - name: data
      emptyDir: {}
Here, the init-container will override the index.html of the nginx home page with the content "This is from INIT container".Create nginx-service.yaml:apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
Create the above pod and service using the following commands:kubectl apply -f nginx-init.yaml
kubectl apply -f nginx-service.yaml
Now, if you try to access the nginx endpoint through a browser, you’ll get the output "This is from INIT container".2. Sidecar ContainerSidecar containers are secondary containers that run along with the main application container within the same Pod. These containers are used to enhance or to extend the functionality of the main application container by providing additional services or functionality, such as logging, monitoring, security, or data synchronization, without directly altering the primary application code.Example of using sidecar containers:Create nginx-sidecar.yaml:apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  # App container
  - name: nginx-container
    image: nginx:latest
    ports:
      - containerPort: 80
    volumeMounts:
      - name: logs
        mountPath: /var/log/nginx
  # This is side container
  - name: sidecar-container
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "tail -f /var/log/nginx/access.log"]
    volumeMounts:
      - name: logs
        mountPath: /var/log/nginx
  volumes:
    - name: logs
      emptyDir: {}
The above pod has two containers: one app container (nginx) and another one is a sidecar container, which we are using to collect the nginx access logs from the main application.Create nginx-svc.yaml:apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
Create the above pod and service using the following commands:kubectl apply -f nginx-sidecar.yaml
kubectl apply -f nginx-svc.yaml
Now, if you try to access the nginx endpoint through a browser, you’ll get the nginx welcome page. Then, if you try to access the sidecar logs using the command below, you’ll get the recent access logs:kubectl logs nginx-pod -c sidecar-container
Output logs:10.244.0.1 - - [17/Jan/2024:18:25:02 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" "-"
10.244.0.1 - - [17/Jan/2024:18:25:02 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://127.0.0.1:53471/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" "-"
10.244.0.1 - - [17/Jan/2024:18:25:24 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" "-"
10.244.0.1 - - [17/Jan/2024:18:25:25 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" "-"
3. Ephemeral ContainerEphemeral containers differ from other containers in that they lack guarantees for resources or execution, and they will never be automatically restarted, so they are not appropriate for building applications. Ephemeral containers are described using the same ContainerSpec as regular containers, but many fields are incompatible and disallowed for ephemeral containers.Ephemeral containers may not have ports, so fields such as ports, livenessProbe, and readinessProbe are disallowed. Pod resource allocations are immutable, so setting resources is disallowed.Ephemeral containers are useful for interactive troubleshooting when kubectl exec is insufficient because a container has crashed or a container image doesn't include debugging utilities.There are two ways to debug a pod:a. Debug using ephemeral containersIf your pod is running and you are unable to exec into it, then you can use this method. You can use the kubectl debug command to add ephemeral containers to a running Pod. This method will create a new container within the same pod.kubectl debug -it <RUNNING-POD> --image=<DEBUG-IMAGE>:<DEBUG-IMAGE-TAG> --target=<RUNNING-CONTAINER>
Here, replace <RUNNING-POD> with your existing pod name and <RUNNING-CONTAINER> with your existing container that you want to debug. This command adds a new busybox container and attaches to it.b. Debug using a copy of the podIf your pod has crashed and you are unable to exec into it, then you can use this method. This method will create a new pod with the new debug container along with the original pod containers.kubectl debug <ORIGINAL-POD> -it --image=<DEBUG-IMAGE>:<DEBUG-IMAGE-TAG> --share-processes --copy-to=<NEW-DEBUG-POD>
Here, replace ORIGINAL-POD with your existing pod name and NEW-DEBUG-POD with the new debug pod name, which is a copy of the original pod.Example:Create ephemeral-pod.yaml:apiVersion: v1
kind: Pod
metadata:
  name: ephemeral-pod
spec:
  containers:
  - image: registry.k8s.io/pause:3.1
    name: ephemeral-container
  restartPolicy: Never
The yaml will create a pause container image and it does not contain debugging utilities.Now, create a pod using the command below:kubectl apply -f ephemeral-pod.yaml
The above command will create an ephemeral-pod in the default namespace.k get po
NAME            READY   STATUS    RESTARTS   AGE
ephemeral-pod   1/1     Running   0          3m41s
Let’s try to exec into it using the command below:kubectl exec -it ephemeral-pod -- sh
But it will give the error below because there is no shell in this container image:OCI runtime exec failed: exec failed: unable to start container process: exec: "sh": executable file not found in $PATH: unknown
command terminated with exit code 126
Now, let’s try to exec into the pod with the help of an ephemeral container. Use the command below to create an ephemeral container and attach it to the above pod (i.e., ephemeral-pod):kubectl debug -it ephemeral-pod --image=busybox --target=ephemeral-container
This command adds a new busybox container and attaches it to ephemeral-pod. The --target parameter targets the container that we want to exec into. Also, this command will automatically attach to the console of the Ephemeral Container.Output:kubectl debug -it ephemeral-pod --image=busybox --target=ephemeral-container
Targeting container "ephemeral-container". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
Defaulting debug container name to debugger-q6svq.
If you don't see a command prompt, try pressing enter.
/ #
/ #
Now you can access the actual container (i.e., ephemeral-container) and debug it.4. Multi ContainerKubernetes allows you to define pods with multiple containers running in parallel. And these containers share the same network namespace and can communicate with each other via localhost. And these multi-container pods are designed for scenarios where tightly coupled processes need to run together, sharing resources and data.ExampleCreate nginx-multi-container-pod.yamlapiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  # App container
  - name: nginx-container
    image: nginx:latest
    ports:
      - containerPort: 80
    volumeMounts:
      - name: data
        mountPath: /usr/share/nginx/html
  # This is extra container
  - name: extra-container
    image: debian
    command: ["/bin/sh", "-c"]
    args:
      - while true; do
          date > /usr/share/nginx/html/index.html;
          sleep 1;
        done
    volumeMounts:
      - name: data
        mountPath: /usr/share/nginx/html
  volumes:
    - name: data
      emptyDir: {}
The above manifest file has two containers: one is for the main application (i.e., nginx), and the second one is an extra container, which will help us to update the nginx web application home page continuously.Create nginx-svc.yamlapiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
Create the above pod and service using the commands below:kubectl apply -f nginx-multi-container-pod.yaml
kubectl apply -f nginx-svc.yaml
Now, if you try to access the nginx endpoint through a browser, you’ll get the date, which is updated by the extra-container from above.
