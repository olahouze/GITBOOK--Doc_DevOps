# MutualTLS

## Description

La fonction de **MutualTLS** proposée par Istio permet de chiffrer les communications entre chaque PODs du **Service Mesh** en utilisant les technologies TLS

{% hint style="info" %}
Cette fonctionnalité permet d'activer le chiffrement des communications entre application **SANS** modification de code au niveau de l'application

Le code applicatif n'a pas besoin de prendre en compte cette sécurité car elle est entièrement gérée par les sidecar de l'architecture Istio pour tous les PODs
{% endhint %}

## Fonctionnement

![](<../../.gitbook/assets/Istio--Architecture Istio.png>)

Il s'agit du composant **Citadel** qui va s'occuper de :

* Générer les certificats pour le TLS pour chaque sidecar
* Renouveler les certificats régulièrement

## Exemple

Pour activer le MutualTLS sur un cluster avec Istio

### Configuration de la fonctionnalité

```
apiVersion: authentication.istio.io/v1alpha1
kind: "MeshPolicy"
metadata:
  name: "default"
spec:
  peers:
  - mtls: {}
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: "enable-mtls"
  namespace: "default" # even though we specify a namespace, this rule applies to all namespaces
spec:
  host: "*.local"
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
 name: "api-server"
spec:
 host: "kubernetes.default.svc.cluster.local"
 trafficPolicy:
   tls:
     mode: DISABLE
```

1. Nous créons le CRD de type "**MeshPolicy**" pour renseigner les paramètres de configuration à utiliser pour le mutualTLS
2. Nous créons une **DestinationRule** pour identifier sur quel élément du cluster nous voulons appliquer le mutualTLS (dans notre cas nous choisissons tous le cluster avec la configuration **\*.local**)
3. Nous <mark style="color:red;">**désactivons**</mark> le mutualTLS pour le namespace "**kubernetes**" qui contient l'api-server (_name: "api-server"_). Ces éléments n'ayant pas été déployé avec Istio ils ne disposent pas des sidecar. Si nous activions le mutualTLS sur ces éléments nous ne pourrions plus communiquer avec eux.

### Création de l'application

Nous déployons classiquement l'application

```
apiVersion: v1
kind: Namespace
metadata:
  name: ns1
---
apiVersion: v1
kind: Namespace
metadata:
  name: ns2
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-tls
  namespace: ns1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        version: v1-tls
    spec:
      containers:
      - name: hello
        image: wardviaene/http-echo
        env:
        - name: TEXT
          value: hello
        - name: NEXT
          value: "world.ns2:8080"
        ports:
        - name: http
          containerPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: world-tls
  namespace: ns2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: world
  template:
    metadata:
      labels:
        app: world
        version: v1-tls
    spec:
      containers:
      - name: hello
        image: wardviaene/http-echo
        env:
        - name: TEXT
          value: world
        - name: NEXT
          value: "end.legacy:8080"
        ports:
        - name: http
          containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: hello
  namespace: ns1
  labels:
    app: hello
spec:
  selector:
    app: hello
  ports:
  - name: http
    port: 8080
    targetPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: world
  namespace: ns2
  labels:
    app: world
spec:
  selector:
    app: world
  ports:
  - name: http
    port: 8080
    targetPort: 8080
```

1. Nous créons les namespace **ns1** et **ns2**
2. Nous utilisons un **Deployment** pour déployer un POD dans chaque Namespace
3. Nous créons un service pour chaque POD

### Configuration de Istio

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: helloworld-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: hello
spec:
  host: hello.ns1.svc.cluster.local
  # uncomment to enable mutual TLS
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
  subsets:
  - name: v1-tls
    labels:
      version: v1-tls
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: helloworld-tls
spec:
  hosts:
  - "hello-tls.example.com"
  gateways:
  - helloworld-gateway
  http:
  - route: 
    - destination:
        host: hello.ns1.svc.cluster.local
        subset: v1-tls # match v3 only
        port:
          number: 8080
```

1. Nous créons l'**Ingress Gateway** qui va recevoir le traffic (_name: helloworld-gatewa_y)
2. Nous créons la **DetinationRule** qui va renvoyer vers le service (_name: hello_). C'est cet objet qui va activer l'utilisation du mutualTLS avec les paramètres de **trafficPolicy**
3. Nous créons le **VirtualService** (_name: helloworld-tls_) pour lier l'**Ingress Gateway** et la **DestinationRule** en fonction de paramètres (ici la route par défaut donc pas de paramètres spécifiques)

## Sources
