---
description: Interconnexion entre K8S et Vault pour recuperer des secrets
---

# \[Configuration] - K8S + Vault - Utilisation Vault-Injector

Il est possible d'utiliser vault en mode "**injecteur**" pour récupérer des secrets à l'aide d'un side-car : [https://www.vaultproject.io/docs/platform/k8s/injector](https://www.vaultproject.io/docs/platform/k8s/injector)

Le chart d’installation officiel est ici : [https://www.vaultproject.io/docs/platform/k8s/helm](https://www.vaultproject.io/docs/platform/k8s/helm)

## Pre requis <a href="#vault+k8s-prerequis" id="vault+k8s-prerequis"></a>

Pour que le système fonctionne il faut que :

* **vault-agent-injector** soit installé sur le cluster : [https://www.vaultproject.io/docs/platform/k8s/injector/installation](https://www.vaultproject.io/docs/platform/k8s/injector/installation)
* Vous disposiez d'un service Vault fonctionnel
* Que le service Vault puisse **recevoir** des flux réseau depuis les clusters qui veulent utiliser ce System (sur le protocole https pour utiliser l'API Vault)
* Que le service Vault puisse **accéder** à l'API des clusters qui veulent utiliser ce syteme (sur le protocole https pour utiliser l'API K8S)

## Fonctionnement <a href="#vault+k8s-fonctionnement" id="vault+k8s-fonctionnement"></a>

Voici l'explication du fonctionnement :&#x20;

![](<../.gitbook/assets/image (6).png>)

1. Le POD de l'application va demander à l'API Vault des acces aux secrets (en pressentant un jeton JWT associé au service account utilisé dans le POD)
2. Le service Vault demande vérification du JWT à l'API K8S du cluster (pour récupérer le namespace du service account et vérifier que le jeton JWT est le bon)
3. SI OK, le serveur Vault va vérifier les habilitations qui sont attribuées à ce service account
4. Si habilitation OK, le Vault renvoi les secret au POD

Pour être plus précis, dans le POD il y a un **sidecar** qui réalise ces actions (le vault-agent-injector) qui fonctionne comme cela :

![](<../.gitbook/assets/image (2) (1).png>)

\
C'est le sidecar lors de l'initialisation qui va récupérer les secrets dans Vault et peupler le **/vault/secrets** du POD

## Configuration <a href="#vault+k8s-configuration" id="vault+k8s-configuration"></a>

### Installation de VAULT sur cluster principal <a href="#vault+k8s-installationdevaultsurclusterprincipal" id="vault+k8s-installationdevaultsurclusterprincipal"></a>

Pour cela vous pouvez utiliser le helm chart de l'entreprise qui développe la solution pour installer un Vault sur un cluster K8S : [https://www.vaultproject.io/docs/platform/k8s/helm](https://www.vaultproject.io/docs/platform/k8s/helm)

**Note** : Il est recommander de créer un namespace spécifique a ce projet et d'installer le helm dans ce namespace (exemple : **kubectl create namespace vault**)

Lorsque l'installation est terminée, vous devez avoir ceci sur le cluster (dans le namespace Vault)

```
NAME                                        READY   STATUS    RESTARTS   AGE
pod/vault-0                                 1/1     Running   0          37d
pod/vault-agent-injector-65478f8d4f-kx6r8   1/1     Running   0          45d
 
NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/vault                      ClusterIP   172.20.79.247   <none>        8200/TCP,8201/TCP   125d
service/vault-agent-injector-svc   ClusterIP   172.20.8.143    <none>        443/TCP             602d
service/vault-internal             ClusterIP   None            <none>        8200/TCP,8201/TCP   125d
 
NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/vault-agent-injector   1/1     1            1           602d
 
NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/vault-agent-injector-65478f8d4f   1         1         1       125d
 
NAME                     READY   AGE
statefulset.apps/vault   1/1     125d
```

### Création des services account <a href="#vault+k8s-creationdesservicesaccount" id="vault+k8s-creationdesservicesaccount"></a>

#### Service account utilisateur <a href="#vault+k8s-serviceaccountutilisateur" id="vault+k8s-serviceaccountutilisateur"></a>

Pour fonctionner le système a besoin de service account qui vont interroger le service Vault (sur lesquels nous attribuerons des droits)

La bonne pratique est de créer des services account par namespaces afin que les utilisateurs d'un namespace puisse l'utiliser sans demander d’extension de droit

Voici la définition de la création d'un service account "**test-sa**" dans le namespace "**test**" avec tous les éléments nécessaire au fonctionnement du Vault injector (clusterrolebinding, etc...)

<details>

<summary><strong>Service account utilisateur</strong></summary>

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
 name: test-sa
 namespace: test
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: test-sa-cluster-role
  namespace: test
rules:
- apiGroups:
  - admissionregistration.k8s.io
  resources:
  - mutatingwebhookconfigurations
  verbs:
  - get
  - list
  - watch
  - patch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
 name: test-sa-cluster-role-binding
 namespace: test
roleRef:
 apiGroup: rbac.authorization.k8s.io
 kind: ClusterRole
 name: test-sa-cluster-role
subjects:
 - kind: ServiceAccount
   name: test-sa
   namespace: test
```

</details>

#### Service account Technique <a href="#vault+k8s-serviceaccounttechnique" id="vault+k8s-serviceaccounttechnique"></a>

Pour fonctionner le système a besoin d'un service account qui va permettre a Vault de vérifier les demandes qu'il recoit en interrogeant l'PAI du cluster

La bonne pratique est de créer ce service account dans le namespace du projet contenant le "vault-injector"

Voici le yaml pour configurer le service account **vault-injector-dev**

<details>

<summary><strong>Service account Technique</strong></summary>

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
 name: vault-injector-dev
 namespace: vault
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: vault-injector-dev-cluster-role
  namespace: vault
rules:
- apiGroups:
  - authentication.k8s.io
  resources:
  - tokenreviews
  verbs:
  - create
- apiGroups:
  - authorization.k8s.io
  resources:
  - subjectaccessreviews
  verbs:
  - create
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
 name: vault-injector-dev-cluster-role-binding
 namespace: vault
roleRef:
 apiGroup: rbac.authorization.k8s.io
 kind: ClusterRole
 name: vault-injector-dev-cluster-role
subjects:
 - kind: ServiceAccount
   name: vault-injector-dev
   namespace: vault
```

</details>

### Installation de Vault-Injector sur les clusters <a href="#vault+k8s-installationdevault-injectorsurlesclusters" id="vault+k8s-installationdevault-injectorsurlesclusters"></a>

Il faut ensuite installer la partie "**vault-injector**" sur les clusters qui hébergent des pods qui vont vouloir accéder au Vault pour agir sur des secrets

Pour cela nous utilisons le même helm chart que nous avons utilisé pour installer le Vault sur le premier cluster

Ce helm va installer les deux éléments, le serveur vault et le composant vault-injector, et nous n'utiliseront que le **vault-injector** sur les clusters qui veulent accéder au serveur Vault installé sur le premier cluster (il est possible de supprimer les pods "vault" sur ces clusters car nous ne les utiliseront pas)

**Note** : Vous pouvez vérifier que le cluster dispose bien des pods qui correspondent **:**&#x20;

```
NAME                                        READY   STATUS    RESTARTS   AGE
pod/vault-agent-injector-65478f8d4f-kx6r8   1/1     Running   0          45d
 
NAME                               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/vault-agent-injector-svc   ClusterIP   172.20.8.143    <none>        443/TCP             602d
service/vault-internal             ClusterIP   None            <none>        8200/TCP,8201/TCP   125d
 
NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/vault-agent-injector   1/1     1            1           602d
 
NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/vault-agent-injector-65478f8d4f   1         1         1       125d
```

### Configuration du système d'authentification K8S <a href="#vault+k8s-configurationdusystemedauthentificationk8s" id="vault+k8s-configurationdusystemedauthentificationk8s"></a>

Il est possible de configurer Vault pour donner des droits sur un ensemble **cluster + account names + namespaces** pour agir sur le Vault

Pour cela nous devons activer la méthode d'authentification **kubernetes** dans les configurations du Vault

#### Pre requis : <a href="#vault+k8s-prerequis" id="vault+k8s-prerequis"></a>

<mark style="color:purple;">**Nomenclature :**</mark>&#x20;

Pour simplifier la maintenance on suivra le schema de nommage

* Politique d'acces : "**kubernetes\_\<provider>\_\<projet>\_\<cluster-name-court>\_\<service\_account\_technique>**".
* Nom des rôles de la politique : "**\<namespace>\_\<service account>**".

_Par exemple:_ pour une application utilisant

* Le compte utilisateur "test-sa"
* Le compte technique "vault-injector-dev"
* Dans le namespace "test"
* Sur le cluser "Common" de "Développement"
* Chez AWS \


Cela nous donne :&#x20;

* Politique : **kubernetes\_aws\_common\_dev\_vault-injector-dev**
* Role **: test\_test-sa**

<mark style="color:purple;">**Informations de configuration**</mark>

Pour la configuration de la méthode K8S sur Vault nous avons besoin des informations suivante :

1. L'URL de l'API du cluster qui va interroger le Vault. (sur AWS on peut le trouver ici ) :  <img src="../.gitbook/assets/image (4) (1).png" alt="" data-size="original">
2.  Le token JWT du service account créé pour le besoin de vérification des jeton (dans l'exemple : **vault-injector-dev**) :&#x20;

    1. Récupérer le nom de l'instanciation du service account : **export **<mark style="color:blue;">**SA\_NAME**</mark>**=$(kubectl -n vault get sa vault-injector-dev -o jsonpath="{.secrets\[\*]\['name']}")**
    2. Récupérer le token du service account : **export **<mark style="color:blue;">**SA\_TOKEN**</mark>**=$(kubectl -n vault get secret **<mark style="color:blue;">**$SA\_NAME**</mark>** -o jsonpath="{.data.token}" | base64 --decode; echo)**

    ****
3. Le Certificat du service account créé pour ce besoin : <mark style="color:blue;">**SA\_CERT**</mark>**=$(kubectl -n vault get secret **<mark style="color:blue;">**$SA\_NAME**</mark>** -o jsonpath="{.data\['ca\\.crt']}" | base64 --decode; echo)**

{% hint style="danger" %}
Il existe un problème de mise en forme de certificat avec cette méthode : [https://stackoverflow.com/questions/14560393/ssl-certificate-to-json/14580203#14580203](https://stackoverflow.com/questions/14560393/ssl-certificate-to-json/14580203#14580203)

Il faut donc stocker dans un fichier le résultat du certificat (celui de la variable <mark style="color:blue;">**SA\_CERT)**</mark>** ** et remplacer les espaces entre les entête et le payload par des retours a la ligne

**Avant** : `-----BEGIN CERT----- XXXXXX -----END CERTIFICATE-----`

**`Apres`**`:`&#x20;

`-----BEGIN CERT-----`

`XXXXXX`

`-----END CERTIFICATE-----`
{% endhint %}

#### Configuration <a href="#vault+k8s-configuration.1" id="vault+k8s-configuration.1"></a>

Pour créer le chemin d'authentification, il est possible d'utiliser l'interface de vault ou la cli.

**GUI** : Access → Enable new methode → choisir Kubertenes →**changer le path** (chemin d'authentification) en utilisant la règle de nommage de Politique d'acces → <mark style="color:red;">Renseigner les paramètres</mark> → Enable method

**CLI** : [https://www.vaultproject.io/docs/auth/kubernetes#configuration](https://www.vaultproject.io/docs/auth/kubernetes#configuration)

```
vault auth enable kubernetes_aws_common_dev_vault-injector-dev
vault write auth/kubernetes_aws_common_dev_vault-injector-dev/config \
    token_reviewer_jwt="$SA_TOKEN" \
    kubernetes_host=https://<URL API K8S>:<your TCP port or blank for 443> \
    kubernetes_ca_cert=$SA_CERT

```

Dans vault sur la méthode authentification il faut configurer les champs suivants:

* **Kubernetes host** : url de l'api du cluster kubernetes qui demande les secrets
* **Kubernetes CA Certificate** : partie publique du certificat du service account (préférer l'upload d'un fichier à la partie texte) que vous avez récupéré précédemment dans <mark style="color:blue;">**$SA\_CERT**</mark>
* **JWT Issuer:**  pas nécessaire
* **Token Reviewer JWT:** partie privé (token) du service account que vous avez récupéré précédemment avec <mark style="color:blue;">**$SA\_TOKEN**</mark>

### Création des rôles <a href="#vault+k8s-creationdesroles" id="vault+k8s-creationdesroles"></a>

Nous pouvons maintenant créer les roles sur le Vault qui permettent d'associer des policy aux services account qui se présentent sur l'API Vault

Nous créons le role **test\_test-sa** en appliquant la politique précédemment créée (dans notre exemple : **read\_olny**)

Renseigner les champs suivants:

* **Bound service account names** : nom du service account
* **Bound service account namespaces**: namespace autorisé à manipuler les secrets
* **Generated Token's Policies**: policy vault à appliquer
* **Do Not Attach 'default' Policy To Generated Tokens**: cocher cette case pour ne pas ajouter la policy default (paramètre avancé du token)\


**GUI** : Access → Editer la méthode Kubertenes correspondant→ onglet Role → Create Role → Renseigner les parametres → Save

![](<../.gitbook/assets/image (1).png>)

![](<../.gitbook/assets/image (3).png>)

**CLI** :

```
vault write auth/kubernetes_aws_common_dev_vault-injector-dev/role/test_test-sa \
        bound_service_account_names=test-sa \
        bound_service_account_namespaces=test \
        policies=read_only\
        ttl=24h
```

### Adaptation de la configuration Vault-Injector <a href="#vault+k8s-adaptationdelaconfigurationvault-injector" id="vault+k8s-adaptationdelaconfigurationvault-injector"></a>

Par défaut, le vault-injector est configuré pour accéder au serveur vault déployé sur le même cluster par le même helm

Nous avons donc besoin de modifier le déploiement du vault-injector pour lui permettre d’accéder au Vault situé à l’extérieur du cluster

Dans le <mark style="color:red;">**déploiement**</mark> du vault-injector (ici : **deployment.apps/vault-agent-injector**) il faut modifier :

* **AGENT\_INJECT\_VAULT\_ADDR** : Mettre l'URL du Vault Externe
* **AGENT\_INJECT\_VAULT\_AUTH\_PATH** : Mettre le chemin de la méthode d'authentification (ici : **kubernetes\_aws\_common\_dev\_vault-injector-dev)**

## Utilisation <a href="#vault+k8s-utilisation" id="vault+k8s-utilisation"></a>

Dans le POD qui a besoin de récupérer les secrets il faut rajouter les éléments suivants :

```
  annotations:
    vault.hashicorp.com/agent-inject: 'true'
    vault.hashicorp.com/role: '<chemin méthode d'authentification>'
    vault.hashicorp.com/agent-inject-secret-<name fichier>: '<chemin dans Vault>'
spec
  serviceAccountName: test-sa
```

## Exemples

Voici un exemple de déploiement pour récupérer des carentiels dans un POD en interrogeant Vault

Nous utilisons le service account **test-sa** qui a ete autorisé dans les roles de la méthode d'authentification K8S sur Vault

Nous récupérons les éléments de la branche **lizeo-europe/project/awx/gitlab**

Nous stockons les éléments dans le fichier **config.txt** (stockés dans /vault/secrets/)

<details>

<summary><strong>Pod test récupération Vault - Simple</strong></summary>

```
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  namespace: test
  annotations:
    vault.hashicorp.com/agent-inject: 'true'
    vault.hashicorp.com/role: 'test_test-sa'
    vault.hashicorp.com/agent-inject-secret-config.txt: 'lizeo-europe/project/awx/gitlab'     
spec:
  containers:
  - image: nginx:latest
    name: nginx
  serviceAccountName: test-sa
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: eks.amazonaws.com/nodegroup
            operator: In
            values:
            - common-spot
            - common-on-d
  tolerations:
    - effect: NoSchedule
      key: dedicated
      operator: Equal
      value: common
```

</details>

**Le résultat est le suivant :**

```
kubectl -n test exec -it webapp -- sh
Defaulted container "nginx" out of: nginx, vault-agent, vault-agent-init (init)
# cat /vault/secrets/config.txt
data: map[ansible-awx:85YdZLCTaymR4XXXXXXX_]
metadata: map[created_time:2020-07-06T19:09:43.412598839Z deletion_time: destroyed:false version:1]
#
```

Nous pouvons récupérer différents secrets dans différents dossiers :

| **Fichier** | **Élément**                          |
| ----------- | ------------------------------------ |
| config.txt  | lizeo-europe/project/awx/gitlab      |
| test.txt    | lizeo-europe/project/awx/development |

<details>

<summary><strong>Pod test récupération Vault - Multiple Fichier</strong></summary>

```

apiVersion: v1
kind: Pod
metadata:
  name: webapp
  namespace: test
  annotations:
    vault.hashicorp.com/agent-inject: 'true'
    vault.hashicorp.com/role: 'test_test-sa'
    vault.hashicorp.com/agent-inject-secret-config.txt: 'lizeo-europe/project/awx/gitlab'
    vault.hashicorp.com/agent-inject-secret-test.txt: 'lizeo-europe/project/awx/development'      
spec:
  containers:
  - image: nginx:latest
    name: nginx
  serviceAccountName: test-sa
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: eks.amazonaws.com/nodegroup
            operator: In
            values:
            - common-spot
            - common-on-d
  tolerations:
    - effect: NoSchedule
      key: dedicated
      operator: Equal
      value: common
```

</details>

**Résultat :**

```
kubectl -n test exec -it webapp -- sh
Defaulted container "nginx" out of: nginx, vault-agent, vault-agent-init (init)
# cat /vault/secrets/config.txt
data: map[ansible-awx:85YdZLCTaymR4rxXXXX_]
metadata: map[created_time:2020-07-06T19:09:43.412598839Z deletion_time: destroyed:false version:1]
# cat /vault/secrets/test.txt
data: map[admin-password:quuQuoDaen6EiY0ahXXX postgresql-password:hlXXXXX secret-key:aXXXX ssh-ansible-passphrase:rr2SXXXX ssh-ansible-private-key:-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABAIrERc6w
ky19YI/zb4eh2pAAAAEAAAAAEAAAEXAAAAB3NzaC1yc2EAAAADAQABAAABAQCwscPE1cTx
X9XY5/nTc++YHK20r+1r/hECzS4FLGOPzgx0uqafLy7lyiCPHPa/rVbNkXQ9qVi1avd2y6
.....
Yiu7ZFjnEMZO6lX/q/hKjNPgyxQvN2SxFjf4dykh+g2jFOWAe+7yTxsYBJg/rbrbS84pSs
WxMYqWL99T+t5EQj4ofy2ozK2XMAMCKvvYYrU4oTEcCI7IJZ0IlA33kvBhlaPuGwxO4VpS
/ziye47ByW9iZzN3pzKm5dRvLKXE7of9W8YmWv4SiddrZNmoL2RsEvY39HypJxR8A4qoJL
HDukqVIFYMQRGn+bpRwpgwR7R6ick=
-----END OPENSSH PRIVATE KEY----- ssh-ansible-public-key:ssh-rsa AAAAB3NzaC1...fZU2fOdzlF57PT0YjaQGRdPEiiAq17vyVQxKZiXq ansible@awx-dev]
metadata: map[created_time:2021-09-03T13:06:08.038424291Z deletion_time: destroyed:false version:5]
```

<details>

<summary>Pod test récupération Vault - Template (fichier test.txt)</summary>

```
---
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  namespace: test
  annotations:
    vault.hashicorp.com/agent-inject: 'true'
    vault.hashicorp.com/role: 'test_test-sa'
    vault.hashicorp.com/agent-inject-secret-config.txt: 'lizeo-europe/project/awx/gitlab'      
    vault.hashicorp.com/agent-inject-secret-test.txt: 'lizeo-europe/project/awx/validation'  
    vault.hashicorp.com/agent-inject-template-test.txt: |
          {{ with secret "lizeo-europe/project/awx/validation" }}
          [DEFAULT]
          LogLevel = DEBUG
          [DATABASE]
          Address=127.0.0.1
          Port=3306
          User={{ .Data.data.login }}
          Password={{ .Data.data.password }}
          Database=app
          {{ end }}
spec:
  containers:
  - image: nginx:latest
    name: nginx
  serviceAccountName: test-sa
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: eks.amazonaws.com/nodegroup
            operator: In
            values:
            - common-spot
            - common-on-d
  tolerations:
    - effect: NoSchedule
      key: dedicated
      operator: Equal
      value: common

```

</details>

**Résultat**

```
kubectl -n test exec -it webapp -- sh
Defaulted container "nginx" out of: nginx, vault-agent, vault-agent-init (init)
# cat /vault/secrets/test.txt

[DEFAULT]
LogLevel = DEBUG
[DATABASE]
Address=127.0.0.1
Port=3306
User=adm
Password=admin_xyze
Database=app

```
