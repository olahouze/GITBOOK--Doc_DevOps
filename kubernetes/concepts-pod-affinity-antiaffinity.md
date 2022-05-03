# \[Concepts] - Pod Affinity / Antiaffinity

Description

Pod Affinity

Il est possible de specifier à un POD de fonctionner sur le meme node qu'un autre POD

Pod Antiaffinity

Il est possible de specifier à un POD de ne pas fonctionner sur le même node qu'un autre POD

## Fonctionnement

Dans ces deux cas, le scheduleur va identifier le(s) POD(s) de reference en utilisant des labels.

Une fois les PODs references identifié il va scheduler le nouveau POD en respectant l'affinity/antiaffinity

Il existe, comme pour les nodes affinity, 2 possibilités :&#x20;

* requiredDuringSchedulingIgnoredDuringExecution: Le POD doit obligatoirement respecter la regle (sinon il ne sera pas schedulé)
* preferedDuringSchedulingIgnoredDuringExecution: Le POD devrait respecter la regle mais peut etre schedulé ailleur si la regle ne peut etre respecté

{% hint style="danger" %}
Les règles **nodeAffinity / toleration / podAffinity** sont toutes évaluées lors de la prise de décision. Il faut faire attention que toutes ces règles **ne rentrent pas en conflit**

Exemple : si une règle de **Pod Affinity** "requiredDuringSchedulingIgnoredDuringExecution "demande au POD de fonctionner à coté d'un autre POD qui se trouve sur le Node A MAIS que le Node A ne respecte pas la regle de NodeAffinity equiredDuringSchedulingIgnoredDuringExecution, le POD ne demarrera pas
{% endhint %}

