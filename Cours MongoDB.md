Mini-sujet donné jeudi matin ou peut-être mercredi aprèm, il faudra rendre un truc.

MongoDB permet d’importer et exporter des données dans différents formats.

SGBD = Système de Gestion de Bases de Données

MongoDB permet de gérer des bases de données et est en JSON.

Collection : Groupe de documents MongoDB.

Les collections n’imposent pas de schéma précis.

Les documents présents au sein d’une même collection peuvent avoir des champs différents, même s’ils sont souvent similaires.

Un document est un ensemble de données clé-valeur.

Le schéma permet de stocker l’information.

MongoDB possède une structure claire et simple : orientée objet

{} est un objet vide en JSON

MongoDB vient ajouter des types :

- le type Date : stocke sous la forme d'un entier signé de 8 octets le nombre de secondes écoulées depuis l'époque Unix (01/01/1970 à minuit)

- le type ObjectId : stocke sur 12 octets, utilisé en interne par MongoDB afin de générer des identifiants

MongoDB afin de générer des identifiants :

- Le type NumberLong et le type NumberInt : par défaut, MongoDB considère toute valeur numérique comme un nombre à virgule codé sur 8 octets. Représentant des entiers signés sur 4 et 8 octets.

- Le type BinData : pour stocker des chaines de caractères ne possédant pas de représentation de l’encodage UTF-8, ou n’importe quel contenu binaire (c’est donc les données en brut)

MongoDB en ligne de commande :

Bash

mongod

mongod –port 27017 –dbpath /data/databases –logpath / tmp/mongodb, log –logappend

sudo service mongodb stop

# Création d’une base de données dans MongoDB

``` bash

use tmp

db.maCollection.insertOne({

	« une_cle » : « une_valeur »

})
``````

Valeur autogénérée par la base de données

# Suppression d’une base de données

```	mongoshell
	
use laDaDbASupprimer;
	
db.dropDatabase()
```

### Les helpers
``` mongoshell
db.runCommand

```

#### Gestion des collections

#### Les collations

Une collation est un objet qui permet de générer la version locale, en adaptant certains éléments à chaque langue

``` javascript
{
	**locale**: < string >,
	caseLevel: < boolean >,
	caseFirst: < string >,
	strength: < entier >,
	numericOrdering: < boolean >,
	...
}
```

### Créer une collection

``` javascript
db.createCollection("maCollection", { "collation": {"locale": "fr"} })
```


### Les documents

#### Insertion d'un tableau de documents

``` javascript
db.maCollection.insert(< documents ou un tableau de documents >)

db.personnes.insert([
	{"_id":"personnel", "nom": "MOURANCHON", "prenom": "Thomas"},
	{"_id":"personnel", "nom": "MARQUET", "prenom": "Louis", "age": 20}
])
```

Du code déprécié (deprecated) est un code qui fonctionne mais qui est voué à disparaitre dans une prochaine mise à jour.

#### Modification d'un document

``` javascript
db.collection.updateOne(<filtre>,<modifications>)

db.personnes.update{
	{"_id": ObjectId:("dupont")}, // nom sans "" peut fonctionner mais il vaut mieux y laisser pour la syntaxe
	{
		$set : { "nom": "Dupont"}
	},
	{"multi": true }
}
```

MongoDB est sensible à la casse, donc attention car mettre une majuscule pour le nom d'un champ ne renverrra pas au même (exemple: "nom" est différent de "Nom")

En SQL, la meilleure façon de cibler un champ est d'utiliser son ID

Cas de l'upsert: la combinaison d'update et d'insert:
``` javascript
db.personnes.updateOne(
	{"prenom": "remi"},
	{
		$set : {"prenom": "Remi"}}
)
```

Effectuer des recherches sur la méthode `findAndModify`

### Validation des documents

``` javascript
var athletes = [{
	"nom": "Eclair",
	"prenom": "Jean-Michel",
	"discipline": "Course"
}, {
	"nom": "Cavalera",
	"prenom": "Max",
	"discipline": "Saut de haies"
}, {
	"nom": "Hammer",
	"prenom": "Hemsi"
}]

