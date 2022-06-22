# Présentation

## Description

Le développement d'application est passé d'un mode **monolithique** à un mode **microservices**

Les **microservices** désignent une approche architecturale du développement d'applications.&#x20;

* En tant que cadre architectural, les microservices sont distribués et faiblement couplés, de sorte que les modifications apportées par une équipe n'affectent pas l'ensemble de l'application.&#x20;
* L'avantage des microservices est que les équipes de développement sont en mesure de créer rapidement de nouveaux composants d'applications pour répondre aux évolutions des besoins de l'entreprise

### Architecture Monolithique

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

#### Control Plane

![](<../../.gitbook/assets/Istio--Architecture Istio.png>)

* Le conteneur utilisé comme sidecar par Istio est les proxy **Envoy**
* Le module **Pilot** permet de manager le comportement des sidecar
* Le module **Mixer** permet d'avoir de la supervision sur les sidecar
* Le module **Citadel** gère la partie TLS

#### Data Plane

![](<../../.gitbook/assets/Istio--Architecture with all component.png>)

Les flux utilisateurs entrants passent par les éléments suivants dans une architecture avec Istio

* Une **Ingress** **Gateway** qui sert de point d'entré (il peut exister plusieurs ingress gateway pour séparer les flux entrant en fonction de certaines contraintes)
* Des **VirtualService** qui servent à router le flux en fonction de règles spécifique (hostname, value dans le header, etc...). Un VirtualService est **toujours associé à une Gateway** et renvoi vers un **Service** ou une **DestinationRule**
* Des **DestinationRules** qui permettent d'identifier les PODs vers lesquels envoyer le flux. Ces DestinationRules remplacent les objets "Service" classique dans K8S pour apporter plus de possibilité dans l'utilisation de Istio (il s'agit d'un CRD Istio)

## Sources
