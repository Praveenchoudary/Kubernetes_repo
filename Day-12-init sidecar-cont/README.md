Types of containers in Kubernetes

In Kubernetes, containers are deployed and managed within pods. A pod is the smallest and simplest unit in the Kubernetes object model that can be created, deployed, and managed. Here, you can use different container patterns within a single pod to achieve specific functionalities. Here are some common container patterns used in Kubernetes:

    Init Container
    Sidecar Container
    Ephemeral Container
    Multi Container


Init Containers
Description

Init containers are used for initialization tasks before the main application containers start. These tasks can include setting up configurations, initializing databases, or waiting for dependencies to be ready.
Example: nginx-init.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-init
  labels:
    app: nginx
spec:
  initContainers:
    - name: init-container
      image: alpine
      command: ['sh', '-c', 'echo "<h1>This is from INIT container</h1>" >> /usr/share/nginx/html/index.html']
      volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
  volumes:
    - name: data
      emptyDir: {}

How to Apply:

kubectl apply -f nginx-init.yaml

Service Configuration: nginx-service.yaml

apiVersion: v1
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

Apply the service:

kubectl apply -f nginx-service.yaml

Expected Output:

Access the NGINX endpoint, and you should see the content injected by the init container (<h1>This is from INIT container</h1>).
Sidecar Containers
Description

Sidecar containers run alongside the main application container in the same Pod and provide supplementary services such as logging, monitoring, or security functionalities.
Example: nginx-sidecar.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
      volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
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

How to Apply:

kubectl apply -f nginx-sidecar.yaml

Service Configuration: nginx-svc.yaml

(Use the same service configuration as in the Init container example.)
Access Logs:

kubectl logs nginx-pod -c sidecar-container

You should see logs like:

10.244.0.1 - - [17/Jan/2024:18:25:02 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" "-"

Ephemeral Containers
Description

Ephemeral containers are temporary containers that are used mainly for debugging purposes. They don't have guarantees for resource allocation, and they are not automatically restarted.
Example: ephemeral-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: ephemeral-pod
spec:
  containers:
    - image: registry.k8s.io/pause:3.1
      name: ephemeral-container
  restartPolicy: Never

How to Apply:

kubectl apply -f ephemeral-pod.yaml

Debugging a Running Pod:

kubectl debug -it ephemeral-pod --image=busybox --target=ephemeral-container

This command will add a debug container and allow you to interact with the pod.
Debugging a Crashed Pod (copy method):

kubectl debug <ORIGINAL-POD> -it --image=busybox --share-processes --copy-to=<NEW-DEBUG-POD>

Multi-Container Pods
Description

Multi-container Pods allow multiple containers to share the same network namespace and resources within a single Pod. This is useful for tightly coupled applications that need to run together.
Example: nginx-multi-container-pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
      volumeMounts:
        - name: data
          mountPath: /usr/share/nginx/html
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

How to Apply:

kubectl apply -f nginx-multi-container-pod.yaml

Service Configuration: nginx-svc.yaml

(Use the same service configuration as in the Init container example.)
Expected Output:

Access the NGINX endpoint, and the page will display the current date being updated every second.
Conclusion

This repository demonstrates various container patterns in Kubernetes, including Init Containers, Sidecar Containers, Ephemeral Containers, and Multi-Container Pods. These patterns help you design and manage more complex applications, from initialization to logging and debugging.