db.athletes.insertMany(athletes)
```

Exemple de schéma de validation:
``` javascript
var proprietes = {
	"nom": {
		"bsonType": "string",
		"pattern": "^[A-Z].*",
		"description": "Chaîne de caractères+ expression régulière - obligatoire" },
	"prenom": {
		"bsonType": "string",
		"description": "Chaîne de caractères - obligatoire" },
	"discipline": {
		"enum": ["Course", "Lancer de marteau", "Saut de haies"],
		"description": "Enumération - obligatoire" }
}
```

Maintenant que nos règles sont établies, nous allons modifier la collection à l'aide de la commande `collMod`

``` javascript
db.runCommand(
	{
		"collMod": "athletes",
		"validator": {
			$jsonSchema: {
				"bsonType": "object",
				"required": ["nom", "prenom", "discipline"],
				"properties": proprietes
			}
		}
	}
)
```

Essayer de lever une erreur qui permet de vérifier que les schémas de validation appliquée fonctionnent

Les méthodes find et findOne ont la même signature et permettent d'effectuer des requêtes mongo
(La 1ère agit comme un filtre, là où la 2nde affichera seulement les valeurs qu'on veut)
(Mettre un simple find() permet d'afficher l'ensemble du tableau)

``` javascript
db.collection.find(<requete>, <projection> )
```

Chacun de ces paramètres sont tous deux des documents.

``` javascript
DBQuery.shellBatchSize = 40
```

mongorc.js est un fichier de config qui se trouve à la racine du répertoire utilisateur
/home/userName

``` javascript
	db.maCollection.find().limit(12)
	db.maCollection.find().limit(12).pretty()
```

On peut utiliser des opérateurs afin d'affiner la recherche:
``` javascript
db.maCollection.find({"age": {$eq: 76} })
db.maCollection.find({"age": 76}, {"prenom": true}) // Affichera le prénom des personnes ayant 76 ans
```

On retrouve d'autres opérateurs tels que:
- $eq : égal à
- $ne : différent de
- $gt : supérieur à,  $gte : supérieur ou égal
- $lt : inférieur à, $lte : inférieur ou égal
- $in et $nin : absence ou présence
- $sort: trier par ordre croissant

On peut combiner ces opérateurs pour effectuer des recherches sur des intervalles:
``` javascript
	db.maCollection.find({"age": { $lt: 50, $gt: 20 }})
	db.maCollection.find({"age": { $nin: [23, 45, 78] }})
	db.maCollection.find({"age": { $exists: true }})
```

### Les opérateurs logiques

``` javascript
	db.personnes.find(
		{
			$and: [
				{
					"age": {
						$exists: 1
					}
				},
				{
					"age": {
						$nin: [23,45,234,100]
					}
				}
			]
		},
		{
			"_id": 0, // 1 veut dire qu'il faut afficher la valeur, et 0 qu'il ne le faut pas
			"nom": 1,
			"prenom": 1
		}
		)
