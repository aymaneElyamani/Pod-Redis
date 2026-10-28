# TP Kubernetes — Déployer et tester Redis avec Minikube

![Kubernetes](https://img.shields.io/badge/Kubernetes-Minikube-326CE5?logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Desktop-2496ED?logo=docker&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7-DC382D?logo=redis&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-11-0078D4?logo=windows&logoColor=white)

Travaux pratiques d'initiation à **Kubernetes** sur Windows : création d'un cluster local avec **Minikube**, déploiement d'un Pod **Nginx**, puis déploiement et test d'un Pod **Redis** décrit dans un manifeste YAML.

---

## Sommaire

- [Objectifs pédagogiques](#objectifs-pédagogiques)
- [Prérequis](#prérequis)
- [Arborescence du projet](#arborescence-du-projet)
- [Démarrage rapide](#démarrage-rapide)
- [1. Vérifier l'environnement](#1-vérifier-lenvironnement)
- [2. Démarrer le cluster](#2-démarrer-le-cluster)
- [3. Exercice 1 — Un Pod Nginx](#3-exercice-1--un-pod-nginx)
- [4. Exercice 2 — Redis via un manifeste YAML](#4-exercice-2--redis-via-un-manifeste-yaml)
- [5. Vérifier et sélectionner les Pods](#5-vérifier-et-sélectionner-les-pods)
- [6. Tester Redis](#6-tester-redis)
- [7. Nettoyage et arrêt du cluster](#7-nettoyage-et-arrêt-du-cluster)
- [Commandes essentielles](#commandes-essentielles)
- [Points clés à retenir](#points-clés-à-retenir)
- [Dépannage](#dépannage)
- [Conclusion](#conclusion)

---

## Objectifs pédagogiques

À la fin de ce TP, vous saurez :

- expliquer les notions de **cluster**, **node**, **Pod** et **conteneur** ;
- démarrer et arrêter un cluster Kubernetes local avec Minikube ;
- créer un Pod en ligne de commande (`kubectl run`) et depuis un fichier YAML (`kubectl apply`) ;
- inspecter une ressource avec `kubectl get` et `kubectl describe` ;
- exécuter des commandes à l'intérieur d'un conteneur avec `kubectl exec` ;
- utiliser les **labels** pour organiser et filtrer les ressources ;
- comprendre la différence entre un Pod et un stockage persistant.

## Prérequis

| Outil | Version indicative | Rôle |
|---|---|---|
| Windows | 10 / 11 | Système hôte |
| [Docker Desktop](https://www.docker.com/products/docker-desktop/) | 4.x | Moteur de conteneurs utilisé par Minikube |
| [Minikube](https://minikube.sigs.k8s.io/docs/start/) | 1.30+ | Cluster Kubernetes local |
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | 1.27+ | Client en ligne de commande |

> Docker Desktop doit être démarré **avant** `minikube start`. Sur Windows, l'utilisation du backend WSL 2 est recommandée.

## Arborescence du projet

```text
minikube/
├── README.md          # Ce document
└── redis-pod.yaml     # Manifeste du Pod Redis
```

**`redis-pod.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
  labels:
    tier: cache
spec:
  containers:
    - name: redis
      image: redis:7
```

## Démarrage rapide

```bash
minikube start --driver=docker
kubectl apply -f redis-pod.yaml
kubectl get pods
kubectl exec redis-pod -- redis-cli ping   # → PONG
minikube stop
```

---

## 1. Vérifier l'environnement

Ouvrir **PowerShell** ou **CMD** puis vérifier que les outils sont installés :

```bash
docker --version
minikube version
kubectl version --client
```

Contrôler également que Docker Desktop est bien lancé :

```bash
docker info
```

## 2. Démarrer le cluster

```bash
minikube start --driver=docker
```

Vérifier l'état du cluster et du nœud :

```bash
minikube status
kubectl get nodes
```

Résultat attendu : le nœud `minikube` apparaît avec le statut `Ready`.

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   1m    v1.30.x
```

## 3. Exercice 1 — Un Pod Nginx

Créer un Pod nommé `mon-pod` à partir de l'image Nginx :

```bash
kubectl run mon-pod --image=nginx
```

Suivre la création puis inspecter le Pod :

```bash
kubectl get pods
kubectl describe pod mon-pod
```

Ouvrir le tableau de bord Kubernetes dans le navigateur :

```bash
minikube dashboard
```

Supprimer le Pod une fois l'observation terminée :

```bash
kubectl delete pod mon-pod
```

## 4. Exercice 2 — Redis via un manifeste YAML

### 4.1 Comprendre le manifeste

| Champ | Signification |
|---|---|
| `apiVersion: v1` | Version de l'API Kubernetes utilisée |
| `kind: Pod` | Type de ressource créée |
| `metadata.name` | Nom du Pod (`redis-pod`) |
| `metadata.labels.tier: cache` | Label servant à identifier et filtrer le Pod |
| `spec.containers` | Liste des conteneurs du Pod |
| `containers[].name` | Nom du conteneur (`redis`) |
| `containers[].image` | Image utilisée (`redis:7`) |

### 4.2 Créer le Pod

```bash
kubectl apply -f redis-pod.yaml
```

Résultat attendu :

```text
pod/redis-pod created
```

### 4.3 Vérifier la création

```bash
kubectl get pods
kubectl get pod redis-pod -o wide
kubectl describe pod redis-pod
```

Attendre que la colonne `STATUS` indique `Running` et que `READY` affiche `1/1`.

## 5. Vérifier et sélectionner les Pods

Afficher les labels du Pod :

```bash
kubectl get pod redis-pod --show-labels
```

Filtrer les Pods portant le label `tier=cache` :

```bash
kubectl get pods -l tier=cache
```

## 6. Tester Redis

Toutes les commandes sont exécutées **dans le conteneur**, sans installer `redis-cli` sur Windows.

| Étape | Commande | Résultat attendu |
|---|---|---|
| Vérifier que Redis répond | `kubectl exec redis-pod -- redis-cli ping` | `PONG` |
| Enregistrer une valeur | `kubectl exec redis-pod -- redis-cli set cours kubernetes` | `OK` |
| Relire la valeur | `kubectl exec redis-pod -- redis-cli get cours` | `kubernetes` |

Exemple de session :

```text
$ kubectl exec redis-pod -- redis-cli ping
PONG

$ kubectl exec redis-pod -- redis-cli set cours kubernetes
OK

$ kubectl exec redis-pod -- redis-cli get cours
kubernetes
```

## 7. Nettoyage et arrêt du cluster

Supprimer le Pod à partir du manifeste :

```bash
kubectl delete -f redis-pod.yaml
```

Vérifier qu'il ne reste plus de Pod :

```bash
kubectl get pods
```

```text
No resources found in default namespace.
```

Arrêter le cluster (les ressources sont conservées sur disque) :

```bash
minikube stop
```

Pour reprendre le TP plus tard :

```bash
minikube start --driver=docker
```

Pour supprimer complètement le cluster et son disque :

```bash
minikube delete
```

## Commandes essentielles

| Commande | Rôle |
|---|---|
| `minikube start --driver=docker` | Démarrer le cluster |
| `minikube status` | Vérifier l'état du cluster |
| `minikube dashboard` | Ouvrir le tableau de bord |
| `kubectl get nodes` | Lister les nœuds |
| `kubectl get pods` | Lister les Pods du namespace courant |
| `kubectl get pods -o wide` | Afficher les détails réseau et le nœud |
| `kubectl get pods --show-labels` | Afficher les labels |
| `kubectl get pods -l tier=cache` | Filtrer par label |
| `kubectl describe pod redis-pod` | Afficher les détails d'un Pod |
| `kubectl apply -f redis-pod.yaml` | Créer / mettre à jour une ressource |
| `kubectl delete -f redis-pod.yaml` | Supprimer une ressource |
| `kubectl exec redis-pod -- redis-cli ping` | Exécuter une commande dans un conteneur |
| `kubectl logs redis-pod` | Consulter les logs du Pod |
| `minikube stop` / `minikube delete` | Arrêter / supprimer le cluster |

## Points clés à retenir

1. Un **Pod** est la plus petite unité déployable de Kubernetes ; il encapsule un ou plusieurs conteneurs.
2. `kubectl apply -f fichier.yaml` est **déclaratif** : la commande crée la ressource ou la met à jour si elle existe déjà.
3. `kubectl get` sert à **lister**, `kubectl describe` à **diagnostiquer** un objet.
4. `kubectl exec` permet d'entrer ponctuellement dans un conteneur, sans installer l'outil sur la machine hôte.
5. Les **labels** sont des paires clé/valeur utilisées pour organiser et sélectionner les ressources.
6. Un fichier YAML rend le déploiement **reproductible** et versionnable avec le code.
7. ⚠️ **Les données Redis de ce TP sont stockées en mémoire.** Aucun `PersistentVolume` n'est configuré : supprimer le Pod entraîne la perte des données. Pour aller plus loin, ajouter un `volumeMount` et un `PersistentVolumeClaim`.

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| `Cannot connect to the Docker daemon` | Docker Desktop n'est pas démarré | Lancer Docker Desktop puis réessayer |
| `minikube start` bloqué ou très lent | Image Minikube en cours de téléchargement | Attendre ou relancer la commande |
| Pod en `ImagePullBackOff` | Image inaccessible ou nom invalide | `kubectl describe pod` puis vérifier l'image |
| Pod en `CrashLoopBackOff` | Le conteneur s'arrête au démarrage | `kubectl logs redis-pod` pour analyser |
| `error: unable to upgrade connection` | Proxy / tunnel Minikube perturbé | `minikube stop` puis `minikube start` |
| Conflit de port ou de contexte | Un autre cluster est sélectionné | `kubectl config current-context` et `kubectl config use-context minikube` |

## Conclusion

Ce TP constitue une première approche concrète de Kubernetes. Il couvre le cycle de vie complet d'un Pod — création, inspection, test, suppression — ainsi que l'usage des manifestes YAML, des labels et de `kubectl exec`. La suite logique consiste à remplacer le Pod nu par un **Deployment** (réplicas, auto-réparation), à l'exposer via un **Service**, puis à ajouter un **stockage persistant** pour Redis.
