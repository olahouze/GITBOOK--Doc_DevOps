# \[Concepts - Docker] - Gestion des "layers"

## Description

Chaque construction d'image Docker repose sur un empilement de "couches" appelées "**layers**"

Les layers représentent des modifications par rapport à l’état précédent et permet de partager des layers entre différentes images docker pour économiser de l’espace disque et accélérer les déploiement (système de cache)

## Fonctionnement

Chaque layers s'appuie sur les éléments fournis par la layer précédente et rajoute ses modifications

La somme de la layer précédente et des différences apportées par la layer courante est exposée à la layer supérieur

![](../.gitbook/assets/container-layers.jpg)

Il est possible que la layer suivante soir le POD

![](../.gitbook/assets/sharing-layers.jpg)

### Driver overlay2

Pour gérer ce système, Docker utiliser les drivers overlay

Ce système repose sur des dossiers dans **/var/lib/docker/overlay2** qui représentent chacun une layer

Dans chacun des dossier nous avons :&#x20;

* **link** : Fichier contenant l'identifiant de la layer "0"&#x20;
* **lower**: Fichier contenant la liste des layers précédentes
* **diff:** Dossiers contenant les éléments du file system modifiés par la layer &#x20;
* **merged** : Contient les éléments du File Systeme qui cumulent les layers précédentes et les modification de la layer courante (file systeme présenté à la couche supérieure)

## Exemple

Sur le Node nous constatons que l'utilisation maximum du stockage est bien réalisé sur le disque de l'EC2 dans le file systeme racine **/**

```
[root@ip-10-1-9-168 /]# df -h
Filesystem Size Used Avail Use% Mounted on
devtmpfs 7.8G 0 7.8G 0% /dev
tmpfs 7.8G 0 7.8G 0% /dev/shm
tmpfs 7.8G 3.0M 7.8G 1% /run
tmpfs 7.8G 0 7.8G 0% /sys/fs/cgroup
/dev/nvme0n1p1 30G 16G 15G 52% /
```

La plus grande consommation d'espace se situe dans : **/var/lib/docker/overlay2**

```
[root@ip-10-1-9-168 /]# du -a / | sort -r -n | head -n 20
31062820 /
29278424 /var
28825336 /var/lib
21139940 /var/lib/docker
21084260 /var/lib/docker/overlay2
7655016 /var/lib/kubelet
7038008 /var/lib/kubelet/pods
1964196 /var/lib/docker/overlay2/5f314b630e7439fbcd6517e6bb8c52020976e6b6979118065edc67784ebe551c
1821452 /var/lib/docker/overlay2/b6ac1f4b4fc0c2e200e50cea354ec756a63441cb392a41f757f110fcb5d28ded
1697004 /var/lib/docker/overlay2/b6ac1f4b4fc0c2e200e50cea354ec756a63441cb392a41f757f110fcb5d28ded/merged
```

Cet espace est utilisé par docker pour les files systems des POD

Grace à cet article nous pouvons identifier les POD qui utilisent les couches de l'overlay2 : [https://fabianlee.org/2021/04/08/docker-determining-container-responsible-for-largest-overlay-directories/](https://fabianlee.org/2021/04/08/docker-determining-container-responsible-for-largest-overlay-directories/)

Apres analyse sur le node qui consomme beaucoup de stockage local, nous obtenons ceci :

```
/k8s_airflow-scheduler_airflow-scheduler-7ff8469c5c-zk9j6_scheduler_9b503a02-297f-4222-a64a-8aac382937fc_0 =>/var/lib/docker/overlay2/5f314b630e7439fbcd6517e6bb8c52020976e6b6979118065edc67784
ebe551c/merged /k8s_superset_superset-worker-6976b99c66-fnlql_superset_6755e71d-630d-4eab-9a0b-e1016b00aeb9_0 =>/var/lib/docker/overlay2/b6ac1f4b4fc0c2e200e50cea354ec756a63441cb392a41f757f110fcb5d28ded/merged 1--Cas du airflow-scheduler_airflow-scheduler
```

### 1--Cas du **airflow-scheduler\_airflow-scheduler:**

Dans le dossier "diff" nous pouvons voir les modification réalisées dans cette couche

```
ll ./diff/
total 0
drwxr-xr-x 3 root root 21 Jan 18 01:16 opt
drwxr-xr-x 4 root root 33 May 11 13:50 run
drwxr-xr-x 3 root root 21 May 9 14:14 var
```

Comme cette couche est associé au POD c'est donc ces dossiers qui sont utilisées pour lire/ecrire par le POD&#x20;

### 2--Cas du **superset\_superset-worker-6976b99c66-fnlql\_superset:**

Dans le dossier "diff" nous pouvons voir les modification réalisées dans cette couche

```
ls -la ./diff/
total 0
drwxr-xr-x 3 root root 22 May 10 14:36 app
drwx------ 3 root root 20 May 10 14:36 root
drwxr-xr-x 3 root root 21 May 10 17:22 run
drwxrwxrwt 3 root root 27 May 10 17:23 tmp
drwxr-xr-x 3 root root 19 Feb 28 00:00 usr
```

### Conclusion&#x20;

Dans le cas de  **airflow-scheduler\_airflow-scheduler**  nous pouvons constaté dans la couche  **5f314b630e7439fbcd6517e6bb8c52020976e6b6979118065edc67784ebe551c**  que les données sont rajoutés régulièrement dans **/diff/opt/airflow/logs/scheduler**

```
root@ip-10-1-9-168 scheduler]# pwd
/var/lib/docker/overlay2/5f314b630e7439fbcd6517e6bb8c52020976e6b6979118065edc67784ebe551c/diff/opt/airflow/logs/scheduler
[root@ip-10-1-9-168 scheduler]# ll
total 0
drwxrwxr-x 7 50000 root 78 May 9 14:14 2022-05-09
drwxrwxr-x 7 50000 root 78 May 10 00:00 2022-05-10
drwxrwxr-x 7 50000 root 78 May 11 00:00 2022-05-11
lrwxrwxrwx 1 50000 root 38 May 11 00:00 latest -> /opt/airflow/logs/scheduler/2022-05-11
```

## Sources

{% embed url="https://docs.docker.com/storage/storagedriver/overlayfs-driver#how-the-overlay2-driver-works" %}

{% embed url="https://docs.docker.com/storage/storagedriver" %}

{% embed url="https://fabianlee.org/2021/04/08/docker-determining-container-responsible-for-largest-overlay-directories" %}
