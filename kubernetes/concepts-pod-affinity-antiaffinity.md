# \[Concepts] - Pod Affinity / Antiaffinity

## Description

Lorsque plusieurs PODs doivent s’exécuter pour un même besoin (replicats dans un deployment ou statefullset) le scheduleur va essayer de <mark style="color:blue;">**repartir**</mark> les PODs sur tous les Nodes éligibles pour faire fonctionner le POD

Pour gérer cela en fonction de nos besoins nous utilisons :

### Pod Affinity

Il est possible de specifier à un POD de fonctionner sur le même node qu'un autre POD

Il peut être nécessaire de faire fonctionner différents PODs sur un même node&#x20;

**exemple** : pour accéder au même volume, <mark style="color:blue;">qui ne supporte pas l'acces concurrent,</mark> depuis plusieurs PODs en même temps

### Pod Antiaffinity

Il est possible de spécifier à un POD de <mark style="color:red;">**ne pas**</mark> fonctionner sur le même node qu'un autre POD

Il peut être nécessaire de faire fonctionner différents PODs sur différents nodes et de garantir que plusieurs composants dans  plusieurs deployment/statefullset ne s’exécutent pas sur le même node

**exemple** : Le deployment de plusieurs PODs Applicatif ne doivent pas êtres déployés sur les mêmes Nodes que les PODs SGBD (afin de ne pas perdre toute l'application en cas de dysfonctionnement d'un Node qui contiendrait les PODs SGBD et Applicatifs)

## Fonctionnement

Dans ces deux cas, le scheduleur va identifier le(s) POD(s) de référence en utilisant des labels.

Une fois les PODs références identifiés il va scheduler le nouveau POD en respectant l'affinity/antiaffinity

Il existe, comme pour les nodes affinity, 2 possibilités :&#x20;

* **requiredDuringSchedulingIgnoredDuringExecution**: Le POD doit obligatoirement respecter la règle (sinon il ne sera pas schedulé)
* **preferedDuringSchedulingIgnoredDuringExecution**: Le POD devrait respecter la règle mais peut être schedulé ailleurs si la règle ne peut être respecté

{% hint style="danger" %}
Les règles **nodeAffinity / toleration / podAffinity** sont toutes évaluées lors de la prise de décision. Il faut faire attention que toutes ces règles <mark style="color:red;">**ne rentrent pas en conflit**</mark>

**Exemple** : si une règle de **PodAffinity** _requiredDuringSchedulingIgnoredDuringExecution_  demande au POD de fonctionner au même endroit qu'un autre POD qui se trouve sur le Node A **MAIS** que le Node A ne respecte pas la règle de **NodeAffinity** _requiredDuringSchedulingIgnoredDuringExecution_, le POD <mark style="color:red;">ne démarrera pas</mark>
{% endhint %}

{% hint style="info" %}
La manière simple d’éviter les conflit consiste à&#x20;

* Utiliser les **Taints** et **Node Affinity** pour identifier un groupe de Node sur lesquels exécuter les PODs
* Utiliser les **pod affinity/antiaffinity** pour repartir les POD sur certains Nodes de ce groupe
{% endhint %}

## Exemple

```
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
```

Dans cet exemple, nous demandons que les PODs fonctionnent sur l'environnement qui contient les labels "**security=S1**"

Ces labels peuvent êtres directement sur un Node (non conseillé pour éviter les conflit avec les nodes Affinity) ou sur des PODs qui tournent sur un Node

```
podAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app.kubernetes.io/instance
          operator: In
          values:
          - prometheus
        - key: app.kubernetes.io/name
          operator: In
          values:
          - grafana
      topologyKey: kubernetes.io/hostname
```

Dans cette exemple on demande au POD de fonctionner sur l'environnement qui contient les labels "**app.kubernetes.io/instance=prometheus**" et "**app.kubernetes.io/name=grafana**"

Ici nous utilisons des labels présents dans d'autres PODs pour que les nouveaux POD s’exécutent toujours sur le même Node que ces autres PODs

## Sources

{% embed url="https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node#inter-pod-affinity-and-anti-affinity" %}
