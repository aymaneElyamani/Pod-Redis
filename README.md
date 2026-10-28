# Kubernetes avec Minikube — TP pratique

## 1. Présentation du projet

Ce projet a pour objectif de découvrir les bases de **Kubernetes** à travers des exercices pratiques sur Windows, en utilisant Minikube et Docker Desktop.

Les manipulations réalisées permettent de comprendre comment créer, inspecter, tester et supprimer des Pods Kubernetes.

### Objectifs

- Comprendre les notions de cluster, Pod et conteneur.
- Démarrer un cluster Kubernetes local.
- Déployer un conteneur Nginx.
- Déployer Redis à partir d'un fichier YAML.
- Exécuter des commandes à l'intérieur d'un Pod.
- Tester Redis et manipuler des données.
- Comprendre les labels et la durée de vie des données.

## 2. Environnement utilisé

| Outil | Utilisation |
|---|---|
| Windows | Système d'exploitation |
| Docker Desktop | Exécution des conteneurs |
| Minikube | Création d'un cluster Kubernetes local |
| kubectl | Communication avec Kubernetes |
| Nginx | Premier exemple de Pod |
| Redis | Exercice pratique sur un service de cache |

**Dossier du projet :**

```text
C:\Users\lenovo\Documents\minikube
```

## 3. Comprendre les notions de base

- **Docker :** permet de construire et d'exécuter des conteneurs.
- **Kubernetes :** permet de gérer des applications conteneurisées.
- **Cluster :** ensemble des composants qui exécutent et gèrent les applications Kubernetes.
- **Node :** machine qui participe au cluster.
- **Pod :** plus petite unité déployable de Kubernetes. Il contient un ou plusieurs conteneurs.
- **kubectl :** outil en ligne de commande utilisé pour communiquer avec Kubernetes.
- **YAML :** format de configuration permettant de décrire les ressources Kubernetes.

## 4. Vérifier les outils

Ouvrir CMD ou PowerShell et exécuter :

```bash
docker --version
minikube version
kubectl version --client
```

Vérifier que Docker Desktop est démarré avant de continuer.

## 5. Démarrer le cluster Kubernetes

Démarrer Minikube avec Docker comme pilote :

```bash
minikube start --driver=docker
```

Vérifier l'état du cluster :

```bash
minikube status
```

Vérifier que le nœud Kubernetes est opérationnel :

```bash
kubectl get nodes
```

Le nœud `minikube` doit avoir le statut `Ready`.

## 6. Premier exercice : créer un Pod Nginx

Créer un Pod nommé `mon-pod` :

```bash
kubectl run mon-pod --image=nginx
```

Vérifier son état :

```bash
kubectl get pods
```

Afficher les détails du Pod :

```bash
kubectl describe pod mon-pod
```

Ouvrir le tableau de bord Kubernetes :

```bash
minikube dashboard
```

Supprimer le Pod :

```bash
kubectl delete pod mon-pod
```

## 7. Deuxième exercice : déployer Redis avec YAML

### Étape 1 — Créer le fichier

Dans le dossier du projet, créer un fichier nommé `redis-pod.yaml`.

Copier le contenu suivant :

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

### Étape 2 — Comprendre la configuration

- `apiVersion: v1` : version de l'API Kubernetes utilisée.
- `kind: Pod` : indique que la ressource à créer est un Pod.
- `metadata.name` : nom du Pod.
- `labels.tier: cache` : ajoute un label pour identifier le rôle du Pod.
- `spec.containers` : définit les conteneurs du Pod.
- `name: redis` : nom du conteneur.
- `image: redis:7` : image utilisée pour exécuter Redis.

### Étape 3 — Créer le Pod

```bash
kubectl apply -f redis-pod.yaml
```

Résultat attendu :

```text
pod/redis-pod created
```

### Étape 4 — Vérifier le Pod

```bash
kubectl get pods
```

Afficher les informations détaillées :

```bash
kubectl get pod redis-pod -o wide
```

Afficher les labels :

```bash
kubectl get pod redis-pod --show-labels
```

Pour sélectionner les Pods portant le label `tier=cache` :

```bash
kubectl get pods -l tier=cache
```

## 8. Tester Redis

### Étape 1 — Vérifier que Redis répond

```bash
kubectl exec redis-pod -- redis-cli ping
```

Résultat attendu :

```text
PONG
```

Cela confirme que Redis répond à la commande envoyée.

### Étape 2 — Enregistrer une valeur

```bash
kubectl exec redis-pod -- redis-cli set cours kubernetes
```

Résultat attendu :

```text
OK
```

### Étape 3 — Lire la valeur

```bash
kubectl exec redis-pod -- redis-cli get cours
```

Résultat attendu :

```text
kubernetes
```

Ces commandes permettent de tester Redis sans installer `redis-cli` directement sur Windows.

## 9. Supprimer les ressources

Supprimer le Pod Redis à partir de son fichier YAML :

```bash
kubectl delete -f redis-pod.yaml
```

Vérifier qu'il a été supprimé :

```bash
kubectl get pods
```

Si aucun Pod ne reste dans l'espace de noms `default`, Kubernetes affiche :

```text
No resources found in default namespace.
```

Pour arrêter le cluster Minikube :

```bash
minikube stop
```

Pour le redémarrer ultérieurement :

```bash
minikube start --driver=docker
```

## 10. Points importants à retenir

1. `kubectl apply -f fichier.yaml` crée ou met à jour les ressources décrites dans un fichier.
2. `kubectl get pods` affiche les Pods présents dans l'espace de noms courant.
3. `kubectl describe pod nom` fournit des informations détaillées sur un Pod.
4. `kubectl exec` permet d'exécuter une commande dans un conteneur.
5. Les labels servent à organiser et sélectionner les ressources Kubernetes.
6. Un fichier YAML permet de conserver la configuration d'une ressource et de la recréer.
7. **Les données Redis de cet exercice sont stockées en mémoire.** Aucun stockage persistant n'a été configuré. Supprimer le Pod entraîne donc la perte de ces données.

## 11. Commandes essentielles

| Commande | Rôle |
|---|---|
| `minikube start --driver=docker` | Démarrer le cluster |
| `minikube status` | Vérifier son état |
| `kubectl get nodes` | Afficher les nœuds |
| `kubectl get pods` | Afficher les Pods |
| `kubectl describe pod redis-pod` | Examiner un Pod |
| `kubectl apply -f redis-pod.yaml` | Créer le Pod depuis YAML |
| `kubectl exec redis-pod -- redis-cli ping` | Tester Redis |
| `kubectl delete -f redis-pod.yaml` | Supprimer le Pod |
| `minikube stop` | Arrêter le cluster |

---

**Conclusion :** ce TP constitue une première introduction à Kubernetes. Il permet de comprendre le cycle de vie d'un Pod, l'utilisation de fichiers YAML, les commandes `kubectl` et les bases de l'exécution d'un service Redis dans un cluster local.
