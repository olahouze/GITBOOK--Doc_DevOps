---
description: Explication pour administrer un cluster EKS
---

# \[EKS] - Administration cluster

## Configuration AWS CLI

Pour installer AWS CLI : [https://docs.aws.amazon.com/fr\_fr/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/fr\_fr/cli/latest/userguide/getting-started-install.html)

Une fois installé, il faut configurer vos identifiants : [https://docs.aws.amazon.com/fr\_fr/cli/latest/userguide/getting-started-quickstart.html](https://docs.aws.amazon.com/fr\_fr/cli/latest/userguide/getting-started-quickstart.html)

{% hint style="info" %}
Pour récupérer vos identifiants (Access KEY) il faut les générer dans le service IAM : [https://console.aws.amazon.com/iamv2/home?#/home](https://console.aws.amazon.com/iamv2/home?#/home)

Une fois dans le service IAM, sélectionner l'utilisateur concerné pour générer les identifiants

![](<../.gitbook/assets/image (3).png>)
{% endhint %}

Vous pouvez ensuite utiliser la commande suivante pour configurer votre CLI : **aws configure**

Vous pouvez tester la configuration de votre CLI avec la commande **: aws sts get-caller-identity**

```
{
    "UserId": "AIDARVVQQPXED23TAC4XX",
    "Account": "1152605546XX",
    "Arn": "arn:aws:iam::1152605546XX:user/olahouze"
}

```

## Habilitations

Pour manager les clusters EKS vous devez disposer des droits suffisants à 2 niveaux

### Droits IAM

Vous devez disposer des droits nécessaires à vos actions sur le service EKS

**Exemple :**&#x20;

![](<../.gitbook/assets/image (2).png>)

### Droit EKS

Vous devez également autoriser l'identifiant IAM du compte dans la ConfigMap **aws-auth** sur le cluster : **kubectl -n kube-system describe configmap aws-auth**

<details>

<summary>aws-auth</summary>

```
Name:         aws-auth
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
mapAccounts:
----
[]

mapRoles:
----
- groups:
  - system:bootstrappers
  - system:nodes
  rolearn: arn:aws:iam::115260554xxx:role/aws-common-validation20201019084311502700000009
  username: system:node:{{EC2PrivateDNSName}}
- groups:
  - system:bootstrappers
  - system:nodes
  rolearn: arn:aws:iam::115260554xxx:role/aws-common-development20200714211143937300000005
  username: system:node:{{EC2PrivateDNSName}}

mapUsers:
----
- "groups":
  - "system:masters"
  "userarn": "arn:aws:iam::115260554xxx:user/ci-yyyy"
  "username": "ci-yyyy"
- "groups":
  - "system:masters"
  "userarn": "arn:aws:iam::115260554xxx:user/api-eks"
  "username": "api-eks"
- "groups":
  - "system:masters"
  "userarn": "arn:aws:iam::115260554xxx:user/toto"
  "username": "toto"

```

</details>

## Configuration Kubectl

Une fois toutes les habilitations OK vous pouvez générer le fichier de configuration vous permettant de se connecter au cluster : **aws eks --region \[region] update-kubeconfig --name \[cluster\_name]**