```

#### L'opérateur $expr

Il permet d'utiliser des expressions dans nos requêtes. Ces expressions pourront contenir des opérateurs, des objets ou encore des chemins de champs (field path).

On va chercher à afficher les noms des documents dont la longueur du nom multipliée par 12 est supérieure à l'âge.

``` javascript
db.personnes.find({
	"nom": {$exists: 1},
	"age": {$exists: 1},
	$expr: { $gt: [ { $multiply: [ { $strLenCP: "$nom" }, 12 ] }, "$age" ] } // $ permet de prendre en compte seulement la valeur
},
{
	"_id": 0,
	"nom": 1
})
```

Afficher les comptes dont la somme des opérateurs de débit est supérieure au montant du crédit.

``` javascript
db.banque.find({
	"_id": {$exists: 1},
	$expr: {
		$gt: [
			{ $sum: "$debit" },
			"$credit"
		]
	}
})
```

#### L'opérateur $type

{ champ: { $type: < type BSON > } }
{ champ: { $type: [< type BSON >, < type BSON >] } }

``` javascript
db.personnes.insertOne({
	"nom": "Zidane", "prenom": "Z", "age": numberInt(50)
})
```

#### L'opérateur $mod

{ champ: { $mod: [ diviseur, reste ] } }

##### L'opérateur $where

Il permet d'exécuter directement du code Javascript dans une fonction, ce qui peut causer de grands ralentissements, c'est donc à éviter le plus possible.

``` javascript
db.personnes.find({
	$where: "this.nom.length > 6" // on peut remplacer this par obj
})

db.personnes.find({ $where: function() {
	return obj.nom.length > 6
} })
```

#### Les opérateurs de tableau

``` javascript
{ $push: { <champ>: <valeur>, ... } }
{ $pull: { <champ>: <valeur>, ... } }
```

``` javascript
db.hobbies.updateOne({ "_id": 1}, { $push: { "passions": "Tekken 7"} } )

db.hobbies.updateOne({ "_id": 2}, { $push: { "passions": "Théâtre"} } )

db.hobbies.updateOne({ "_id": 2}, { $pull: { "passions": "Théâtre"} } )
```



##### L'opérateur $addToSet

Il permet d'éviter l'ajout de doublons, quand on remplace $push par lui
recup les doc dans la collec personnes dont les personnes sont passionnées de jardinage

``` javascript
db.personnes.find({"interets": "jardinage"})

db.personnes.find({"interets": ["jardinage", "bridge"]})

db.personnes.find({"interets": { $all: ["bridge", "jardinage"] }})

db.personnes.find({"interets.1": "jardinage" }) // Attention car le système ne vérifie pas s'il est en-dehors du tableau

db.personnes.find({"interets": {$size: 2}})

db.personnes.find({"interets.1": {$exists: 1}}) // Récupère les personnes avec au moins 2 hobbis
```

#### L'opérateur $elemMatch

``` javascript
db.eleves.find({"notes": {$elemMatch: { $gt: 0, $lt: 10 } }})

db.eleves.find({"notes": {$all: [5, 18.50] } })
```

#### Les tableaux de documents

``` javascript
db.eleves.find({"notes.note: 10"})
```

Renvoyer les documents dont les élèves ont au moins une matière quelconque:

``` javascript
db.eleves.find({
	"notes" : {
		$elemMatch: { "note": {$gt: 10, $lte: 15 } }
		}
	}
)

db.eleves.find({ "notes.note": {$gt: 10, $lte: 15 } })
```

##### Le tri

``` javascript
curseur.sort(<tri>)
```

##### Les index

Algolia et elk pour elasic Surge

La nature de votre application devra impacter votre logique d'indexation: est-elle orientée écriture (write-heavy) ou lecture (read-heavy) ?

``` javascript
db.collection.createIndex(<champ + type>, <options?>)

db.personnes.createIndex({"age: -1"})

{
	"createCollectionAutomatically": false,
	"numIndexesBefore": 1,
	..
	ok:1
}

db.personnes.getIndexes()

