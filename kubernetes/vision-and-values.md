---
description: Gestion du stockage dans K8S
---

# \[Concepts] - Stockage

## Description :

Le système K8S est construit pour être le plus agnostique de l’écosystème sous-jacent. La gestion du stockage répond donc à cette architecture grâces aux [StorageClass](vision-and-values.md#storage-class), [PersistenntVolumes (PV)](vision-and-values.md#persistent-volume-pv) et [PersistentVolumesClaim (PVC)](vision-and-values.md#persistent-volume-claim-pvc)

### **Storage Class :**&#x20;

**Définition** : [https://kubernetes.io/docs/concepts/storage/storage-classes/](https://kubernetes.io/docs/concepts/storage/storage-classes/)

Une Storage Classe représente les spécificités d'un média qui va supporter les PV. et permettre d'offrir différentes performances utilisable par les PVC

### **Persistent Volume (PV) :**&#x20;

**Définition** :[https://kubernetes.io/fr/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/fr/docs/concepts/storage/persistent-volumes/)

Un PV est un espace de stockage de donnée manipulable dans un cluster K8S. Il permet de proposer une quantité définie d'espace de stockage que les PODs pouront utiliser.

### **Persistent Volume Claim (PVC) :**&#x20;

**Définition** : [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)

Un PVC est un objet K8S qui peut être appelé par un POD pour demander de l'espace sur un PV



## Fonctionnement

Il existe 2 méthodes pour utiliser ces objets

* **Static Provisioning** : Les PV doivent  êtres créés manuellement par les OPS pour être utilisables par les PVC
* **Dynamic Provisioning** : Les PV sont automatiquement générés lors de l'appel d'un PVC en utilisant une Storage Classe

![](<../.gitbook/assets/K8S--Storage PVC.png>)



## Exemples

### Storage Classe

Voici la définition d'une Storage Classe sur du stockage AWS  service EBS (type = io1) : [https://kubernetes.io/docs/concepts/storage/storage-classes/#aws-ebs](https://kubernetes.io/docs/concepts/storage/storage-classes/#aws-ebs)

<details>

<summary>Storage Classe AWS EBS</summary>

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: aws-ebs
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  iopsPerGB: "10"
  fsType: ext4
```

</details>

### Persistent Volume

Voici la définition d'un PV sur du stockage locale (type = **local**) appartenant à la Storage Classe "**manual**"

<details>

<summary>PV stockage local</summary>

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

</details>

### Persistent Volume Claim

#### Static provisioning

Voici la définition d'un PVC qui demande un PV correspondant à celui créé précédemment [ici](vision-and-values.md#persistent-volume). Comme nous utilisons la storageClassName = manual, le cluster va automatiquement associé ce PVC sur le PV répondant à ce besoin

<details>

<summary>PVC static provisioning</summary>

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

</details>

#### **Dynamic Provisioning**&#x20;

Voici la définition d'un PVC qui va utiliser une Storage Classe permettant de générer à la demande les PV (ici sur AWS). Dans ce cas le driver utilisé dans la storage classe va s'occuper de demander à l'API AWS de créer les volumes adéquats et générer le PV

<details>

<summary>PVC Dynamic Provisioning</summary>

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: aws-ebs
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

</details>

## Sur le Terrain

### Storage Classes

Visualiser les StoragesClasses : **`kubectl get StorageClass --all-namespaces`**

```
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
efs-sc          efs.csi.aws.com         Delete          Immediate              false                  499d
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   true                   501d
```

Nous avons sur ce cluster 2 Storage Classes (nous pouvons les visualiser avec : **`kubectl describe StorageClass XXX`** :&#x20;

<details>

<summary>efs-sc</summary>

```
Name:            efs-sc
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"efs-sc"},"provisioner":"efs.csi.aws.com"}

Provisioner:           efs.csi.aws.com
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```

</details>

<details>

<summary>gp2</summary>

```
Name:            gp2
IsDefaultClass:  Yes
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"name":"gp2"},"parameters":{"fsType":"ext4","type":"gp2"},"provisioner":"kubernetes.io/aws-ebs","volumeBindingMode":"WaitForFirstConsumer"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner:           kubernetes.io/aws-ebs
Parameters:            fsType=ext4,type=gp2
AllowVolumeExpansion:  True
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>

