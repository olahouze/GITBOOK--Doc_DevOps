# \[Concepts] - Liveness , Readiness and LifeCycle

## Description

Sur chaque worker node, le composant **Kubelet** est en charge de gérer les PODs qui tournent sur le node.

Pour cela, il a besoin de connaitre l'état de fonctionnement du POD avant de prendre des décisions

* redémarrer le POD
* attendre un peu
* démarrer un POD lorsque un autre POD a terminé son démarrage
* Ne plus envoyer de trafic client sur le POD
* etc...

Pour connaitre l’état du POD nous devons indiquer à Kubelet ce qu'il doit vérifier et le résultat attendu pour considérer l’état du POD comme étant correcte

Pour cela K8S utiliser les concepts suivants :

### **Liveness**

Ce test de santé permet à Kubelet d'identifier le moment ou le POD (et les conteneurs qu'il contient) est correctement démarrée et "**en vie**". Cela ne signifie pas forcement que le POD peut traiter les demandes clientes, simplement que les conteneurs à l’intérieur du POD sont bien tous démarré correctement

{% hint style="danger" %}
Lorsque le **Liveness** est KO, le Kubelet va **redémarrer** le conteneur
{% endhint %}

{% hint style="info" %}
Il est possible de mettre un délais dans le **Liveness** pour demander au Kubelet d'attendre un certain temps avant de réaliser le test de santé (cela permet de laisser le temps aux conteneurs de démarrer correctement)
{% endhint %}

### **Readiness** :

Ce test de santé permet à Kubelet d'identifier si le POD peut traiter les demandes clientes. Il est notamment utilisé pour extraire / rajouter un POD dans un service lorsque le service dispose de plusieurs POD pour traiter les demandes clients avec un système déséquilibrage de charge.

* Lorsque le Readiness est KO : Le POD quitte le service et ne reçoit plus les demandes clientes
* Lorsque le Readiness est OK : Le POD réintègre le service et recommence a traiter les demandes clientes

{% hint style="info" %}
Il est important de configurer un Readiness différent du Liveness car les actions que kubelet va mettre en place sont différente sur un **Liveness** KO (redémarrage du pod) sont différentes des actions sur un **Readiness** KO (sorti du service)
{% endhint %}

## Fonctionnement

![](<../../.gitbook/assets/K8S--lifecycle pod.png>)

* **Init container** : Ce conteneur réalise des actions pour préparer l'environnement du conteneur principale. Une fois les actions terminées il s’éteint généralement
* **Post start kock** : Action au démarrage du conteneur principale
* <mark style="color:red;">**Readiness/liveness**</mark> : Test de santé du Kubelet sur le POD (avec possible délais avant démarrage dans la configuration)
* **Pre stop hock** : Actions réalisées entre la demande de terminaison du POD et l'extinction de celui-ci

## Sources

{% embed url="https://kubernetes.io/fr/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes" %}
