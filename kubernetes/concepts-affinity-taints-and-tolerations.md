# \[Concepts] - !!! Affinity/NodeGroupe et Taints/Tolerations

## Description

Les PODs doivent êtres démarrés sur des Nodes afin de disposer de ressources (CPU/Me-moire/Stockage/réseau)

Il est possible de configurer les nodes et les PODs avec des éléments spécifiques K8S afin d'organiser le cluster et de spécifier sur quel Nodes des PODs vont s’exécuter

Pour cela, K8S utilise les elements suivants :

### Affinity/NodeGroupe

Ces éléments permettent de regrouper des PODs avec des Nodes (= <mark style="color:red;">**aimant attirant**</mark>)

#### NodeGroupe

Ce type d’élément permet d'identifier des worker nodes dans le cluster en les estampillant avec une étiquette particulière.

Les workers nodes disposent d'un **label** qui indique à quel NodeGroupe ils appartiennent

#### Affinity

Ce type d’élément permet d'indiquer, dans la définition d'un POD, sur quel Node le POD doit s’exécuter.La valeur de l’élément Affinity doit correspondre à des valeurs de NodeGroupe présent dans le cluster

Le cluster va chercher à exécuter les PODs avec ces Affinity sur les Nodes appartenant au NodeGroupe correspondant



## Exemple

### NodeGroupe

Lorsque nous créons un NodeGroupe dans le cluster, les workers nodes qui sont démarrés dans ce NodeGroupe se retrouvent avec le label spécifique (exemple ici chez AWS pour le node groupe : **common-spot**)

```
eks.amazonaws.com/nodegroup=common-spot
```

### Affinity

Exemple de définition d'affinity dans un POD

```
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: eks.amazonaws.com/nodegroup
            operator: In
            values:
            - common-spot
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 10
        preference:
          matchExpressions:
          - key: another-node-label-key-high
            operator: In
            values:
            - another-node-label-value-high
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
```

Dans cet exemple nous avons :&#x20;

* <mark style="color:blue;">**requiredDuringSchedulingIgnoredDuringExecution**</mark> : <mark style="color:red;">**Oblige**</mark> le POD a s’exécuter sur un node avec le label <mark style="color:red;">**eks.amazonaws.com/nodegroup = common-spot**</mark>
* <mark style="color:blue;">**preferredDuringSchedulingIgnoredDuringExecution**</mark> : <mark style="color:red;">**Préférence**</mark> pour que le POD s’exécute :&#x20;
  * En priorité (poids de 10) sur les Nodes avec le label **another-node-label-key-high = another-node-label-value-high**
  * Si pas possible, en seconde priorité (poids 1) sur les Nodes avec le label **another-node-label-key = another-node-label-value**

{% hint style="info" %}
Il s'agit de <mark style="color:blue;">**preferredDuringSchedulingIgnoredDuringExecution,**</mark> donc si aucun node ne correspond à ces demandes, le POD sera executé sur n'importe quel autre POD qui respecte seulement la demande <mark style="color:blue;">**requiredDuringSchedulingIgnoredDuringExecution**</mark>&#x20;
{% endhint %}



## Sources

{% embed url="https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration" %}
