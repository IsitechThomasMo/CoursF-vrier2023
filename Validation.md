#### Exercice 1

Modifiez la collection salle afin que soient dorénavant validés les documents destinés à y être insérés ; cette validation aura lieu en mode « strict » et portera sur les champs suivants :

nom sera obligatoire et devra être de type chaîne de caractères.

capacite sera obligatoire et devra être de type entier (int).

Dans le champ adresse, les champs codePostal et ville, tous deux de type chaîne de caractères, seront obligatoires.

``` javascript
db.runCommand(
{
	collMod: "salles",
	validator:
	{
	    $jsonSchema:
		{
			bsonType: "object",
		    title: "objet de validation des salles",
		    required: ["nom","capacite","adresse.codePostal","adresse.ville"],
		    properties:
		    {
	        "nom": { bsonType: "string" },
	        "capacite": { bsonType: "int" },
	        "adresse.codePostal": { bsonType: "string" },
	        "adresse.ville": { bsonType: "string" }
			}
	    }
	}
})
```

Que constatez-vous lors de la tentative d’insertion suivante, et quelle en est la cause ?

``` javascript
db.salles.insertOne( 
{"nom": "Super salle", "capacite": 1500, "adresse": {"ville": "Musiqueville"}} 
)
```

Le champ CodePostal est manquant alors qu'on cherche à le rendre obligatoire

Que proposez-vous pour régulariser la situation ?

``` javascript
db.salles.insertOne( 
{
	"nom": "Super salle",
	"capacite": 1500, 
	"adresse": 
	{
		"ville": "Musiqueville",
		"codePostal":"69360"
	}
})
```

#### Exercice 2

Rajoutez à vos critères de validation existants un critère supplémentaire : le champ _id devra dorénavant être de type entier (int) ou ObjectId.

``` javascript
db.runCommand(
{
	collMod: "salles",
	validator:
	{
	    $jsonSchema:
	    {
		    bsonType: "object",
		    title: "objet de validation des salles",
		    required: ["nom","capacite","adresse.codePostal","adresse.ville"],
		    properties:
		    {
		        "id": { bsonType: ["int","objectId"] },
		        "nom": { bsonType: "string" },
		        "capacite": { bsonType: "int" },
		        "adresse.codePostal": { bsonType: "string" },
		        "adresse.ville": { bsonType: "string" }
		    }
	    }
	}
})
```

Que se passe-t-il si vous tentez de mettre à jour l’ensemble des documents existants dans la collection à l’aide de la requête suivante :

``` javascript
db.salles.updateMany({}, {$set: {"verifie": true}}) 
```

Seul le dernier document de la collection est modifié.

Supprimez les critères rajoutés à l’aide de la méthode delete en JavaScript

#### Exercice 3

Rajoutez aux critères de validation existants le critère suivant :

Le champ smac doit être présent OU les styles musicaux doivent figurer parmi les suivants : "jazz", "soul", "funk" et "blues".

Que se passe-t-il lorsque nous exécutons la mise à jour suivante ?

``` javascript
db.salles.update({"_id": 3}, {$set: {"verifie": false}})
```
Seul l'une des salles est mise à jour, la 3ème.