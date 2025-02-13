# TP n°4, MapReduce avec CouchDB

**Fait par Daria Kulieshova INFOA3**

# Présentation de CouchDB

Apache CouchDB est une base de données NoSQL orientée documents qui stocke les données sous forme de documents JSON. Elle repose sur une architecture distribuée et un modèle sans schéma, ce qui la rend particulièrement adaptée aux applications nécessitant une réplication efficace et une tolérance aux pannes.

## Caractéristiques principales :
- **Stockage sous format JSON** : Les documents sont stockés en JSON, facilitant la manipulation des données.
- **Requêtes avec JavaScript** : CouchDB utilise JavaScript et MapReduce pour l'indexation et les requêtes.
- **Répartition et réplication** : CouchDB permet la réplication entre plusieurs instances, ce qui est utile pour la haute disponibilité et le travail hors ligne.
- **Interface RESTful** : Les interactions avec la base se font via une API HTTP, ce qui simplifie l'accès aux données depuis n'importe quelle application.
- **Gestion des conflits** : Grâce à son modèle de réplication, CouchDB gère naturellement les conflits et permet une synchronisation efficace des données.

CouchDB est particulièrement utilisé dans des environnements distribués où la synchronisation et la flexibilité sont essentielles, comme les applications mobiles ou les systèmes nécessitant une résilience élevée.

Pour installer CouchDB sur Docker, il faut exécuter la commande suivante :
```
# docker run -d --name couchdb -e COUCHDB_USER=admin -e COUCHDB_PASSWORD=password -p 5984:5984 couchdb
```

```
# curl -X GET http://admin:password@localhost:5984/
{"couchdb":"Welcome","version":"3.4.2","git_sha":"6e5ad2a5c","uuid":"95974ffe4409c305ae9ecea0b542595f","features":["access-ready","partitioned","pluggable-storage-engines","reshard","scheduler"],"vendor":{"name":"The Apache Software Foundation"}}

# curl -X PUT http://admin:password@localhost:5984/films
{"ok":true}
```

Ensuite, il faut aller à http://localhost:5984/_utils/ et insérer à la main le fichier films.json.
On vérifie l'insértion : 
```
# curl -X GET http://admin:password@localhost:5984/films                           
{"instance_start_time":"1739439129","db_name":"films","purge_seq":"0-g1AAAABPeJzLYWBgYMpgTmHgzcvPy09JdcjLz8gvLskBCeexAEmGBiD1HwiyEhlwqEtkSKqHKMgCAIT2GV4","update_seq":"1-g1AAAACLeJzLYWBgYMpgTmHgzcvPy09JdcjLz8gvLskBCeexAEmGBiD1HwiyMpgTGXOBAuwGpoZmpsYG6HpwmJLIkFQP1c4A1m6cZGqUnGSBrjgLANsgKhg","sizes":{"file":168282,"external":290115,"active":150155},"props":{},"doc_del_count":0,"doc_count":1,"disk_format_version":8,"compact_running":false,"cluster":{"q":2,"n":1,"w":1,"r":1}}

# curl -X GET http://admin:password@localhost:5984/films/_all_docs?include_docs=true
```

---

# Présentation de MapReduce

MapReduce est un modèle de programmation utilisé pour traiter et générer de grandes quantités de données de manière distribuée. Il repose sur deux étapes principales :

1. **Map** : Chaque nœud traite une partie des données en parallèle et génère des paires clé-valeur intermédiaires.
2. **Reduce** : Les résultats intermédiaires sont agrégés pour produire un résultat final.

## Fonctionnement :
- **Scalabilité** : MapReduce permet de traiter efficacement de grands volumes de données en répartissant la charge sur plusieurs machines.
- **Tolérance aux pannes** : En cas de panne d’un nœud, les tâches peuvent être redistribuées.
- **Simplicité** : Il permet de traiter des données massives sans se soucier des détails de la distribution.

Dans CouchDB, MapReduce est utilisé pour créer des vues et indexer les documents JSON. Les fonctions Map sont écrites en JavaScript et permettent de structurer les données, tandis que Reduce permet d'agréger ces résultats pour des analyses plus poussées.

MapReduce est donc un outil puissant pour l’analyse de données distribuées et l’optimisation des requêtes dans des bases NoSQL comme CouchDB.

# Solution de l'exercice n° 1 :

### 1. Modèle de représentation de la matrice

