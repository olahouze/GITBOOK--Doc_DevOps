# \[Concept] - Deployment, ReplicatSet, StatefullSet, DeamonSet

## Description :

Pour conserver une cohérence entre les éléments présents sur le cluster et les éléments que l'administrateur a demandé dans ses spécifications, K8S utilise plusieurs types d'objets

### Deployment

**Définition** : [https://kubernetes.io/fr/docs/concepts/workloads/controllers/deployment/](https://kubernetes.io/fr/docs/concepts/workloads/controllers/deployment/)

Ce type d'objet permet de gérer les objets **ReplicatSet** qui sont utiles pour garantir que le cluster dispose bien du bon nombre de POD demandé dans le deployment.

L'administrateur va indiquer dans le deployment le nombre de replicat qu'il souhaite pour un type de POD (serveur web, sgbd, etc...)

Le deployment va creer/mettre à jour le replicaset en consequence

### ReplicaSet

**Définition** : [https://kubernetes.io/fr/docs/concepts/workloads/controllers/replicaset/](https://kubernetes.io/fr/docs/concepts/workloads/controllers/replicaset/)

Ce type d'objet permet au cluster K8S de gérer le bon nombre de POD dans le cluster pour correspondre à la demande déclarative de l'administrateur

{% hint style="danger" %}
Il est recommandé de ne jamais agir sur ces **ReplicaSet** et de manipuler les objets **Deployment** qui se chargent de la gestion des ReplicaSet

La manipulation des ReplicaSet ne doit se faire que dans des cas très spécifiques
{% endhint %}

{% hint style="info" %}
Notez que le nom du ReplicaSet est toujours formaté comme: **`[DEPLOYMENT-NAME]-[RANDOM-STRING]`**
{% endhint %}

### StatefullSet

**Definition** : [https://kubernetes.io/fr/docs/concepts/workloads/controllers/statefulset/](https://kubernetes.io/fr/docs/concepts/workloads/controllers/statefulset/)

Un **StatefullSet** réalise la même chose qu'un Deployment mais avec <mark style="color:red;">une gestion de "l'identité"</mark> des POD et de <mark style="color:red;">l’ordonnancement</mark> <mark style="color:red;">du démarrage</mark> des POD

Dans un Deployment, les POD **n'ont pas d'identité** et lors d'un redémarrage ou d'une mise a jour, de **nouveaux** PODs sont démarrés de manière aléatoires et en parallèles pour atteindre la cible le plus rapidement possible.

{% hint style="info" %}
Cela signifie, par exemple, que les volumes de stockages qui sont associés avec l'ID d'un POD ne peuvent pas se rattacher sur le "**même**" POD après un redémarrage/mis a jour d'un **Deployment**

En effet **** il s'agit de nouveaux PODs qui seront démarrés pour remplir le même rôle que l'ancien (serveur web, serveur applicatif, sgbd, etc...) et l'association **ID Pod <=>Volume** n'est plus possible
{% endhint %}

Un **StatefullSet** permet de réaliser plusieurs choses différentes par rapport au Deployment et notamment:

* <mark style="color:blue;">**Conservation de l'identité du POD**</mark> : lors d'un redémarrage les ID de PODs gérés par un **StatefullSet** <mark style="color:red;">sont conservés</mark>
* <mark style="color:blue;">**Ordre de redémarrage**</mark>** ** : Lorsque X POD doivent demarrer/s'arreter, le **StatefullSet** effectue les actions sur les PODs l'un après l'autre et n'agit <mark style="color:red;">que si les N-1 autres PODs sont dans l’état attendu</mark>

### DeamonSet

Définition : [https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

Un **DeamonSet** permet de demander au cluster d’exécuter les PODs définis dans le DeamonSet <mark style="color:red;">sur chaque Worker Node du cluster</mark>

## Sources

{% embed url="https://kubernetes.io/fr/docs/concepts/workloads/controllers/deployment" %}

{% embed url="https://kubernetes.io/fr/docs/concepts/workloads/controllers/replicaset" %}

{% embed url="https://kubernetes.io/fr/docs/concepts/workloads/controllers/statefulset" %}

{% embed url="https://kubernetes.io/docs/concepts/workloads/controllers/daemonset" %}
