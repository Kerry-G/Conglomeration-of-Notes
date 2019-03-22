# Deployment Configuration

- [Deployment Configuration](#deployment-configuration)
  - [Volumes](#volumes)
    - [Introduction to volumes](#introduction-to-volumes)
    - [Spec](#spec)
    - [Types](#types)
    - [Shared Volume Example](#shared-volume-example)
    - [Persistent Volumes and Claims](#persistent-volumes-and-claims)
  - [Dynamic Provisioning](#dynamic-provisioning)
  - [Secrets](#secrets)
    - [Secrets as environment variables](#secrets-as-environment-variables)
    - [Mounting Secrets as Volume](#mounting-secrets-as-volume)
  - [Portable Data with ConfigMaps](#portable-data-with-configmaps)
  - [Using ConfigMaps](#using-configmaps)
  - [Deployment Configuration Status](#deployment-configuration-status)
  - [Scaling and Rolling Update](#scaling-and-rolling-update)
  - [Deployments Rollback](#deployments-rollback)
  - [Labs](#labs)
    - [Configure the Deployment: Secret and ConfigMap](#configure-the-deployment-secret-and-configmap)
    - [Configure the deployment: Attaching Store](#configure-the-deployment-attaching-store)
    - [Using ConfigMaps Configure Ambassador Containers](#using-configmaps-configure-ambassador-containers)
    - [Rolling Updates and Rollback](#rolling-updates-and-rollback)

## Volumes

Containers considered transient, therefore no storage than outlives the container. A k8s volume share the Pod lifetime, but not the containers within, shoulda container terminate, the data would continue to be available to the new container. 

A volume is a directory, possibly pre-populated. As of v1.8 there is 25 different volumes type. Each one are different and has different dependencies. A future feature would be the CSI (Container Storage Interface) that would allow an industry standard interface for a container orchestration to allow arbitrary storage systems. One can use persistent volume if one wants a storage independent from the pod. 

There's two API Objects which exist to provide data to a Pod already. Encoded data can be passed through Secret and non-encoded data can be passe through ConfigMap

### Introduction to volumes

A Pod spec can declare one or + volumes and where they are made available. Each requires a name, type and a mount point. The same volume can be made available to multiple containers within a Pod. It can be a *container-to-container* commmunication. 

A volume can be access by multiple Pod which each given a access mode to write. There is no concurrency checking (data corruption!)

Cluster first groups volumes with the same mode together, then sorts volume by size, from - to +. The claim is then checked against each in that access mode group, until a vol with enough size is found. There is three mode access: 

* RWO: ReadWriteOnce -> Allows read-write by one node
* ROX: ReadOnlyMany -> Allows read-only by many nodes
* RWX: ReadWriteMany -> Allows read-write by many nodes

When a volume is requested, the local kubelet uses *kubelet_pods.go* script to map the raw devices and make the symlink on the host node fs to associate the storage to the container. 

If not storageClass provided, the only parameters are access mode and size, and there is no way to determine which available ones will be uses. 

### Spec

One of the many type of storae is an emptyDir. Not a persistent storage solution. It basicly create a directory in the container, but not mount any storage. This YAML example createa a Pod with a single container and a volume named scracth-volume, and then create the /scratch directory inside the container.

```yaml
kind: Pod
metadata:
    name: busybox
    namespace: default
spec:
    containers:
    - image: busybox
      name: busy
      command:
        - sleep
        - "3600"
      volumeMounts:
    - mountPath: /scratch
      name: scratch-volume
    volumes:
    - name: scratch-volume
            emptyDir: {} 
```

### Types

* GCE/AWS: Volumes of type *GCEpersistentDisk* or *awsElasticBlockStore* allows to mount GCE and EBS disks in your Pods, assuming you have an account and privileges.

* emptyDir / hostPath: easy to use. *emptyDir* is an empty directory that gets erased with the Pod, but recreated when the container restarts. *hostPath* mounts a resource from the host node fs. There are two types: *DirectoryOrCreate* and *FileOrCreate*

* NFS (Network File System) and iSCSI (Internet Small Computer System Interface): straightforward choices for multiple readers scenarios

* rbd / CephFS / GlusterFS: if available, straightforward choices for multiple writer needs

Also: azureDisk, azureFile, csi, downwardAPI, fc, flocker, gitRepo, local, projected, portworxVolume, quobyte, scaleIO, secret, storageos, vsphereVolume, persistentVolumeClaim, etc...

### Shared Volume Example

This yaml example creates a pod with **two** containers with both access to a shared volume.

```yaml
containers:
       - image: busybox
     volumeMounts:
       - mountPath: /busy
     name: test
     name: busy
       - image: busybox
     volumeMounts:
       - mountPath: /box
     name: test
     name: box
     volumes:
       - name: test
       emptyDir: {} 
```

### Persistent Volumes and Claims

A persistent volume (pv) is a way to keep the data even after the Pod is done using it. Pods can define a volume of type *persistentVolumeClaim* with various parameters for size and maybe StorageClass. The cluster then attaches the persistentVolume.

There are several phases to persistent storage:

* Provisioning: Can be from pvs made in advanced or requested from a cloud provider

* Binding: when the control loop on master notice the PVC (persistentVolumeClaim), the watcher locate a PV that match the requests or wait for StorageClass to create one. 

* Use: Bound volume is mount on the Pod.

* Releasing: When the Pod is done, an API request is sent that deletes the PVC. The PV will then wait for another claim. The data that remains depends on the *persistentVolumeReclaimPolicy*

* Reclaim: This has three option:
  * Retain: keeps the data intact
  * Delete: delete the API object and the storage behind it
  * Recycle: Planned to be deprecated but it removes the mountpoint and make it available to a new claim.


Example YAML of a pv using the hostPath type:

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
    name: 10Gpv01
    labels:
         type: local
spec:
    capacity:
        storage: 10Gi
    accessModes:
        - ReadWriteOnce
    hostPath:
        path: "/somepath/data01"
```

Each type has its own config settings. After the pv is created, we need a manifest for a claim and use that claim in the pod definition. In the Pod, the volume uses the persistentVolumeClaim:

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
    name: myclaim
spec:
    accessModes:
        - ReadWriteOnce
    resources:
        requests:
                storage: 8GI
# In the pod:
....
spec:
    containers:
....
    volumes:
        - name: test-volume
          persistentVolumeClaim:
                claimName: myclaim
```

The pod configuration can be a bit more complex depending on the needs:

```yaml
volumeMounts:
      - name: Cephpd
        mountPath: /data/rbd
  volumes:
    - name: rbdpd
      rbd:
        monitors:
        - '10.19.14.22:6789'
        - '10.19.14.23:6789'
        - '10.19.14.24:6789'
        pool: k8s
        image: client
        fsType: ext4
        readOnly: true
        user: admin
        keyring: /etc/ceph/keyring
        imageformat: "2"
        imagefeatures: "layering" 
```

## Dynamic Provisioning

StorageClass API allows an **admin** to define a pv provisioner of a certain type.

With a StorageClass created, a **user** can request a claim. 

Here is an example of a StorageClass using GCE:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast                         # Could be any name
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

## Secrets

the Secret API resource, passwords/tokens can be encoded. 

```bash
kubectl get secrets
kubectl create secret generic mysql --from-literal=password=root
```

Secrets are not encrypted by default, only base64-encoded. The secret will decoded as a string in a file on the container. In order to encrypts secrets, one need to create an EncryptionConfiguration object with a key and proper identity.  The kube-apiserver then needs the --encryption-provider-config flag set to a previously configured provider such as aescbc or ksm. 

Then one need to recreate all of its secrets since they are encrypted on write. 

A secret can be made manually too.

```bash
$ echo LFTr@1n | base64
TEZUckAxbgo=

$ vim secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: LF-secret
data:
  password: TEZUckAxbgo= 
```

### Secrets as environment variables

A secret can be set as an env variables in a Pod. 

```yaml
spec:
    containers:
    - image: mysql:5.5
        env:
        - name: MYSQL_ROOT_PASSWORD
        valueFrom:
            secretKeyRef:
                name: mysql
                key: password
            name: mysql 
```

There is no limit to the number of secrets used, but there's a limit of 1MB limit on their size.

They are stored in the tmpfs storage on the host node. A secret must exist prior to being requested.

### Mounting Secrets as Volume

Secrets can be mounted as files using a volume definition in a pod manifest. 

```yaml
spec:
    containers:
    - image: busybox
      command:
        - sleep
        - "3600"
      volumeMounts:
      - mountPath: /mysqlpassword
        name: mysql
      name: busy
    volumes:
    - name: mysql
        secret:
            secretName: mysql
```

## Portable Data with ConfigMaps

Similar API to Secrets, but data isn't encoded. It's a way to decouple configuration from the container image. Store data as a set of key/value pair or plain config files in any format. 

ConfigMaps can be consumed in various ways:

* Pod env. variables from 1 or + ConfigMaps
* Use ConfigMap values in Pod commands
* Populate Volume from ConfigMap
* Add ConfigMap data to specific path in Volume
* Set file names and access mode in Volume from ConfigMap data
* Can be used by system components and controllers

## Using ConfigMaps

Like Secrets, one can use ConfigMaps as env. variable or using a volume mount. They must exist prior to being used by a Pod, unless marked as optional. IN case of env. variables, the pod manifest will use the valueFrom key and the configMapKeyRef

```yaml
env:
- name: SPECIAL_LEVEL_KEY
  valueFrom:
    configMapKeyRef:
      name: special-config
      key: special.how


# Define a volume with the configMap type
volumes:
    - name: config-volume
      configMap:
        name: special-config
```

## Deployment Configuration Status

The *Status* output is generated when the informations is requested

```yaml
status:
  availableReplicas: 2 # How many replicas were configured by the ReplicaSet. Later gets compared to the later value of readyReplicas to determine if everything is OK
  conditions:
  - lastTransitionTime: 2017-12-21T13:57:07Z
    lastUpdateTime: 2017-12-21T13:57:07Z
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  observedGeneration: 2 # Show how often the deployment has been updated. Usefull to understand the rollout/rollback situation of the deployment
  readyReplicas: 2
  replicas: 2
  updatedReplicas: 2 
```

## Scaling and Rolling Update

API server allows for the configurations settings to be updated. A common udpate is `kubectl scale [deploy/name] --replicas=4`. If replicas would be set to 0, that would mean no containers, but there would still be a *ReplicaSet* and a *Deployment*. 

A non-immutable values can also be edited via a text editor by using `kubectl edit deployment [name]`

## Deployments Rollback

```bash
kubectl run [name] --image=[nameImage] --record                 # Annotate the change-cause in the resource definition
kubectl set image deployment/[name] ghost=ghost:09 --all        # Update all image
#Now if a updates fail
kubectl rollout history deployment/[name]                       # Gets you all the revisions
kubectl rollout undo deployment/[name]                          # Execute the rollback
kubectl rollout undo deployment/[name] --to-revision=[number]   # Rollback to a specific revision
kubectl rollout pause deployment/[name]                         # Pause the deployment
kubectl rollout resume deployment/[name]                        # Resume the deployment
```

Note: You can do an rolling update on ReplicationControllers with `kubectl rolling-update`, but this is done on the client side! (If you close your client, the update will stop.)

## Labs

### Configure the Deployment: Secret and ConfigMap

```bash
kubectl create configmap [name] --from-literal=text=[string] --from-file=./[nameOfFile] # Create a configMap from literals and files
kubectl get configmap [name]            # Get the details about the configmap [name]
kubectl get configmap [name] -o yaml    # Get the yaml detail about the configmap
```

We can then update our yaml file to use the configMap as env. parameters

```yaml
containers:
  env:
  - name: ilike
    valueFrom:
      configMapKeyRef:
        name: colors
        key: favorite
```

and then we can call commands like. (-- means pass the following as standard in)

```bash
kubectl exec -c [appname] -it [PodName] -- /bin/bash -c 'echo $ilike' # Will print 'blue'
```

We can also use the configMap this way, this will add all the information of the configMap to the env. variables.

```yaml
containers:
  envFrom:
    - configMapRef:
      name: colors
```
We can test it by running

```bash
kubectl exec -it [PodName] -- /bin/bash -c 'env'
```

To increase flexiblity, we can create configMaps from yaml files, example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fast-car
  namespace: default
data:
  car.make: Ford
  car.model: Mustang
  car.trim: Shelby
```

```bash
kubectl create -f [nameFile]            # Create the ConfigMap kind
kubectl get configmap fast-car -o yaml  # Get the details in YAML about the configmap
```

We can add the configMap settings to the our app yaml file as a volume, so both containers in the our deployment can access the same volume

```yaml
spec:
  containers:
    - image: [image/name]
      volumeMounts:
        - mountPath: /etc/cars
          name: car-vol
  volumes:                      # Same level as containers:
  - name: car-vol
    configMap:
      defaultMode: 420
      name: fast-car
```

And then we can check the file

```bash
kubectl exec -c [appname] -it [podname] -- /bin/bash -c 'cat /etc/cars/car.trim'
```

### Configure the deployment: Attaching Store

The begining of the tutorial starts by setting up a nfs server on the first VM, I won't write any note about this since it is not related to K8s.

Example of a pv YAML and its claim that uses NFS

```yaml

# pv

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvvol-1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /opt/sfw
    server: [ip]
    readOnly: false

# pvc

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-one
spec:
  accessModes:
  - ReadWriteMany
  resources:
     requests:
       storage: 200Mi

```

### Using ConfigMaps Configure Ambassador Containers

This is an example of a configmap that send the information to fluentd to do its job. More doc can be found about fluentd [here](https://docs.fluentd.org/v1.0/categories/config-file)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluentd.conf: |
    <source>
      @type tail
      format none
      path /var/log/nginx/access.log
      tag count.format1
    </source>

    <match *.**>
      @type forward

      <server>
        name localhost
        host 127.0.0.1
      </server>
    </match>
```

### Rolling Updates and Rollback



```bash
kubectl edit deployment [name]                          # Change settings in the deployment to change the image version
kubectl get events                                      # Verify the event
kubectl rollout history deployment [name]               # View the update history of the deployment.
kubectl rollout undo --dry-run=true deployment/[name]   # View the change that would happen if rollout
kubectl rollout undo deployment [name] --to-revision=1  # Apply the effect we saw with --dry-run
```