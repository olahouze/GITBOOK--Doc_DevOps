# Présentation

## Description

Le développement d'application est passé d'un mode **monolithique** à un mode **microservices**

Les **microservices** désignent une approche architecturale du développement d'applications.&#x20;

* En tant que cadre architectural, les microservices sont distribués et faiblement couplés, de sorte que les modifications apportées par une équipe n'affectent pas l'ensemble de l'application.&#x20;
* L'avantage des microservices est que les équipes de développement sont en mesure de créer rapidement de nouveaux composants d'applications pour répondre aux évolutions des besoins de l'entreprise

### Architecture monolithique

![](<../../.gitbook/assets/Istio--Architecture Monolitique.png>)

### Architecture Microservices

![](<../../.gitbook/assets/Istio--Architecture Microservices.png>)

Les communications entre application dans ce type d'architecture présentent les problèmes suivants :&#x20;

* No encryption&#x20;
* No retries, no failover&#x20;
* No intelligent loadbalancing&#x20;
* No routing decisions&#x20;
* No metrics/logs/traces&#x20;
* No access control

Le système **Istio** permet de répondre à ces problématiques dans un cluster k8s (nous parlons également d'architecture "**Service Mesh**")

## Fonctionnement

Pour répondre aux différentes problématiques le système **Istio** utilise le système de **Sidecare**

{% hint style="info" %}
Les **sidecar** sont des conteneur qui fonctionnent dans un POD pour apporter des fonctionnalités supplémentaires au conteneur principale du POD
{% endhint %}

### Architecture avec Sidecar

![](<../../.gitbook/assets/Istio--Architecture with Sidecar.png>)

### Architecture Istio

#### Ingress Gateway

!!! Schema ingress

#### PODs et Sidecar

![](<../../.gitbook/assets/Istio--Architecture Istio.png>)

* Le conteneur utilisé comme sidecar par Istio est les proxy **Envoy**
* Le module **Pilot** permet de manager le comportement des sidecar
* Le module **Mixer** permet d'avoir de la supervision sur les sidecar
* Le module **Citadel** gère la partie TLS

## Sources
