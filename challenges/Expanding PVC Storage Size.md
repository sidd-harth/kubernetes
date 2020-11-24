# Expanding PVC Storage Size

Expanding `PVC` Storage Size depends upon the `StorageClasses` being used.

Edit any existing or create a `StorageClass` with `allowVolumeExpansion` field set to `true`

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true      # <--- this should be true
mountOptions:
  - debug
volumeBindingMode: Immediate
```

The following types of volumes support volume expansion, when the underlying `StorageClass` has the field `allowVolumeExpansion set to true`.

|Volume type | Required Kubernetes version|
|----|----|
|gcePersistentDisk	|1.11|
|awsElasticBlockStore	|1.11|
|Cinder	|1.11|
|glusterfs	|1.11|
|rbd	|1.11|
|Azure File	|1.11|
|Azure Disk	|1.11|
|Portworx	|1.11|
|FlexVolume	|1.13|
|CSI	|1.14 (alpha), 1.16 (beta)|

### Resize a existing PVC named `myclaim`
```
kubectl patch pvc myclaim -p '{"spec":{"resources":{"requests":{"storage":"10Gi"}}}}'
```
#### Points to note -
- only dynamically provisioned pvc can be resized
- the storageclass that provisions the pvc must support resize
```
Error from server (Forbidden): persistentvolumeclaims "myclaim" is forbidden: only dynamically provisioned pvc can be resized and the storageclass that provisions the pvc must support resize
```
- storage size can only be increased 
```
The PersistentVolumeClaim "myclaim" is invalid: spec.resources.requests.storage: Forbidden: field can not be less than previous value
```