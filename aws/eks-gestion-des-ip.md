---
description: Fonctionnement de la gestion des IP par EKS
---

# \[EKS] - Gestion des IP

Lors de la création d'un cluster EKS, chaque node est constitué une instance EC2

## ENI

Pour chacune des types d'instance EC2 nous avons un nombre maximum de ENI (interface réseau) et d'IP par ENI (définition : [https://docs.aws.amazon.com/fr\_fr/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI](https://docs.aws.amazon.com/fr\_fr/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI))

Dans les paramètres du CNI EKS (gestion du réseau) nous avons le paramétré _WARM\_ENI\_TARGET_ qui indique combien d'ENI doit être disponible en réserve (par défaut : WARM\_ENI\_TARGET=1)

**Exemple :**

* Nous avons 2 nodes&#x20;
* Chaque node est une instance _t3.2xlarge_ (4 ENI max avec 15 IP max/ENI)
* Chaque ENI utilise 1 IP de management = 14 IP utilisables
* Chaque node a le paramètre _WARM\_ENI\_TARGET=1_
* Chaque node a 1 interface ENI par défaut pour son fonctionnement nominale

Résultat : 2 node \* ((1 ENI Management \* 14 IP) + (1 ENI Spare \* 14 IP))= 60 IP

La formule :&#x20;

`X node * ((1 ENI Management * (IP Max - 1)) + (Y ENI Spare * (IP Max - 1))`

<mark style="color:red;">**Règle 1 : Une ENI doit etre disponible en réserve**</mark>

## IPAM

Le composant L-IPAM de AWS va automatiquement réserver dans le VPC les IP qui ont été attribuées aux ENI (qu'elles soient véritablement utilisées ou non par l'EKS)

Le CNI EKS a comme règle simple : avoir à disposition au minimum un nombre d'IP équivalente aux PODs en fonctionnement

<mark style="color:red;">**Règle 2 : Nous devons avoir autant d'IP disponibles que de POD sur le cluster**</mark>

## Fonctionnement

Lorsque le nombre de POD augmente et que nous arrivons à utiliser un nombre d'IP qui ne permet pas d'avoir une ENI de réserve ==> Le CNI va demander le rajout d'une ENI

Lorsque le nombre de POD diminue et que nous arrivons a avoir un nombre d'IP nécessaire qui peut être offert par moins d'ENI ==> Le CNI va supprimer les ENI supplémentaire en respectant la règle _WARM\_ENI\_TARGET_

## Exemple

Nous pouvons donc avoir une situation ou le cluster EKS utilise une Ingress GW mais consomme sur le VPC un nombre d'IP calculé avec le nombre d'ENI nécessaire pour offrir 1 IP pour chaque POD

**Exemple avec** : 1 Node avec avec 2 ENI de 15 IP = (15 \* 2 IP - 2 IP management) = 28 IP disponibles et 30 IP consommées sur le VPC

* Nous avons 10 POD
  * Les 28 IP sont suffisante et il reste bien 1 ENI de disponible
  * Nous consommons 30 IP dans le VPC
* Nous avons 15 POD
  * Les 28 IP disponibles sont suffisante **MAIS** nous utilisons les 14 IP de l'ENI 01 et la première IP de l'ENI 02 => La Règle 1 n'est plus respecté donc l'ajout d'une ENI 03 est effectué
  * Nous consommons 3 ENI de 15 IP = 45 IP sur le VPC

## Modification Possible

### Modifier les règles de ressources en resserve

Nous pouvons changer ce comportement en utilisant : **kubectl -n kube-system set env daemonset aws-node WARM\_IP\_TARGET=2**

Dans ce cas nous ne demandons plus d'avoir une ENI de disponible mais simplement 2 IP de réserve

**Exemple avec** : 1 Node avec avec 2 ENI de 15 IP = (15 \* 2 IP - 2 IP management) =  28 IP disponibles et 30 IP consommées sur le VPC

* Nous avons 10 POD
  * Les 28 IP sont suffisante et il reste bien 1 ENI de disponible
  * Nous consommons 30 IP dans le VPC
* Nous avons 26 POD
  * Les 28 IP disponibles sont suffisante et la regle WARM\_IP\_TARGET=2 est OK
  * Nous consommons 30 IP dans le VPC
* Nous avons 27 POD
  * Les 28 IP disponibles sont suffisante **MAIS** la regle WARM\_IP\_TARGET=2 est KO=> Donc l'ajout d'une ENI 03 est effectué
  * Nous consommons 3 ENI de 15 IP = 45 IP sur le VPC\


### Redéploiement nodes dans un autre VPC

Comme proposé ici : [https://aws.amazon.com/fr/blogs/containers/optimize-ip-addresses-usage-by-pods-in-your-amazon-eks-cluster/](https://aws.amazon.com/fr/blogs/containers/optimize-ip-addresses-usage-by-pods-in-your-amazon-eks-cluster/)

Une solution consiste à créer un VPC "large" (/8 ou /16) et de redéployer les worker dans ce VPC

La migration est supporté par EKS

\


## Bibliographie :&#x20;

[https://betterprogramming.pub/amazon-eks-is-eating-my-ips-e18ea057e045](https://betterprogramming.pub/amazon-eks-is-eating-my-ips-e18ea057e045)

[https://medium.com/compass-true-north/experiences-for-ip-addresses-shortage-on-eks-clusters-a740f56ac2f5](https://medium.com/compass-true-north/experiences-for-ip-addresses-shortage-on-eks-clusters-a740f56ac2f5)

[https://docs.aws.amazon.com/eks/latest/userguide/cni-increase-ip-addresses.html](https://docs.aws.amazon.com/eks/latest/userguide/cni-increase-ip-addresses.html)
