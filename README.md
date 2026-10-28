# TP Kubernetes — Déployer et tester Redis avec Minikube

![Kubernetes](https://img.shields.io/badge/Kubernetes-Minikube-326CE5?logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Desktop-2496ED?logo=docker&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7-DC382D?logo=redis&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-11-0078D4?logo=windows&logoColor=white)

Travaux pratiques d'initiation à **Kubernetes** sur Windows : création d'un cluster local avec **Minikube**, déploiement d'un Pod **Nginx**, puis déploiement et test d'un Pod **Redis** décrit dans un manifeste YAML.

---

## Sommaire

- [Arborescence du projet](#arborescence-du-projet)
- [Démarrage rapide](#démarrage-rapide)
- [1. Démarrer le cluster](#2-démarrer-le-cluster)
- [2. Exercice 1 — Un Pod Nginx](#3-exercice-1--un-pod-nginx)
- [3. Exercice 2 — Redis via un manifeste YAML](#4-exercice-2--redis-via-un-manifeste-yaml)
- [4. Vérifier et sélectionner les Pods](#5-vérifier-et-sélectionner-les-pods)
- [5. Tester Redis](#6-tester-redis)
- [6. Nettoyage et arrêt du cluster](#7-nettoyage-et-arrêt-du-cluster)

---

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

## 1. Démarrer le cluster

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

## 2. Exercice 1 — Un Pod Nginx

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

## 3. Exercice 2 — Redis via un manifeste YAML

### 3.1 Comprendre le manifeste

| Champ | Signification |
|---|---|
| `apiVersion: v1` | Version de l'API Kubernetes utilisée |
| `kind: Pod` | Type de ressource créée |
| `metadata.name` | Nom du Pod (`redis-pod`) |
| `metadata.labels.tier: cache` | Label servant à identifier et filtrer le Pod |
| `spec.containers` | Liste des conteneurs du Pod |
| `containers[].name` | Nom du conteneur (`redis`) |
| `containers[].image` | Image utilisée (`redis:7`) |

### 3.2 Créer le Pod

```bash
kubectl apply -f redis-pod.yaml
```

Résultat attendu :

```text
pod/redis-pod created
```

### 3.3 Vérifier la création

```bash
kubectl get pods
kubectl get pod redis-pod -o wide
kubectl describe pod redis-pod
```

Attendre que la colonne `STATUS` indique `Running` et que `READY` affiche `1/1`.

## 4. Vérifier et sélectionner les Pods

Afficher les labels du Pod :

```bash
kubectl get pod redis-pod --show-labels
```

Filtrer les Pods portant le label `tier=cache` :

```bash
kubectl get pods -l tier=cache
```

## 5. Tester Redis

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

## 6. Nettoyage et arrêt du cluster

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

## Conclusion

Ce TP constitue une première approche concrète de Kubernetes. Il couvre le cycle de vie complet d'un Pod — création, inspection, test, suppression — ainsi que l'usage des manifestes YAML, des labels et de `kubectl exec`. La suite logique consiste à remplacer le Pod nu par un **Deployment** (réplicas, auto-réparation), à l'exposer via un **Service**, puis à ajouter un **stockage persistant** pour Redis.
