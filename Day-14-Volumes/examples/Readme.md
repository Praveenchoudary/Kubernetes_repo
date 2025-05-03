📦 Kubernetes Volumes: A Comprehensive Guide

Kubernetes Volumes are 🔑 essential for managing persistent data. They allow stateful applications to survive pod restarts, enable data sharing between containers, and ensure proper storage lifecycles within a Kubernetes cluster.
1️⃣ Persistent Volumes (PV)
📌 Definition:

A Persistent Volume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically by Kubernetes using Storage Classes.
🧾 Key Attributes:

    🧮 Capacity: Amount of storage (e.g., 10Gi)

    🔐 Access Modes:

        ReadWriteOnce – mounted by one node as read/write

        ReadOnlyMany – mounted by many nodes as read-only

        ReadWriteMany – mounted by many nodes as read/write

    🏷 StorageClassName: Links PV to a Storage Class

    ♻️ Reclaim Policy: Retain, Delete, or Recycle

📄 Example: Manual PV
'''
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
...
2️⃣ Persistent Volume Claims (PVC)
📌 Definition:

A Persistent Volume Claim (PVC) is a request for storage by a user or pod. It abstracts the provisioning details and simplifies storage access.
🧾 Key Attributes:

    🔐 Access Modes (same as PV)

    📦 Storage Requests – how much space is needed

    🏷 StorageClassName – determines the backend storage type

📄 Example: PVC
'''
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
  volumeMode: Filesystem
...
3️⃣ Storage Classes (SC)
📌 Definition:

A Storage Class provides a way for administrators to describe the "classes" of storage they offer. It’s key to enabling dynamic provisioning.
🧾 Key Attributes:

    🔧 Provisioner: Volume plugin used (e.g., DigitalOcean CSI)

    ⚙️ Parameters: Additional driver settings

    ♻️ Reclaim Policy

    🧲 Volume Binding Mode

📄 Example: Storage Class
'''
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Delete
volumeBindingMode: Immediate
...
4️⃣ Provisioning of PersistentVolumes

Kubernetes supports two main types of provisioning:
⚙️ Static Provisioning

🧑‍🔧 Administrator manually creates PVs ahead of time.
📌 Steps:
➤ Create PV
'''
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-static-pv
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  csi:
    driver: dobs.csi.digitalocean.com
    fsType: ext4
    volumeAttributes:
      storage.kubernetes.io/csiProvisionerIdentity: <id>-dobs.csi.digitalocean.com
    volumeHandle: <id>
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
...
kubectl apply -f static-pv.yaml

➤ Create PVC
'''
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: static-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  volumeMode: Filesystem
...
kubectl apply -f static-pvc.yaml

➤ Deploy App Using Volume
'''
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alpine-writer
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alpine-writer
  template:
    metadata:
      labels:
        app: alpine-writer
    spec:
      containers:
        - name: alpine-writer
          image: alpine
          command: ["/bin/sh", "-c", "while true; do echo $(date) >> /mnt/data/date.txt; sleep 1; done"]
          volumeMounts:
            - name: data-volume
              mountPath: /mnt/data
      volumes:
        - name: data-volume
          persistentVolumeClaim:
            claimName: static-claim
...
kubectl apply -f deployment.yaml

✅ Verify:

kubectl get pvc,pv,pods

⚡ Dynamic Provisioning

💡 Kubernetes automatically creates a PV when a PVC requests it using a StorageClass.
📌 Steps:
➤ Create Storage Class
'''
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-own-sc
provisioner: dobs.csi.digitalocean.com
reclaimPolicy: Retain
volumeBindingMode: Immediate
allowVolumeExpansion: true
...
kubectl apply -f my-storage-class.yaml

➤ Create PVC
'''
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  storageClassName: my-own-sc
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  volumeMode: Filesystem
...
kubectl apply -f my-dynamic-pvc.yaml

➤ Deploy App
'''
apiVersion: apps/v1
kind: Deployment
metadata:
  name: alpine-writer
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alpine-writer
  template:
    metadata:
      labels:
        app: alpine-writer
    spec:
      containers:
        - name: alpine-writer
          image: alpine
          command: ["/bin/sh", "-c", "while true; do echo $(date) >> /mnt/data/date.txt; sleep 1; done"]
          volumeMounts:
            - name: data-volume
              mountPath: /mnt/data
      volumes:
        - name: data-volume
          persistentVolumeClaim:
            claimName: myclaim
...
kubectl apply -f deployment.yaml

✅ Verify:

kubectl get po,pvc,sc,pv

✅ Summary
Concept	Purpose
📦 PV	Actual storage in the cluster
📝 PVC	User request for storage
🏷 StorageClass	Template for dynamic volume provisioning
⚙️ Static Provision	Admin-managed pre-created volumes
⚡ Dynamic Provision	Auto-created volumes via StorageClass
