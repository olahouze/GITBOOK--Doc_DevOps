---
description: Explication du fonctionnement des subnet dans EKS
---

# \[EKS] - !!! Gestion des subnets

## Architecture

Les clusters AWS respectent l'architecture suivante : [https://docs.aws.amazon.com/eks/latest/userguide/eks-networking.html](https://docs.aws.amazon.com/eks/latest/userguide/eks-networking.html)

En résumé nous avons :&#x20;

1.  Les nodes Masters K8S managés par AWS (non visible sur notre compte) qui exposent l'API de management selon différent mode (configurable dans les paramétrages du cluster EKS)

    * **API Publique** : Accès des utilisateurs et des workers nodes **uniquement** par Internet sur des IP Publiques
    * **API Privée** : Access des utilisateurs et des workers nodes **uniquement** possible par les ENI qui sont déployées sur des réseaux du VPC
    * **API Publique et Privé** : les 2 accès sont possibles, depuis internet ou à travers les ENI Privée


2. Un choix de subnet pour exposer sur des ENI l’accès à l'API de management en mode **Privé**
3. Un choix de subnet pour déployer les worker nodes

### API Publique

Si le choix est fait d'utiliser l'API **Publique :**&#x20;

<mark style="color:blue;">**Worker**</mark>: les communications Worker <=> Master devront passer par internet. Cela implique que les réseaux des workers devront avoir accès à Internet&#x20;

* Soit un réseau publique avec une route par defaut vers une Internet GW
* Soit un réseau privée avec l'utilisation d'une NAT GW associée à une Internet GW

<mark style="color:blue;">**Administrateur**</mark> : Les accès de management des administrateurs passeront par Internet sur des IP Publiques AWS

### API Privé

Si le choix est fait d'utiliser l'API **Privée :**&#x20;

<mark style="color:blue;">**Worker**</mark>: les communications Worker <=> Master passeront par les ENI déployées dans le VPC

<mark style="color:blue;">**Administrateur**</mark> : Les accès de management des administrateurs passeront également par les ENI (ce qui implique d'avoir une connexion initiée depuis une IP dans le VPC

## API de Management

Cette API est exposée par les Master sur une URL que nous pouvons visualiser dans : **EKS > Clusters > \[nom cluster] > Configuration > Details**&#x20;

![](../.gitbook/assets/01.png)

Subnet&#x20;

## Sources

{% embed url="https://docs.aws.amazon.com/eks/latest/userguide/eks-networking.html" %}

{% embed url="https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html" %}

