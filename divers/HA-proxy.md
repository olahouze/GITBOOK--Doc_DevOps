---
description: Explication simplifier de la solution HA Proxy
---

# HA Proxy

## Fonctionnement

![](<../.gitbook/assets/HA Proxy - SEMIPUB-Default.drawio.png>)

Le HA proxy est un système de load balanceur

La configuration repose sur des fichiers de configuration dans **/etc/haproxy** qui s’agrège les un avec les autres grâce à leurs numéro dans le nom (00-xxx puis 01\_xx, etc...)

Une bonne pratique est d'utiliser le dossier **/etc/haproxy/lists** pour stocker les éléments permettant d'identifier un flux (IP ou FQDN). Ces fichiers permettrons de créer facilement des ACL

## Configuration

### FrontEnd

Un fichier FrontEnd se présente comme suite

```
frontend ft_impala
  mode tcp
  # Ecoute des flux sur les IP suivantes
  bind 51.68.66.34:21050 name IMP1
  bind 10.10.64.6:21050 name IMP1_INT
  ###### ACLs
  acl nw_lomg src -f /etc/haproxy/lists/nw_lomg
  acl src_common-production src -f /etc/haproxy/lists/src_common-production
  acl ip_ext dst 51.68.66.34
  # Rules
  tcp-request connection reject unless nw_lomg OR src_common-production 
  # Routing
  use_backend bk_impala

```

Nous avons plusieurs parties :&#x20;

1. Le nom du frontEnd
2. Le **mode** d’écoute du frontend (TCP/HTTP/HTTPS)
3. Le bind sur les interfaces qui doivent traiter ce flux
4. **ACL** : création des différentes ACL à utiliser dans les règles en les peuplant avec les fichiers de lists
5. **Rules** : Règles pour le traitement du flux reçu
6. **Routing** : Une fois la règle passée, action de routage du flux sur le bon backend

### BackEnd

Un fichier BackEnd se présente comme suite

```
backend bk_impala
  mode tcp
  timeout server     15m
  server srv_impala1 10.10.64.214:21050 check
  server srv_impala2 10.10.64.215:21050 check
  server srv_impala3 10.10.64.216:21050 check
  server srv_impala4 10.10.64.217:21050 check
  server srv_impala5 10.10.64.218:21050 check
  server srv_impala6 10.10.64.219:21050 check
```

Nous avons plusieurs parties :&#x20;

1. Le nom du BackEnd
2. Le **mode** de communication avec le backend
3. Les paramètres pour ce pool de serveur
4. La liste des serveur au format : **\[nom] \[FQDN/IP]:port \[option]**



## Sources

{% embed url="https://www.haproxy.org" %}
