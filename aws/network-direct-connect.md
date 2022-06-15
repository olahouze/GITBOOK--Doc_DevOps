---
description: Explication du fonctionnement du service Direct Connect
---

# \[Network] - Direct Connect

Ce service permet de créer un lien direct entre notre compte AWS et un FAI

## Fonctionnement

Pour cela, nous devons réaliser les objets suivants

1. La connexion du partenaire arrive sur une "**Connection**" du service Direct Connect
2.  Sur cette connexion nous devons réaliser des "**Virtual Interfaces**" pour pouvoir l'utiliser

    Dans cette "Virtual Interface" nous choisissons si elle est Privée / Publique / Transit
3. Nous devons ensuite créer la "**Direct Connect Gateway**". Cette objet permettra de faire l'association : **Virtual Interface <=> Transit Gateway / Virtual Private Gateway**

### Avec Virtual Private Gateway

![](<../.gitbook/assets/AWS--Direct Connect-01.png>)

### Avec Transit Gateway

![](<../.gitbook/assets/AWS--Transite Gateway-01.png>)

## Sources : &#x20;

{% embed url="https://docs.aws.amazon.com/directconnect/latest/UserGuide/getting_started.html" %}

{% embed url="https://docs.aws.amazon.com/directconnect/latest/UserGuide/direct-connect-gateways-intro.html" %}
