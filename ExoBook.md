Créez une base de données sample nommée "sample_db" et une collection appelée "employees". Insérez les documents suivants dans la collection "employees":

``` javascript
db.createCollection("employees")

db.employees.insert({ name: "John Doe", age: 35, job: "Manager", salary: 80000 })

db.employees.insert({ name: "Jane Doe", age: 32, job: "Developer", salary: 75000 })

db.employees.insert({ name: "Jim Smith", age: 40, job: "Manager", salary: 85000 })
```


Écrivez une requête MongoDB pour trouver tous les documents dans la collection "employees".

``` javascript
db.employees.find()
```


Écrivez une requête pour trouver tous les documents où l'âge est supérieur à 33.

``` javascript
db.employees.find(
{
	"age": 
	{
		$gt: 33
	}
})
```


Écrivez une requête pour trier les documents dans la collection "employees" par salaire décroissant.

``` javascript
db.employees.aggregate([ 
{ 
	$sort: 
	{
		"$salary": 1
	}
} ]) // On peut aussi remplacer aggregate par find à condition d'intégrer son contenu dans un $expr
```


Écrivez une requête pour sélectionner uniquement le nom et le job de chaque document.

``` javascript
db.employees.find(
{
	"_id": {$exists: 1}
},
{
	"_id": 0,
	"name": 1,
	"job": 1
})
```


Écrivez une requête pour compter le nombre d'employés par poste.

``` javascript
db.employees.aggregate(
{ 
	$group: 
	{ 
		_id: "$job", count: { $sum: 1 } 
	}
}) // _id est ici un paramètre pour $group, et $sum : 1 signifie qu'on incrémente de 1 à chaque fois
```


Écrivez une requête pour mettre à jour le salaire de tous les développeurs à 80000.

``` javascript
db.employees.updateMany(
{
	job:"Developer"
},
{
	$set: { "salary": 80000} 
})
```