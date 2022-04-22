---
description: Liste des commandes utiles en toutes circonstences
---

# \[SOS] - Commandes utiles

## Debug

Exécuter un POD pour avoir l’accès à un shell avec suppression du POD une fois terminé

```
kubectl run -i --tty --rm debug --image=busybox --restart=Never -- sh
```
