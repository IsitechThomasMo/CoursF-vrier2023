### Préparation des données:

##### A. Importez un jeu de données de localisation, comme les restaurants dans une ville. Vous pouvez utiliser la commande mongoimport pour importer des données dans MongoDB.

Récupération d'un jeu de données de localisation sur Kaggle, à l'URL suivante: 
https://www.kaggle.com/datasets/jackywang529/michelin-restaurants?select=one-star-michelin-restaurants.csv

Celui-ci contient des données sur les restaurants du guide Michelin de 2018 et 2019

##### B. Assurez-vous d'avoir un champ de localisation géospatiale, comme la latitude et la longitude.

Ceux-ci sont bien inclus.


### Requêtes MongoDB:

##### A. Recherchez les restaurants qui sont ouverts à partir de 18h00. Utilisez la méthode find () et les opérateurs de comparaison pour trouver les documents qui correspondent à vos critères.

``` javascript
// On génère d'abord des horaires d'ouverture et de fermeture qui n'existe pas dans la base

db.uneEtoile.updateMany({},
[{
	$addFields:
	{
		"schedule":
		[
			{
				"opening":
				{ 
					$floor: 
					{
						$multiply: [ { $rand: {} }, 24 ]
					}
				}
			},
			{
				"closing":
				{ 
					$floor: 
					{
						$multiply: [ { $rand: {} }, 24 ]
					}
				}
			}
		]
	}
}])

// Puis voici la réponse à la question

db.uneEtoile.find(
{
	"schedule.opening": { $eq: 18 }
})
```

##### B. Triez les restaurants par note, du plus haut au plus bas. Utilisez la méthode sort () pour trier les résultats.

``` javascript
// On génère d'abord une note, car n'existant pas dans la base

db.uneEtoile.updateMany({},
[{
	$addFields:
	{
		"note":
		{
			$floor: 
			{
				$multiply: [ { $rand: {} }, 10 ]
			}
		}
	}
}])

// Puis voici la réponse

db.uneEtoile.find().sort( { note: -1 } )
```



### Indexation avec MongoDB:

##### A. Créez un index sur le champ d'ouverture des restaurants pour améliorer les performances de la recherche. Utilisez la méthode createIndex ().

``` javascript
db.uneEtoile.createIndex(
{ "index": 1 },
{ 
	unique: true,
	sparse: true,
	expireAfterSeconds: 3600
})
```

##### B. Vérifiez que l'index a été créé en utilisant la méthode listIndexes ().

``` javascript
db.uneEtoile.getIndexes()
```


### Requêtes géospatiales:

##### A. Recherchez les restaurants qui se trouvent à moins de 2 km d'une certaine localisation. Utilisez la méthode find () avec un opérateur géospatial pour trouver les restaurants à l'intérieur d'un cercle.

##### B. Recherchez les restaurants qui se trouvent dans un certain rayon autour d'un point de localisation spécifique. Utilisez la méthode find () avec un opérateur géospatial pour trouver les restaurants à l'intérieur d'un cercle.



### Framework d'agrégation:

##### A. Calculez la moyenne des notes des restaurants. Utilisez le framework d'agrégation de MongoDB pour effectuer des calculs sur les données.

``` javascript
db.uneEtoile.aggregate([
{
	$group:
	    {
			$avg: {"note"}}
		}
	}
])
```

##### B. Trouvez les restaurants les plus populaires en fonction du nombre de commentaires. Utilisez le framework d'agrégation de MongoDB pour groupes les données et effectuer des calculs.

``` javascript
// Ajout d'un nombre de commentaires car non-défini

db.uneEtoile.updateMany({},
[{
	$addFields:
	{
		"commentsNumber":
		{
			$floor: 
			{
				$multiply: [ { $rand: {} }, 20 ]
			}
		}
	}
}])

// Puis la solution

db.uneEtoile.aggregate([
{
    $sort: { "commentsNumber":-1 }
}
])
```

### Export de la base de données:

##### A. Exportez les résultats des requêtes dans un fichier CSV pour un usage ultérieur. Utilisez la commande mongoexport pour exporter des données de MongoDB.