db.personnes.dropIndex("age -1")
db.personnes.createIndex({"age": -1}, {"name": "index_age"})
```

Un index peut porter sur un ou plusieurs champs, ce dernier est ce qu'on applle un index composé

Attention car une recréation d'index peut avoir un impact très négatif sur les performances du serveur, donc avec du trafic important il vaut mieux faire cela en période de nuit ou de maintenance.

``` javascript
db.personnes.createIndex({"age": -1, "nom": 1}, { "name": "idx_age_nom", collation: {locale: "fr"} })
```

MongoDB permet l'utilisation de deux types d'index qui permettent de gérer les requêtes géospatiales::
- Les index de type '2dsphere' sont utilisés par des requêtes géospatiales intervenant sur une surface sphérique.
- Les index '2d' concernent des requêtes intervennant sur un plan Euclidien.

Pour un champ nommé 'donneesSpatiales' d'une collection 'cartographie' vous pouvez par exemple créer un index de type '2d' avec la commande:

``` javascript
db.cartographe.createIndex({"donnnesSpatiales": "2d" })
```

Pour la création d'un index '2dsphere' on utilisera plutôt:

``` javascript
db.cartographie.creatIndex({"donnesSpatiales": "2dsphere" })
```

Les index 2d font intervenir des coordonnées de type 'legacy'

``` javascript
	db.plan.insertOne({ "nom": "firstPoint", "geodata": [1,1] })

	db.plan.insertOne({ "nom": "firstPoint_bis", "geodata": [4.7, 44.5] })

	db.plan.insertOne({ "nom": "firstPoint_bis", "geodata": { "lon": 4.7, "lat": 44.5 } })
