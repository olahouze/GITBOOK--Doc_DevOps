---
description: Outils pour gérer les ressources d'un cluster K8S
---

# \[Concepts] - Scale UP, Scale DOWN and Manage Scheduling

## Description

Un cluster K8S dispose d'un nombre de ressources (CPU / Mémoire) équivalente à <mark style="color:red;">la somme des ressources de tous les Workers Nodes</mark> pressent dans le cluster.

Par défaut le cluster K8S ne sait pas automatiquement ajouter/supprimer des worker nodes pour gérer de manière optimisé les ressources nécessaires au fonctionnement

{% hint style="info" %}
Si des <mark style="color:blue;">ressources sont manquantes</mark>, les nouveaux PODs resteront en **pending** tant qu'un administrateur ne rajoute pas de ressources dans le cluster (nouveau worker node ou suppressions d'autres POD pour libérer de la ressource)
{% endhint %}

{% hint style="info" %}
Si un cluster est <mark style="color:blue;">sous utilisé</mark>, K8S ne va pas éteindre un worker node qui n'utilise pas ses ressources, celui-ci va resté actif et non utilisé
{% endhint %}

Pour palier ce manque, il existe 2 projets :&#x20;

* **Autoscaler** : Gestion de l'ajout/suppression de workers nodes ([https://github.com/kubernetes/autoscaler](https://github.com/kubernetes/autoscaler))
* **Descheduleur** : Déplacement des PODs dans un cluster pour optimiser l'utilisation des ressources et le fonctionnement du projet Autoscaler

## Fonctionnement

### Autoscaler

Ce projet repose sur des PODs déployés dans le cluster K8S qui va faire l'interface entre le cluster et l'API du provider d'infrastructure (AWS, GCP, OPK, etc...)

Le POD de l'autoscaler va vérifier de manière régulière l’état du cluster (le temps entre chaque versification est paramétrable) et réaliser les appels API sur le provider d'infrastructure en fonction de l’état du cluster et de sa configuration

#### Scale UP

Lorsque le cluster est en manque de ressource, les nouveaux PODs restent en Pending

Lors de la vérification périodique de l'autoscaler, si des PODs sont en Pending, il va faire appel à l'API du Provider pour démarrer une nouvelle machine qui deviendra un nouveau worker node

Vous devez indiquer dans la configuration le(s) nom(s) du (des) NodesGroupes à utiliser dans le cluster pour que l'autoscaler réalise le bon appel API

{% hint style="info" %}
Si vous avez plusieurs NodesGroupes vous pouvez mettre en place des priorités : [https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/expander/priority/readme.md](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/expander/priority/readme.md)
{% endhint %}

![Scale UP](../.gitbook/assets/Autoscaler.drawio.png)

#### Scal DOWN

Lorsque le cluster dispose de worker node qui ne sont plus utilisés par des PODs "applicatifs" (sans prise en compte des DeamonSet) l'autoscaler va prévoir une extinction du worker node

Lors de la vérification périodique de l'autoscaler, si des workers nodes sont inutilisés, il va prévoir la suppression des ceux-ci.

![Scale DOWN](<../.gitbook/assets/Autoscaler-Scale DOWN.drawio.png>)

## Sources

{% embed url="https://github.com/kubernetes/autoscaler" %}
