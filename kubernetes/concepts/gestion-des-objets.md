# Gestion des objets

## Description

L'ensemble des éléments internes au cluster K8S sont représentes par des **objets**.

Ces objets sont décrits par des **fichiers de configurations** (généralement au format yaml)

Ces objets sont manipulables grâce à **l'API** du cluster K8S (appel web)

## Fonctionnement

La gestion des objets dans K8S s'effectue comme suite :

![Gestion des objets](<../../.gitbook/assets/K8S--API K8S.png>)

**Etape 1** : L'utilisateur demande a l'api K8S (par le biais de l'apiserver) la mise en place d'un objet. Pour cela il fourni un fichier de "**spec**" à K8S.

{% hint style="info" %}
Ce fichier doit contenir un minimum d'information obligatoire
{% endhint %}

**Etape 2 :** L'apiserver dispose d'un versionning des API de K8S (afin de garantir une rétrocompatibilité) et va choisir la bonne version d'API et l'objet qui ont ete fournis dans le fichier "**spec**"

**Etape 3** : Si toutes les étapes de validations sont OK (pas d'erreur dans le fichier "**spec**", validation des "admissions controller, etc...) les informations sont stockés dans la base **etcd**

**Etape 4 :** Le cluster va mettre tout en oeuvre pour que son état corresponde aux demandes dans le fichier "**spec**".

{% hint style="info" %}
Nous pouvons visualiser dans le cluster à chaque instant la différence entre le **status** du cluster et la demande de "**spec**".

Les différents objets disposent des informations :

* **Spec** : demande cible
* **Status** : état réel dans le cluster
{% endhint %}

## Sources

[https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)