```

#### Les objets GeoJSON

``` javascript
{ type: <type d'objet>, coordinates: <coordonnees> }
```

##### Le type Point

``` javascript
{
"type": "Point",
"coordinates": [ 14.0, 1.0]
}
```

##### Le type MultiPoint

``` javascript
{
	"type": "MultiPoint",
	"coordinates": [
		[13.0, 1.0], [13.0, 3.0]
	]
}
```

##### Le type LineString

``` javascript
{
	"type": "LineString",
	"coordinates": [
		[13.0, 1.0], [13.0, 3.0]
	]
}
```

##### Le type Polygon

``` javascript
{
	"type": "Polygon",
	"coordinates": 
	[
		[
			[13.0, 1.0], [13.0, 3.0]
		],
		[
			[13.0, 1.0], [13.0, 3.0]
		]
	]
}
```

Création d'index:

``` javascript
db.avignon.createIndex({"localisation": "2dsphere"})

db.avignon2d.createIndex({"localisation": "2d"})
```

##### L'opérateur $nearSphere

``` javascript
{
	$nearSphere: {
		$geometry: {
			type: "Point",
			coordinates: [<longitude>, <latitude>]
		},
		$minDistance: <distance en metres>,
		$maxDistance: <distance en metres>
	}
}

{
	$nearSphere: [ <x>, <y> ],
	$minDistance: <distance en radians>,
	$maxDistance: <distance en radians>
}
```

``` javascript
var opera = { type: "Point", coordinates: [43.949749, 4.805325] }
```
Effectuer une requête sur la collection avignon

``` javascript
db.avignon.find(
{
	"localisation": {
		$nearSphere: {
			$geometry: opera
		}
	}, {"_id": 0, "nom": 1}
})
```

##### L'opérateur $geoWithin

Sélectionne les documents avec des données géospatiales qui existent entièrement dans une forme spécifique.

Cet opérateur n'effectue aucun tri et ne nécessite pas la création d'un index géospatial, on l'utilise de la manbière suivante:

``` javascript
{
	<champ des documents contenant les coordonnées>:
	{
		$geoWithin: 
		{
			<operateur de forme>: <coordonnes>
		}
	}
}
```

Création d'un polygone pour notre exemple :
``` javascript
var polygone = [
	[43.9548 , 4.80143],
	[43.95475 , 4.80779],
	[43.95045 , 4.81097],
	[43.4657 , 4.80449]
]

db.avignon.find(
{
	"localisation":
	{
		$geowithin:
		{
			$geometry:
			{
				type: "Polygon",
				coordinates: [polygone]
			}
		}
	}
}, {"_id": 0, "nom": 1}
)
```

La requête suivante utilise ce polygone :
``` javascript
db.avignon2d.find(
{
	"localisation":
	{
		$geowithin:
		{
			$polygon: polygone
		}
	}
})
```

## Le framework d'aggrégation

MongoDB met à disposition un puissant outil d'analyse et de traitement de l'information: le pipeline d'aggrégation (ou framework)

Métaphore du tapis roulant d'usine

Méthode utilisée:

``` javascript
db.collection.aggregate(pipeline, options)
```
- pipelin: désigne un tableau d'étapes
- options: désigne un document

Parmi les options, nous retiendrons:
- collation, permet d'affecter une collation à l'opération d'aggrégation
- bypassDocumentValidation: fonctionne avec un opérateur appelé $out et permet de passer au travers de la validation des documents.
- allowDiskUse: donne la possibilité de faire déborder les opérations d'écriture sur le disque

Vous pouvez appeler aggregate sans argument:
``` javascript
db.personnes.aggregate()
```

Au sein du shell, nous allons créer une variable pipeline:

``` javascript
var pipeline = []
db.personnes.aggregate(pipeline)
db.personnes.aggregate(
	pipeline,
	{
		"collation":
		{
			"locale": "fr"
		}
	}
)
```

Le filtrage avec $match

Cela permet




Commençons par la première étape

``` javascript
var pipeline = [{
	$match : {
		"interets: "jardinage"
	}
}]

db.personnes.aggregate(pipeline)
```

Cale corrsepond à la requête:
``` javascript
db.personnes.find({ "interets": "jardinage"})
```

``` javascript
var pipeline = [{
	$match : {
		"interets: "jardinage"
	},
	$match : {
		"nom": /^L/,
		"age": {$gt: 70}
	},
}]

db.personnes.aggregate(pipeline)
```

Sélection/modification de champs: $project
syntaxe:

``` javascript
{ $project: { <spec> } }
```

``` javascript
var pipeline = [
{
	$match : 
	{
		"interets: "jardinage"
	},
	$project : 
	{
		"_id": 0,
		"nom": 1,
		"prenom": 1,
		"super_senior": { $get: ["$age", 70] }
	},
	{
		$match: { "super_senior": true}
	}
}]

db.personnes.aggregate(pipeline)
```

``` javascript
var pipeline = [
{
	$match : 
	{
		"interets: "jardinage"
	},
	$project : 
	{
		"_id": 0,
		"nom": 1,
		"prenom": 1,
		"ville": "$adresse.ville"
	},
	{
		$match:
		{ 
			"ville": {$exists: true}
		}
	}
}]
```


### L'opérateur $addFields

``` javascript
{ $addFields: { <nouveau champ> : <expression>, ... } }
```

``` javascript
db.personnes.aggregate([
{
	$addFields:
	{
		"numero_secu_s": ""
	}
}
])

db.personnes.aggregate([
{
	$project:
	{
		......
		"numero_secu_s": 1
	}
}
])
```

#### L'opérateur $group

Regroupe des documents et crée un nouveau document par groupe

``` javascript
{
	$group: {
		"_id": <expressions>, // Cette ligne est obligatoire
		<champ>: { <operateur d'accumulation> }
	}
}
```

Opérateur d'accumulation: `$push`, `$sum`, `$avg`, `$min`, `$max`

``` javascript
var pipeline = [{
	$group: {
		"_id": "$age",
		"nombre_personnes": { $sum: 1 }
	}
},{
	$sort: {
		"nombre_personnnes": 
	}
}
}]

db.personnes.aggregate(pipeline)


db.personnes.aggregate([
	{
		$sortByCount: "$age"
	}
])


var pipeline = [{
	$group:
	{
		"_id": null,
		"nombre_personnes": { $sum: 1}
	}
}]


var pipeline = [{
	$match:
	{
		"age": { $exists: true}
	},
	{
		$group: 
		{
			"_id": null,
			"avg": { $avg: "$age" }
		}
	},
	{
		$project:
		{
			"_id": 0,
			"Age_moyen": { $ceil: '$avg' }
		}
	}
}]
```