Pour représenter la matrice **M** sous forme de documents structurés, nous nous inspirons du modèle PageRank. Chaque document dans la collection **C** représente une page **P\_i** avec les informations suivantes :

- L'identifiant de la page **i**
- Une liste des pages liées à cette page avec leurs poids correspondants

#### Exemple de document JSON :

```json
{
  "page_id": "P1",
  "liens": [
    {"page": "P2", "poids": 0.5},
    {"page": "P3", "poids": 0.2}
  ]
}
```

### 2. Calcul de la norme d’un vecteur

Pour calculer la norme d’un vecteur **V = (v\_1, v\_2, ..., v\_N)**, nous utilisons l’expression suivante :

```math
||V|| = \sqrt{v_1^2 + v_2^2 + ... + v_N^2}
```

Dans un système MapReduce, nous pouvons effectuer ce calcul en deux étapes :

1. **Map** : chaque page **P\_i** émet ses valeurs \(v_j^2\) sous forme de paires clé-valeur.
2. **Reduce** : la somme des valeurs est calculée, puis la racine carrée est appliquée.

#### Implémentation MapReduce

- **Map** :

```json
{
  "key": "norme",
  "value": v_j^2
}
```

- **Reduce** :

```json
{
  "key": "norme",
  "value": "\sqrt{\sum v_j^2 }"
}
```

### 3. Produit matrice-vecteur

Le produit de la matrice **M** et du vecteur **W = (w\_1, w\_2, ..., w\_N)** est défini par :

```math
\varphi_i = \sum_{j=1}^{N} M_{ij} w_j
```

#### Traitement MapReduce pour le produit matrice-vecteur

1. **Map** : Pour chaque élément \(M_{ij}\), émettre **(i, M\_{ij} w\_j)**.
2. **Reduce** : Pour chaque \(i\), sommer les valeurs reçues pour obtenir \(\varphi_i\).

#### Implémentation MapReduce

- **Map** :

```json
{
  "key": i,
  "value": M_ij * w_j
}
```

- **Reduce** :

```json
{
  "key": i,
  "value": "\sum M_ij * w_j"
}
```

Ainsi, la multiplication matrice-vecteur est parallélisée efficacement avec MapReduce.

**Solution de l'exercice n° 1 :**

## 1. Modèle de représentation de la matrice

Pour représenter la matrice **M** sous forme de documents structurés, nous nous inspirons du modèle PageRank. Chaque document dans la collection **C** représente une page **P_i** avec les informations suivantes :
- L'identifiant de la page **i**
- Une liste des pages liées à cette page avec leurs poids correspondants

### Exemple de document JSON :
```json
{
  "page_id": "P1",
  "liens": [
    {"page": "P2", "poids": 0.5},
    {"page": "P3", "poids": 0.2}
  ]
}
```

## 2. Calcul de la norme d’un vecteur

Pour calculer la norme d’un vecteur **V = (v_1, v_2, ..., v_N)**, nous utilisons l’expression suivante :
```math
||V|| = \sqrt{v_1^2 + v_2^2 + ... + v_N^2}
```

Dans un système MapReduce, nous pouvons effectuer ce calcul en deux étapes :
1. **Map** : chaque page **P_i** émet ses valeurs \( v_j^2 \) sous forme de paires clé-valeur.
2. **Reduce** : la somme des valeurs est calculée, puis la racine carrée est appliquée.

### Implémentation MapReduce
- **Map** :
```json
{
  "key": "norme",
  "value": v_j^2
}
```
- **Reduce** :
```json
{
  "key": "norme",
  "value": "\sqrt{\sum v_j^2 }"
}
```

## 3. Produit matrice-vecteur

Le produit de la matrice **M** et du vecteur **W = (w_1, w_2, ..., w_N)** est défini par :
```math
\varphi_i = \sum_{j=1}^{N} M_{ij} w_j
```

### Traitement MapReduce pour le produit matrice-vecteur
1. **Map** : Pour chaque élément \( M_{ij} \), émettre **(i, M_{ij} w_j)**.
2. **Reduce** : Pour chaque \( i \), sommer les valeurs reçues pour obtenir \( \varphi_i \).

### Implémentation MapReduce
- **Map** :
```json
{
  "key": i,
  "value": M_ij * w_j
}
```
- **Reduce** :
```json
{
  "key": i,
  "value": "\sum M_ij * w_j"
}
```
