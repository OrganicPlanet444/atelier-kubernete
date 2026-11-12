# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom : Eddy FABRE**
**Groupe : RS2**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > 3 nœuds. `k3d-lab-server-0` a le rôle `control-plane` : il fait tourner l'API server, le scheduler, les controllers et la base d'état (avec k3s, il peut aussi exécuter des pods). `k3d-lab-agent-0` et `k3d-lab-agent-1` sont des workers (rôle `<none>`) : ils exécutent les pods des applications.
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   > 4 conteneurs Docker : les 3 nœuds (image `rancher/k3s`) et `k3d-lab-serverlb` (image `k3d-proxy`), un load balancer qui expose l'API Kubernetes sur l'hôte (port 44311). Dans k3d, un nœud est donc simplement un conteneur Docker qui fait tourner k3s, avec son propre containerd à l'intérieur pour lancer les pods. Ce n'est pas une VM.


## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > Un Deployment web, qui crée un ReplicaSet web-68d995574f, qui crée lui-même le Pod web-68d995574f-xxxxx (le Service kubernetes existait déjà par défaut).
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > Le pod supprimé est passé en Terminating et un nouveau pod (web-68d995574f-2t8v2) a été recréé en 6 s, car le ReplicaSet maintient en permanence le nombre de réplicas demandé (boucle de réconciliation).

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non : le message était écrit dans le système de fichiers éphémère du conteneur, et le nouveau pod repart de l’image nginx d’origine.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Même lancé sur svc/web, le port-forward crée un tunnel vers un seul pod, choisi au démarrage : quand ce pod a été supprimé, le tunnel s’est coupé (Exit 1) et il a fallu le relancer.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > create est impératif et échoue si la ressource existe déjà ; apply est déclaratif : il crée ou met à jour la ressource pour qu’elle corresponde au manifeste, et on peut le relancer sans erreur (idempotent).
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Le selector choisit, par leurs labels, les pods qui reçoivent le trafic du Service ; s’il ne correspond à aucun pod, le Service n’a aucun endpoint et les requêtes échouent.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   @OrganicPlanet444 ➜ /workspaces/atelier-kubernete (main) $ kubectl get pods -l app=site -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP          NODE               NOMINATED NODE   READINESS GATES
site-5dd5f89b86-g4xvv   1/1     Running   0          5m34s   10.42.2.7   k3d-lab-agent-0    <none>           <none>
site-5dd5f89b86-gf6g8   1/1     Running   0          58s     10.42.1.9   k3d-lab-agent-1    <none>           <none>
site-5dd5f89b86-pj4k5   1/1     Running   0          58s     10.42.0.6   k3d-lab-server-0   <none>           <none>
@OrganicPlanet444 ➜ /workspaces/atelier-kubernete (main) $ 

10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Le nouveau pod (site-b4f5fb6f7-n47wk) était en ImagePullBackOff ; kubectl describe pod -l app=site donne la cause dans les Events : l’image nginx:9.99 n’existe pas sur le registre.

11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Oui : le rolling update garde l’ancien pod (site-5dd5f89b86-g4xvv, Running) tant que le nouveau n’est pas Ready, donc le Service continuait à lui envoyer le trafic.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    > https://automatic-spork-4q77jg5p4v4w27wp6-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > kubectl get deploy mario retrouve le Deployment créé dans Portainer : c’est un Deployment Kubernetes classique (Portainer ajoute seulement quelques labels et annotations), car Portainer et kubectl parlent au même composant, le kube-apiserver.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** ... / ...
