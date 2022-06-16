# \[Concepts] - Service

## Description

Les services permettent d'exposer sur <mark style="color:red;">**une IP et un port fixe**</mark> l’accès a des PODs

Il existe différents type d'exposition

1. **Cluster IP** : Permet d'exposer des services uniquement pour les POD à l'interrieur du cluster (type par defaut)
2. **Node Port** : Permet d'exposer sur tous les nodes un port et une IP fixe pour acceder au service (non recommendé)
3. **Loadbalanceur** : Permet de demander au Provider la création d'un Load Balanceur qui va donner l’accès au service au monde extérieur (création automatique des éléments "cluster IP" + "Node Port")

{% hint style="info" %}
La methode de redirection des flux entre le LoadBalanceur et le service est différent selon les Providers : [https://www.stackrox.io/blog/kubernetes-networking-demystified/](https://www.stackrox.io/blog/kubernetes-networking-demystified/)
{% endhint %}

## Fonctionnement

Les différentes étapes lors de l'exposition d'un service

1. L'utilisateur demande la création d'un service en spécifiant
   1. Port d'exposition du service
   2. Port d’écoute des POD (optionnel)
   3. IP d'exposition du service (optionnel)
2. Le cluster va créer le service et lui associé l'IP adéquat (Virtual IP)
3. Le cluster va créer les différents **endpoints** en fonction du nombre de POD concernés par le service (Endpoint = IP POD + port d'ecoute du POD)
4. Le **kube-proxy** de chaque node va agir sur les règles **iptables** pour permettre à chaque node de
   1. Traiter le trafic à destination de l'IP du service (Virtual IP)
   2. Renvoyer le paquet sur un node qui héberge un POD concerné par le service
5. (option si type = Loadbalancer) : Demande au provider de créer le loadbalanceur pour rediriger le flux vers les nodes sur les bons ports

### Cluster IP

![](<../../.gitbook/assets/K8S--Network K8S-Cluster IP.png>)

### LoadBalancer

![](<../../.gitbook/assets/K8S--Network K8S-LoadBalancer.png>)

## Exemple

Les règles IPtables pour chaque service (ip virtuelle + port) sont créées sur chaque nodes ([https://guillaume.fenollar.fr/blog/kubernetes-kube-proxy-iptables/](https://guillaume.fenollar.fr/blog/kubernetes-kube-proxy-iptables/))

### Définition des services

Voici un exemple de service créé sur un cluster (IP d'exposition : 192.168.18.202)

```
spec:
  clusterIP: 192.168.18.202
  ports:
  - port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
```

Cela donne sur le cluster

```
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes-dashboard   ClusterIP   192.168.18.202   <none>        443/TCP   111d
~$ kubectl get endpoints -nkube-system kubernetes-dashboard
NAME                   ENDPOINTS            AGE
kubernetes-dashboard   192.168.34.17:8443   111d
```

### Regles IPTables

Nous voyons les réglés IPtables sur les nodes pour le service (**IP virtuelle + port**)

```
Chain KUBE-SERVICES (2 references)KUBE-SVC-XGLOHA7QRQ3V22RZ  tcp  --  0.0.0.0/0  192.168.18.202  /* kube-system/kubernetes-dashboard: cluster IP */ tcp dpt:443
```

Cette règle renvoi vers les réglés pour chaque endpoint

```
Chain KUBE-SVC-XGLOHA7QRQ3V22RZ (1 references)
target     prot opt source               destination         
KUBE-SEP-NG56X4MU77FFZ5RK  all  --  0.0.0.0/0   0.0.0.0/0  /* kube-system/kubernetes-dashboard: */
```

Cette règle s'occupe ensuite de faire le DNAT pour envoyer au POD sur le port d'exposition :

```
Chain KUBE-SEP-NG56X4MU77FFZ5RK (1 references)
target     prot opt source               destination         
DNAT       tcp  --  0.0.0.0/0  0.0.0.0/0  /* kube-system/kubernetes-dashboard: */ tcp to:192.168.34.17:8443
```

{% hint style="info" %}
Si nous avons plusieurs POD nous aurons plusieurs endpoint comme ceci :

```
Chain KUBE-SVC-FAITROITGXHS3QVF (1 references)
target     prot opt source               destination         
KUBE-SEP-TJ2CXVZOWVM6T4YV  all  --  0.0.0.0/0  0.0.0.0/0 /* kube-system/coredns:dns-tcp */ 
     statistic mode random probability 0.50000000000
KUBE-SEP-HWX34FYUV4Y7PLZG  all  --  0.0.0.0/0  0.0.0.0/0  /* kube-system/coredns:dns-tcp */
```
{% endhint %}

### Exposition du service à l’extérieur

Si nous demandons un service de type **NodePort** ou **LoadBalancer**, nous avons une exposition du service sur l'IP des Nodes sur un port spécifique (identique sur chaque node)

Nous pouvons visualiser cela dans la définition du service :

```
NAME                  TYPE           CLUSTER-IP      EXTERNAL-IP                                                                     PORT(S)                                                                                AGE
service/traefik       LoadBalancer   172.20.94.201   ab4367c2ff9ee4332b4177dfc86df9ac-21320c7a8fbd5518.elb.eu-west-3.amazonaws.com   3306:30699/TCP,3307:30004/TCP,2223:30375/TCP,22:32755/TCP,80:31992/TCP,443:31377/TCP   110d
```

Dans cet exemple, le port exposé à l’extérieur pour ce service 80 est le port <mark style="color:red;">**31992**</mark>

#### **LoadBalanceur AWS**

Si ce service est exposé derrière un AWS Loadbalancer voici \*\*\*\* le paramétrage des loadbalanceur

Ici notre LB **ab4367c2ff9ee4332b4177dfc86df9ac-21320c7a8fbd5518.elb.eu-west-3.amazonaws.com**

![](<../../.gitbook/assets/K8S--AWS Console-03.png>)

La redirection se fait sur un groupe que nous pouvons visualiser

![](<../../.gitbook/assets/K8S--AWS Console-02.png>)

Nous constatons donc bien que le LB renvoi bien vers un groupe de node (contenant tous les nodes du cluster) sur le **Nodeport** créé pour le service (ici <mark style="color:red;">**31992**</mark>**)**

**Note :** Dans cette exemple nous n'avons qu'un seul node qui répond car nous avons le paramètre suivant dans la définition du service : **External Traffic Policy: Local**

## Sources

{% embed url="https://nigelpoulton.com/explained-kubernetes-service-ports" %}

{% embed url="https://www.stackrox.io/blog/kubernetes-networking-demystified" %}

{% embed url="https://kubernetes.io/docs/concepts/services-networking/service" %}

{% embed url="https://rtfm.co.ua/en/kubernetes-service-load-balancing-kube-proxy-and-iptables" %}

{% embed url="https://guillaume.fenollar.fr/blog/kubernetes-kube-proxy-iptables" %}

{% embed url="https://aws.amazon.com/fr/premiumsupport/knowledge-center/eks-kubernetes-services-cluster" %}
