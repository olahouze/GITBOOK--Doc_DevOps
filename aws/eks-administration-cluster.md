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

<img src="../.gitbook/assets/AWS--IAM Access Key user.png" alt="" data-size="original">
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

### Gestion des profiles

Il est possible de créer plusieurs profils avec AWS CLI pour ne pas ressaisir a chaque fois les identifiants : **aws configure --profile \[name profil]**

Nous pouvons lister tous les profiles existants avec : **aws configure list-profiles**

{% hint style="info" %}
Lorsque vous avez creer les différents profiles vous pouvez en definir un par defaut avec : **export AWS\_DEFAULT\_PROFILE=\[name profil]**


{% endhint %}

## Habilitations

Pour manager les clusters EKS vous devez disposer des droits suffisants à 2 niveaux

### Droits IAM

Vous devez disposer des droits nécessaires à vos actions sur le service EKS

**Exemple :**&#x20;

![](<../.gitbook/assets/AWS--IAM Rights EKS.png>)

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

{% hint style="info" %}
Vous pouvez ajouter dans le fichier **kubeconfig** l'utilisation d'un profile AWS pour chaque cluster avec l'option **env:**

```
name: arn:aws:eks:eu-west-3:11526055XXXX:cluster/cluster
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      args:
      - --region
      - eu-west-3
      - eks
      - get-token
      - --cluster-name
      - aws-common-production
      command: aws
      env:
      - name: AWS_PROFILE
        value: profil1
      interactiveMode: IfAvailable
      provideClusterInfo: false
```
{% endhint %}

## **Changer de contexte Kubectl**&#x20;

Il est possible de configurer son kubectl avec plusieurs accès à différents clusters

Dans ce cas nous pouvons changer de contexte facilement de la manière suivante :&#x20;

1. Visualiser les différents contexte : **kubectl config get-contexts**

<details>

<summary>kubectl config get-contexts</summary>

```
CURRENT   NAME                                                                CLUSTER                                                             AUTHINFO                                                            NAMESPACE
*         arn:aws:eks:eu-west-3:115260554xxx:cluster/aws-common-development   arn:aws:eks:eu-west-3:115260554xxx:cluster/aws-common-development   arn:aws:eks:eu-west-3:115260554xxx:cluster/aws-common-development   
          arn:aws:eks:eu-west-3:115260554xxx:cluster/aws-common-validation    arn:aws:eks:eu-west-3:115260554xxx:cluster/aws-common-validation    arn:aws:eks:eu-west-3:115260554xxx:cluster/aws-common-validation
```

</details>

2\. Pour changer de contexte : **kubectl config use-context \[context]**
