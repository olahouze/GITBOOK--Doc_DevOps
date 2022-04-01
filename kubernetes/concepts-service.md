# \[Concepts] - Service

## Description

Les services permettent d'exposer sur <mark style="color:red;">**une IP et un port fixe**</mark> l’accès a des PODs

Il existe différents type d'exposition

1. **Cluster IP** : Permet d'exposer des services uniquement pour les POD à l'interrieur du cluster (type par defaut)
2. **Node Port** : Permet d'exposer sur tous les nodes un port et une IP fixe pour acceder au service (non recommendé)
3. **Loadbalanceur** : Permet de demander au Provider la création d'un Load Balanceur qui va donner l’accès au service au monde extérieur&#x20;

{% hint style="info" %}
La methode de redirection des flux entre le LoadBalanceur et le service est différent selon les Providers : [https://www.stackrox.io/blog/kubernetes-networking-demystified/](https://www.stackrox.io/blog/kubernetes-networking-demystified/)
{% endhint %}

## Fonctionnement

Les différentes étapes lors de l'exposition d'un service

1. L'utilisateur demande la création d'un service en spécifiant
   1. Port d'exposition du service
   2. Port d’écoute des POD (optionnel)
   3. IP d'exposition du service (optionnel)
2. Le cluster va créer le service et lui associé l'IP adéquat
3. Le cluster va créer les différents **endpoints** en fonction du nombre de POD concernés par le service
4. Le **kube-proxy** de chaque node va agir sur les règles **iptables** pour permettre à chaque node de
   1. Traiter le trafic à destination de l'IP du service (IP virtuelle)
   2. Renvoyer le paquet sur un node qui héberge un POD concerné par le service
5. (option si type = Loadbalancer) : Demande au provider de créer le loadbalanceur pour rediriger le flux vers les nodes sur les bons ports)

### Cluster IP

![](<../.gitbook/assets/Network K8S-Cluster IP.drawio.png>)

### LoadBalancer

![](<../.gitbook/assets/Network K8S-LoadBalancer.drawio.png>)

## Sources

{% embed url="https://nigelpoulton.com/explained-kubernetes-service-ports" %}

{% embed url="https://www.stackrox.io/blog/kubernetes-networking-demystified" %}

{% embed url="https://kubernetes.io/docs/concepts/services-networking/service" %}
