---
description: Explication du fonctionnement des subnet dans EKS
---

# \[EKS] - Gestion des subnets

## Architecture

Les clusters AWS respectent l'architecture suivante : [https://docs.aws.amazon.com/eks/latest/userguide/eks-networking.html](https://docs.aws.amazon.com/eks/latest/userguide/eks-networking.html)

En résumé nous avons :&#x20;

* Les nodes Masters K8S managé par AWS (non visible sur notre compte) qui exposent l'API de management
* Un choix de subnet pour exposer sur des ENI l'acces à l'API qui sera utilisée pour la communication Worker <=> Master
* Un choix de subnet pour deployer les worker

## API de Management

Cette API est exposée par les Master sur une URL que nous pouvons visualiser dans : **EKS > Clusters > \[nom cluster] > Configuration > Details**&#x20;

![](../.gitbook/assets/01.png)

Subnet&#x20;

## Sources

{% embed url="https://docs.aws.amazon.com/eks/latest/userguide/eks-networking.html" %}

{% embed url="https://docs.aws.amazon.com/eks/latest/userguide/network_reqs.html" %}

