
Kubernetes Volumes: A Comprehensive Guide

Kubernetes Volumes are pivotal for managing data within your cluster. They enable stateful applications and facilitate data sharing between pods, providing persistent storage that transcends the lifecycle of individual containers.
Table of Contents

    Persistent Volumes (PV)

    Persistent Volume Claims (PVC)

    Storage Classes (SC)

    Provisioning of PersistentVolumes

        Static Provisioning

        Dynamic Provisioning

1. Persistent Volumes (PV)

Definition: Persistent Volumes are storage resources provisioned in a Kubernetes cluster, managed by the cluster administrator. They abstract the underlying physical storage, allowing users to access storage without delving into infrastructure specifics.

Key Attributes:

    Capacity: Specifies the size of the volume.

    Access Modes:

        ReadWriteOnce: Mounted as read-write by a single node.

        ReadOnlyMany: Mounted as read-only by multiple nodes.

        ReadWriteMany: Mounted as read-write by multiple nodes.

    Storage Class Name: Associates the PV with a specific Storage Class.

    Volume Mode: Determines whether the volume is mounted as a filesystem or block device.

    Persistent Volume Reclaim Policy: Defines the action taken when a PVC is deleted (Retain, Delete, or Recycle).

    Mount Options: Additional options passed to the mount command.

Example: Manual PV Creation

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

2. Persistent Volume Claims (PVC)

Definition: Persistent Volume Claims are user or application requests for storage within the cluster. They allow users to consume storage without needing to understand the underlying provisioning details.

Key Attributes:

    Access Modes: Requests the required access mode (ReadWriteOnce, ReadOnlyMany, ReadWriteMany).

    Resources: Specifies the minimum size of the volume.

    Storage Class Name: Requests a specific Storage Class.

    Volume Mode: Determines whether the volume is mounted as a filesystem or block device.
    Medium+2Stack Overflow+2Kubernetes+2

Example: Dynamic PV Provisioning with SC

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

3. Storage Classes (SC)

Definition: Storage Classes define the types of storage available in the cluster and their provisioning methods. They enable dynamic provisioning of PVs based on predefined templates or policies.

Key Attributes:

    Provisioner: Specifies the type of volume plugin used for provisioning.

    Parameters: Custom parameters for the provisioner.

    Reclaim Policy: Determines the action taken when a PVC is deleted (Retain, Delete, or Recycle).

    Volume Binding Mode: Determines when a PV is bound to a PVC (Immediate or WaitForFirstConsumer).

    Mount Options: Additional options passed to the mount command.

Example: Creating a Storage Class

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Delete
volumeBindingMode: Immediate

4. Provisioning of PersistentVolumes

PersistentVolumes can be provisioned in two ways: statically or dynamically.
Static Provisioning

In static provisioning, the cluster administrator manually creates PVs with details of available storage. These PVs are pre-defined in the Kubernetes API and are ready for use by cluster users.

Steps:

    Create PersistentVolume (PV):

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

Apply the PV:

kubectl apply -f static-pv.yaml

    Create PersistentVolumeClaim (PVC):

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

Apply the PVC:

kubectl apply -f static-pvc.yaml

    Create Deployment:

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

Apply the Deployment:

kubectl apply -f deployment.yaml

    Verify Resources:

    kubectl get pvc,pv,pod

Dynamic Provisioning

In dynamic provisioning, when no static PV matches a user's PVC, Kubernetes automatically provisions a volume based on the specified StorageClass.

Steps:

    Create StorageClass:

    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: my-own-sc
    provisioner: dobs.csi.digitalocean.com
    reclaimPolicy: Retain
    volumeBindingMode: Immediate
    allowVolumeExpansion: true

Apply the StorageClass:

kubectl apply -f my-storage-class.yaml

    Create PersistentVolumeClaim (PVC):

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

Apply the PVC:

kubectl apply -f my-dynamic-pvc.yaml

    Create Deployment:

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

Apply the Deployment:

kubectl apply -f deployment.yaml

    Verify Resources:

kubectl get po,pvc,sc,pv
