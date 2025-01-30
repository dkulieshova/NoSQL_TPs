# TP n°2 : Prise en main de MongoDB

## Un serveur de gestion de bases de données (NoSQL) documentaires et distribuées

**Fait par Daria Kulieshova INFOA3**

MongoDB est une base de données NoSQL orientée documents, qui stocke les données sous forme de documents JSON/BSON flexibles, contrairement aux bases relationnelles qui utilisent des tables et des schémas fixes. Ses avantages incluent :

- Une grande scalabilité horizontale grâce au sharding
- Une flexibilité dans la gestion des données semi-structurées
- Une forte performance pour les requêtes complexes
- Une intégration facile avec les applications modernes

Cependant, MongoDB a aussi des inconvénients, notamment :

- Une consommation mémoire plus élevée
- L’absence de transactions multi-documents aussi robustes que dans les bases SQL traditionnelles
- Une cohérence éventuellement plus faible selon le modèle de réplication utilisé

## Installation et prise en main

Pour ce TP, on va utiliser **MongoDB Compass** sur Windows.

1. Créer une nouvelle connexion (`TP2`).
2. Ajouter la collection `films` sous la base `local`.
3. Importer le fichier `films.json` contenant les données en choisissant :
   - `Add data` > `Import JSON or CSV file`.
4. Cliquer sur le bouton `Open MongoDB shell` pour exécuter les requêtes.

## Requêtes MongoDB

### 1. Vérifier que les données ont été importées

```javascript
db["films"].count()
// DeprecationWarning: Collection.count() is deprecated. Use countDocuments or estimatedDocumentCount.
db.films.countDocuments()
// 278
```

### 2. Afficher un document de la collection `films`

```javascript
db.films.findOne()
```

### 3. Afficher la liste des films d’action

```javascript
db.films.find({ genre: "Action" })
```

### 4. Afficher le nombre de films d’action

```javascript
db.films.count({ genre: "Action" })
// 36
```

### 5. Afficher les films d’action produits en France

```javascript
db.films.find({ genre: "Action", country: "FR" })
```

### 6. Afficher les films d’action produits en France en 1963

```javascript
db.films.find({ genre: "Action", country: "FR", year: 1963 })
```

### 7. Afficher les films d’action réalisés en France sans les grades

```javascript
db.films.find({ genre: "Action", country: "FR" }, { grades: 0 })
```

### 8. Afficher les films d’action réalisés en France sans identifiants

```javascript
db.films.find({ genre: "Action", country: "FR" }, { _id: 0 })
```

### 9. Afficher les titres et grades des films d’action réalisés en France sans identifiants

```javascript
db.films.find({ genre: "Action", country: "FR" }, { _id: 0, title: 1, grades: 1 })
```

### 10. Afficher les films d’action réalisés en France avec une note supérieure à 10

```javascript
db.films.find(
    { genre: "Action", country: "FR", "grades.note": { $gt: 10 } },
    { _id: 0, title: 1, "grades.note": 1 }
)
```

### 11. Afficher les films d’action réalisés en France avec uniquement des notes supérieures à 10

```javascript
db.films.find(
    { genre: "Action", country: "FR", "grades.note": { $not: { $lte: 10 } } },
    { _id: 0, title: 1, "grades.note": 1 }
)
```

### 12. Afficher les différents genres de la base `films`

```javascript
db.films.distinct("genre")
```

### 13. Afficher les différents grades attribués

```javascript
db.films.distinct("grades.grade")
// [ 'A', 'B', 'C', 'D', 'E', 'F' ]
```

### 14. Afficher tous les films avec au moins un des artistes suivants `["artist:4", "artist:18", "artist:11"]`

```javascript
db.films.find({ "actors._id": { $in: ["artist:4", "artist:18", "artist:11"] } })
```

### 15. Afficher tous les films qui n’ont pas de résumé

```javascript
db.films.find({ summary: { $exists: false } })
```

### 16. Afficher tous les films tournés avec Leonardo DiCaprio en 1997

```javascript
db.films.find({ "actors.last_name": "DiCaprio", "actors.first_name": "Leonardo", year: 1997 })
```

### 17. Afficher les films tournés avec Leonardo DiCaprio ou en 1997

```javascript
db.films.find({
    $or: [
        { "actors.last_name": "DiCaprio", "actors.first_name": "Leonardo" },
        { year: 1997 }
    ]
})
