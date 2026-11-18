# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom : FLEURDEN-BISSIÈRE Lucas**
**Groupe : RS2**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   >3 noeuds, un control plane (le master) et les deux workers
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >CONTAINER ID   IMAGE                            COMMAND                  CREATED          STATUS          PORTS                                                              NAMES
   7a6f245fdd9c   ghcr.io/k3d-io/k3d-proxy:5.9.0   "/bin/sh -c nginx-pr…"   31 minutes ago   Up 30 minutes   0.0.0.0:8081->80/tcp, [::]:8081->80/tcp, 0.0.0.0:33251->6443/tcp   k3d-lab-serverlb
   f71c5080e240   rancher/k3s:v1.35.5-k3s1         "/bin/k3d-entrypoint…"   31 minutes ago   Up 30 minutes                                                                      k3d-lab-agent-1
   a1da78d67c7b   rancher/k3s:v1.35.5-k3s1         "/bin/k3d-entrypoint…"   31 minutes ago   Up 30 minutes                                                                      k3d-lab-agent-0
   f41a894896a6   rancher/k3s:v1.35.5-k3s1         "/bin/k3d-entrypoint…"   31 minutes ago   Up 31 minutes                                                                      k3d-lab-server-0
   >Un noeud k3d est un container Docker.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   >Trois objets Kubernetes distincts  :   Un Deployment (ex. deployment.apps/web) : l'objet de haut niveau que vous avez directement demandé. Il décrit l'état désiré de l'application (image du conteneur, nombre de réplicas, stratégie de mise à jour).
   >Un ReplicaSet (ex. replicaset.apps/web-68d995574f) : créé et géré automatiquement par le Deployment. Son rôle est de s'assurer qu'un nombre précis de pods identiques tourne en permanence.
   >Un Pod (ex. pod/web-68d995574f-q97hg) : créé et géré par le ReplicaSet. C'est la plus petite unité déployable dans Kubernetes, qui encapsule le conteneur applicatif (ici nginx).
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   >Le site devient temporairement inaccessible uniquement parce que le tunnel port-forward se coupe, mais le service lui-même est immédiatement rétabli par Kubernetes.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   >Non, le message n'a pas survécu.  Le système de fichiers d'un conteneur est éphémère. Lorsqu'un pod est supprimé et recréé par le ReplicaSet, Kubernetes démarre un tout nouveau conteneur à partir de l'image nginx d'origine, ce qui efface toutes les modifications effectuées directement à l'intérieur du conteneur précédent.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   >Bien que la commande kubectl port-forward cible le Service (svc/web), Kubernetes établit en réalité le tunnel réseau directement vers un seul pod derrière ce Service. Lorsque ce pod précis est supprimé, le tunnel est rompu.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   >kubectl create (approche impérative) : donne un ordre direct au cluster pour créer une nouvelle ressource à partir de zéro. Si l'objet existe déjà, la commande échoue avec une erreur (AlreadyExists) ; elle ne permet pas de mettre à jour une ressource existante.
   >kubectl apply (approche déclarative) : décrit l'état désiré dans un fichier manifeste (YAML) que l'on peut versionner dans Git. Kubernetes compare cet état désiré avec l'état actuel du cluster : si la ressource n'existe pas, il la crée ; si elle existe déjà, il calcule la différence et ne met à jour que ce qui a changé (opération idempotente).
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >Il permet au Service d'identifier dynamiquement quels pods doivent recevoir le trafic réseau, en filtrant ceux qui possèdent exactement les mêmes labels (ici app: site). Le Service maintient ainsi à jour sa liste d'adresses IP cibles (Endpoints) même lorsque des pods sont supprimés, recréés ou mis à l'échelle.
   >S'il ne correspond à aucun pod : le Service est quand même créé et conserve son adresse IP virtuelle (ClusterIP), mais sa liste d'Endpoints reste vide (<none>, vérifiable avec kubectl get endpoints site). Toute requête envoyée vers le Service échoue (erreur de connexion / Connection refused ou timeout), car aucun pod n'est associé pour traiter le trafic.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
 
   `NAME                    READY   STATUS    RESTARTS   AGE     IP          NODE               NOMINATED NODE   READINESS GATES
site-7676b744b7-fvdsc   1/1     Running   0          97s     10.42.2.5   k3d-lab-agent-0    <none>           <none>
site-7676b744b7-jjk85   1/1     Running   0          3m12s   10.42.0.9   k3d-lab-agent-1    <none>           <none>
site-7676b744b7-zdpgd   1/1     Running   0          97s     10.42.1.7   k3d-lab-server-0   <none>           <none>`

10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    >
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    >

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    >

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** ... / ...
