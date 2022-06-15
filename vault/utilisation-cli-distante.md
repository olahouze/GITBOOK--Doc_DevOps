---
description: Configuration pour utiliser la CLI Vault depuis un poste distant
---

# Utilisation CLI Distante

Il est possible d'utiliser la CLI de Vault en dehors d'un nœud du cluster, directement sur son poste

Pour cela

1. Installer le binaire Vault sur son poste : [https://learn.hashicorp.com/tutorials/vault/getting-started-install#install-vault](https://learn.hashicorp.com/tutorials/vault/getting-started-install#install-vault)
2. Exporter la variable d'environnement permettant d’accéder au Vault concerné. (exemple : **VAULT\_ADDR="https://vault.mycompagny.net**")
3. Récupérer votre Token d’authentification depuis la GUI de Vault une fois authentifié avec votre méthode

![Recuperation du Token utilisateur](<../.gitbook/assets/VAULT-- Copy token.png>)

4\. Utiliser la méthode **vault login** (donner votre token dans la CLI pour se connecter)



Vous pouvez maintenant utiliser la CLI Vault depuis votre poste

```
vault list auth/jwt/role/
Keys
----
airflow-deploy
bd4m-deploy
k8s-ci-deploy
talend-deploy
tyreupload-pirelli-deploy
velero-deploy

```
