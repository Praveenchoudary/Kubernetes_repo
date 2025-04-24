Before going into replication controller and replica set.when we create a pod. the pod is created container inside
the container the application is running.
when the load on the pod increased the pod will deleted. the application inside the pod not accessible.It is
because of:
a) No self-healing; if a pod fails, it won't restart automatically.
•
It Doesn’t support the features like Auto scaling and Healing.

![image](https://github.com/user-attachments/assets/905c636f-60ba-425a-8cc8-353e6be13a7b)

![image](https://github.com/user-attachments/assets/5768ff21-6985-4b91-9937-a25de1ab46ec)



# Replication Controller & ReplicaSet in Kubernetes


## 📌 What is a Replication Controller?

![image](https://github.com/user-attachments/assets/ff093109-18ab-4838-ab07-3d56fe9d40cb)


A **Replication Controller** is an older Kubernetes object used to ensure that a specified number of pod replicas are running at all times.

### ✅ Key Features:
- Ensures the desired number of pod replicas.
- Replaces failed pods.
- Supports rolling updates (manually).

![image](https://github.com/user-attachments/assets/53c6abba-37a5-42ca-88de-a23f57a66acc)


📌 What is a ReplicaSet?

![image](https://github.com/user-attachments/assets/92f77520-1080-4a42-9446-0ab48943d6f7)


A ReplicaSet is the next-generation controller that replaces the Replication Controller. It's mainly used by Deployments but can also be used independently.
✅ Key Features:

    Similar to ReplicationController but supports set-based selectors.

    Automatically managed by Deployments in production setups.

  ![image](https://github.com/user-attachments/assets/682c9842-fcf1-4262-b3c7-8b53d9e5ff19)


Note:
ReplicationControllers are now considered legacy.

Use ReplicaSets or Deployments in modern Kubernetes workflows.

Deployments provide higher-level features like rolling updates and rollback.