```

</details>

### Persistence Volume

Visualiser les PV : **kubectl get pv --all-namespaces**

```
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                 STORAGECLASS   REASON   AGE
pv-16442277-d8e0-43ad-ab8b-5587bcc149da   50Gi       RWO            Retain           Bound    tyreupload-pirelli/data-tyreupload-pirelli-mariadb-0   gp2                     331d
pv-188c5ada-8385-4ba9-92a9-9a67bcf5a84e   6Gi        RWO            Delete           Bound    superset/superset                                      gp2                     9d
```

Nous avons sur ce cluster plusieurs PV (nous pouvons les visualiser avec : **`kubectl describe pv XXX`**:&#x20;

<details>

<summary>pvc-16442277-d8e0-43ad-ab8b-5587bcc149da</summary>

```
Name:              pvc-16442277-d8e0-43ad-ab8b-5587bcc149da
Labels:            failure-domain.beta.kubernetes.io/region=eu-west-3
                   failure-domain.beta.kubernetes.io/zone=eu-west-3c
Annotations:       kubernetes.io/createdby: aws-ebs-dynamic-provisioner
                   pv.kubernetes.io/bound-by-controller: yes
                   pv.kubernetes.io/provisioned-by: kubernetes.io/aws-ebs
Finalizers:        [kubernetes.io/pv-protection]
StorageClass:      gp2
Status:            Bound
Claim:             tyreupload-pirelli/data-tyreupload-pirelli-mariadb-0
Reclaim Policy:    Retain
Access Modes:      RWO
VolumeMode:        Filesystem
Capacity:          50Gi
Node Affinity:     
  Required Terms:  
    Term 0:        failure-domain.beta.kubernetes.io/zone in [eu-west-3c]
                   failure-domain.beta.kubernetes.io/region in [eu-west-3]
Message:           
Source:
    Type:       AWSElasticBlockStore (a Persistent Disk resource in AWS)
    VolumeID:   aws://eu-west-3c/vol-0c6007de5b5e75569
    FSType:     ext4
    Partition:  0
    ReadOnly:   false
Events:         <none>

```

</details>

### Persistence Volume Claim

Visualiser les PVC : **kubectl get pvc --all-namespaces**

```
NAMESPACE            NAME                                                                                                             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
prometheus           alertmanager-prometheus-kube-prometheus   Bound    pv-b352fe13-94b9-421d-be87-920fe4054d86   1Gi        RWO            gp2            151d
prometheus           prometheus-grafana                        Bound    pv-e30996d5-a63e-42cb-9a28-0463f04ae549   5Gi        RWO            gp2            157d
prometheus           prometheus-prometheus-kube-prometheus     Bound    pv-bb409f4b-7519-4375-ab22-1e63be040dd2   20Gi       RWO            gp2            156d
```

Nous avons sur ce cluster plusieurs PV (nous pouvons les visualiser avec : **`kubectl describe pvc XXX -n [namespace]`**:&#x20;

<details>

<summary>prometheus-grafana</summary>

```
Name:          prometheus-grafana
Namespace:     prometheus
StorageClass:  gp2
Status:        Bound
Volume:        pvc-e30996d5-a63e-42cb-9a28-0463f04ae549
Labels:        app.kubernetes.io/instance=prometheus
               app.kubernetes.io/managed-by=Helm
               app.kubernetes.io/name=grafana
               app.kubernetes.io/version=7.3.5
               helm.sh/chart=grafana-6.1.17
Annotations:   aws-ebs-tagger/tags: {"project": "GDP290"}
               meta.helm.sh/release-name: prometheus
               meta.helm.sh/release-namespace: prometheus
               pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: kubernetes.io/aws-ebs
               volume.kubernetes.io/selected-node: ip-10-1-26-96.eu-west-3.compute.internal
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      5Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       prometheus-grafana-97d686d56-rrwbj
Events:        <none>

```

</details>

## Sources

{% embed url="https://kubernetes.io/docs/concepts/storage/storage-classes" %}

{% embed url="https://kubernetes.io/fr/docs/concepts/storage/persistent-volumes" %}

{% embed url="https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims" %}
