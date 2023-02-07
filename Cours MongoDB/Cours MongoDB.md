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