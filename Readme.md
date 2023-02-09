# Mongo DB

Si les images ne s'affiche pas correctement : [View on Notion](https://www.notion.so/Mongo-DB-548c9379efcc495390dda8e8c907ddaa)

[Examens TP](https://www.notion.so/Examens-TP-774cdf708d8d4d23b601759414bd6432)

À faire : 

- Comment renommer une collection
- Expliquer la méthode `findAndModify`
- Lever une exception grâce aux validateurs
- Diminué le temp d’une requête avec les indexation (montrer des analyses)
- Comment arrondir un nombre grâce à la pipeline ($round)

Collection : ensemble de documents (Equivalent d’une table dans une bdd)

Document : ensemble de donnée clé-valeur, le schéma d’un document est dit “dynamique” car les champs pouvant lui être assigné ne sont pas prédéfinie.

![https://media.geeksforgeeks.org/wp-content/uploads/20200219180521/MongoDB-database-colection.png](https://media.geeksforgeeks.org/wp-content/uploads/20200219180521/MongoDB-database-colection.png)

![https://miro.medium.com/max/702/0*rlLOjmeLlGQyzETu.png](https://miro.medium.com/max/702/0*rlLOjmeLlGQyzETu.png)

Un nombre signé est un nombre négatif ou positif

MongoDB est sensible à la casse.

Dans le cli de MongoDB Compass, `db` est un alias vers la bdd en cours d’éxecution

- MongoDb
    
    MongoDB ajoute des types au format JSON:
    
    c’est appelé le format BSON
    
    - Date → sous la forme d’un entier signé de 8 octets
    - ObjectId → stocké sur 12 octets (id →clé primaire SQL)
    - Le type NumberInt et le type NumberLong: par défaut, Mongo considere toute valeur numérique comme un nombre à virgule codé sur 8 octets. Représentent des entiers signé sur 4 (NumberInt) et 8 (NumberLong) octets.
    - BinData: pour stocker des caines de catactères ne possédant pas de représentation dans l’encodage utf-8
    
- Commande shell MongoDB
    
    Lister les bases de données: `show database`
    
    Changer la base de donnée sélectionné : `use admin` (utilise la base de donnée admin)
    
    ### Les helpers
    
    `db.runCommand({"collstats": "testing"})`
    
    Affiche les info de la collection `testing`
    
    `db.adminCommand("currentOp")`
    
    Affiche les opération effectué actuellement sur la base de donnée
    
    ### Les collations
    
    ```jsx
    {
       locale: <string> required,
       caseLevel: <boolean>,
       caseFirst: <string>,
       strength: <int>,
       numericOrdering: <boolean>,
       alternate: <string>,
       maxVariable: <string>,
       backwards: <boolean>
    }
    ```
    
    ### Les validations
    
    ```jsx
    const properties = {
    	"prenom": {
    		bsonType: "string",
        description: "'prenom' must be a string and is required"
    	},
    }
    
    db.runCommand(
    {
    	"collMod": "documentName",
    	"validator": {
    		$jsonSchema: {
    			"bsonType": "object",
    			"required": ["nom", "prenom", "discipline"],
    			"properties": properties
    		}
    	}
    });
    ```
    
    Les validations permettent d’effectuer des 
    
    - Commandes d’insertion / création
        - Créer ou insérer un document dans une collection
            
            `db.maCollection.insertOne({"cle": "valeur"})`
            
            Ici `maCollection` est comparable à une table en SQL
            
            Si la base de donnée utilisé n’existe pas elle est automatiquement créer lors de l’éxecution de la commande
            
        - Créer une collection
            
            ```jsx
            db.createCollection("collectionName", 
            	{"collation": {
            		"local": "fr"
            	}
            })
            ```
            
        
        - Créer un document dans une collection
            
            ```jsx
            db.collectioName.insertOne({}) /// empty object
            db.collectioName.insertMany([{},{}]) /// array of 2 empty object
            ```
            
    - Commandes de suppression
        
        supprimer une collection:
        
        ```bash
        db.dropDatabase()
        db.maCollection.drop()
        ```
        
    - Commandes de recherche
        - Lister les documents d’une collection
            
            `db.maCollection.find().pretty()`
            
        - Affiner la recherche
            
            ```jsx
            db.collectionName.find().limit(12) /// renvoie maximum 12 résultats
            db.collectionName.find({
            	"age": { 
            		$lt: 50,
            		$gt: 20
            	}
            })
            db.collectionName.find({
            	"age": {
            		$nin: [23, 45, 78] 
            	}
            })
            db.collectionName.find({
            	"age": {
            		$exist: true 
            	}
            })
            
            ```
            
        
        $eq = equals
        
        $ne = not equals
        
        $in = have
        
        $nin = don’t have
        
        $gte = supérieur ou égal à
        
        $lt = inférieur à
        
    - Commandes de Modification
        
        ### Modification d’un document
        
        ```jsx
        /// db.documentName.updateOne(<filtre>, <modificateur>)
        db.documentName.update(
        	{"nom": "Jean"},
        	{
        		$set: {"nom": "newName"}
        	},
        	{
        		multi: true
        	}
        )
        ```
        
        - L’Upsert
            
            Upsert est la combinaison d’un update et d’un insert.
            
            ```jsx
            db.documentName.update(
            	{"nom": "Jean"},
            	{
            		$set: {"nom": "newName"}
            	},
            	{
            		"upset": true
            	}
            )
            ```
            
            Ici si aucun document n’est trouvé avec le filtre, il sera insert
            
    
    ### Le trie
    
    Utiliser pour trié les résultat de la requête
    
    ```jsx
    curseur.sort(<trie>)
    ```
    
- Les opérateurs
    
    ### L’opérateur $expr
    
    Il permet d’utiliser des expressions dans nos requêtes. Ces expressions pourront contenir des operateurs, des objects ou encore des chemins d’objet pointant vers des champs.
    
    Afficher les documents dont la longueur du nom multiplié par 12 est supérieur à l’âge.
    
    ```jsx
    db.personnes.find({
    	"nom": { $exist: true },
    	"age": { $exist: true },
    	$expr: {
    		$gt: [
    			{
    				$multiply: [
    					{
    						$strlenCP: "$nom"
    					},
    					12
    				]
    			},
    			"$age"
    		]
    	}
    },
    {
    	"_id": false,
    	"nom": true
    })
    ```
    
    ### L’opérateur $type
    
    ```jsx
    db.personnes.insertOne({
    	"nom": "unNom",
    	"age": numberInt(50)
    })
    ```
    
    ### L’opérateur $mod
    
    Prend un tableau en paramètre, effectue un modulo
    
    ```jsx
    db.personnes.find({
    	$mod: [<diviseur>, <reste>]
    })
    ```
    
    ### L’opérateur $where
    
    Permet d’éxecuter du code JavaScript passé sous forme de chaine de caractère
    
    ```jsx
    db.personnes.find({
    	$where: "this.nom.lenght > 6"
    })
    /// Ou
    db.personnes.find({
    	$where: function() { obj.nom.lenght > 6 }
    })
    ```
    
    ## Les opérateurs de tableau
    
    ### L’opérateur $push
    
    Permet d’ajouter un élément dans un tableau
    
    ```jsx
    /// Syntaxe 
    { $push: { <champ>: <valeur>, ... }}
    ```
    
    ### L’opérateur $pull
    
    Permet de supprimer un élément dans un tableau
    
    ```jsx
    /// Syntaxe 
    { $pull: { <champ>: <valeur>, ... }}
    ```
    
    ### L’opérateur $addToSet
    
    Permet d’ajouter un élément dans un tableau si il n’est pas déjà présent dans celui ci
    
    ```jsx
    { $addToSet: { <champ>: <valeur>, ... }}
    ```
    
    ### L’opérateur $all
    
    L’opérateur $all permet de chercher sans se soucier de l’ordre des éléments, et il retourne uniquement les documents qui contient strictement ces valeurs (il agit comme un AND)
    
    ```jsx
    db.personnes.find({
    	"interets": {
    		$all: ["bridge", "jardinage"]
    	}
    })
    
    db.eleves.find({
    	"notes": {
    		$all: [5, 7.50]
    	}
    })
    ```
    
    Rechercher par index
    
    ```jsx
    db.personnes.find({
    	"interets.1": "jardinage"
    })
    ```
    
    Rechercher les éléments ayant au moins 2 valeur dans le tableau
    
    ```jsx
    db.personnes.find({
    	"interets.1": { exists: 1 }
    })
    ```
    
    ### L’opérateur $elemMatch
    
    Retourne les éléments ayant une note entre 0 et 10.
    
    ```jsx
    db.eleves.find({
    	"notes": {
    		$elemMatch: {
    			$gt: 0,
    			$lt: 10
    		}
    	}
    })
    ```
    
- Les tableau de document
    
    Cette requête va renvoyer les documents dont les eleves ont au moins une note entre 10 et 15 dans une matière quelconque
    
    ```jsx
    db.eleves.find({
    	"notes": {
    		$elemMatch: {
    			"note": { $gt: 10, $lte: 15 }
    		}
    	}
    })
    ```
    
    ```jsx
    db.eleves.find({
    	"notes.note": { $gt: 10, $lte: 15 }
    })
    ```
    
    ```jsx
    db.eleves.find({
    	"notes.0.note": { $gt: 10 }
    })
    ```
    

### Les indexes

l’indéxation réduis le temps d’écriture, mais améliore le temps de lecture

Il faut ciblé à indexé en priorité les champs cherché en priorité lors d’une requête

Algolia et elk (elasticsearch) sont des outils qui facilite la gestion des index

La nature de votre application devra impacter votre logique d’indexation : est elle orienté écriture (write-heavy) ? ou lecture (read-heavy)

Créer un index pour une collection

```jsx
db.collection.createIndex(<champ + type>, <option?>)

// Example :
db.collection.createIndex({ "age": -1 }) // -1 correspond à l'ordre d'apparition
// 1 = croissant
// -1 = décroissant

// Passer des options à la méthode "createIndex"

db.collection.createIndex({ "age": -1 }, { "name": "index_age" }) 
// l'index sera nommé "index_age" 

db.collection.createIndex({ "age": -1 }, { "background": true })
// la création de l'index se fera en background

db.collection.createIndex(
	{
		"age": -1,
		"nom": 1
	},
	{
		"name": "index_age_nom",
		"background": true,
		"collation": {
			"locale": "fr"
		}
	}
)
```

Un index peut  porter sur plusieurs champs : c’est un index composé.

L’ordre dans lequel les champs son énuméré son important

Supprimer un index

```jsx
db.collection.dropIndex("index_-name")
```

Supprimer et ajouter des indexes sur une grosse base de donné peut avoir un effet négatif sur l’utilisation de celle-ci pendant la modification. (Provoquer des ralentissements)

### Les requêtes géospaciales

il existe 2 type d’index pour les requêtes géospaciales :

- l’index `2dsphere`, utilisé pour les requêtes géospaciales intervenant sur une surface sphérique
- l’index `2d`, utiliser pour les requêtes sur un plan Euclidien.

Créer un index géospaciale

```jsx
db.cartographie.createIndex({ "donneesSpaciales": "2d" })
db.cartographie.createIndex({ "donneesSpaciales": "2dsphere" })
```

Les index 2d font intervenir des coordonnes de type `legacy`

Différentes manière de créer un point géographique

```jsx
db.plan.insertOne({ "nom": "firstPoint" }, { "geodata": [1,1] })

db.plan.insertOne({ "nom": "firstPoint_bis"}, { "geodata": [4.7, 44.5] })

db.plan.insertOne({ "nom": "firstPoint_bis"}, { "geodata": {"long": 4.7, "lat": 44.5 }})
```

### Le type GeoJson

[RFC 7946 GeoJson](https://www.rfc-editor.org/rfc/rfc7946)

Les différents type geoJson :

- Point
- MultiPoint
- LineString
- MultiLineString
- Polygon
- MultiPolygon

### L’opérateur $nearSphere

```jsx
{
    $nearSphere: {
        $geometry {
            "type": "Point",
            "coordinate": [<longitude>, <latitude>]
        },
        $minDistance: <distance en metre>
        $maxDistance: <distance en metre>
    }
}
{
    $nearSphere: [<longitude>, <latitude>]
    $minDistance: <distance en metre>
    $maxDistance: <distance en metre>
}
```

### L’opérateur $geoWithin

Cet opérateur n’effectue aucun tri et ne necessite pas la création d’un index geospaciale, on l’utilise de la maniere suivante.

```jsx
const polygone = [
    [43.9548, 4.80143],
    [43.95475, 4.80779],
    [43.95045, 4.81097],
    [43.4657, 4.80449]
];

db.avignon2d.find( 
	{
		"localisation": { $geoWithin: { $polygon: polygone }} 
	}
)
```

```jsx
const coord = {
    "type": "Point",
    "coordinates": [43.94985229378091, 4.8055980697123415]
}

db.avignon.find( 
	{
		"localisation": { $nearSphere: { $geometry: coord }} 
	}
)
```

### La méthode aggregate

$match dois être utilisé le plus en amont possible

on peut construire la requête en amont 

Quelques examples :

```jsx
const pipeline = [{
    $match: {
        "interets": "jardinage"
    }
}]
db.personnes.aggregate(pipeline)
```

```jsx
const pipeline = [
    {
        $match: {
            "interets": "cuisine"
        }
    },
    {
        $project: {
            "_id": 0,
            "nom": 1,
            "prenom": 1,
            "ville": "$adresse.ville"
        }
    },
    {
        $match: {
            "ville": { $exists: true }
        }
    }
]
db.personnes.aggregate(pipeline)
```

### L’opérateur $addFields

permet d’ajouter des champs dans le retour de la requête

Example: 

```jsx
db.personnes.aggregate({
	$addFields: {
		"numero_secu": ""
	}
})
/// renvoie la liste des document avec un nouveau champ "numero_secu"
```

### L’opérateur $group

Synthaxe

```jsx
{
	$group: {
		"_id": <expression>,
		<champ>: { <operateur d'accumulation> }
	}
}
```

### Les opérateurs d’accumulaation

- $push
- $sum
- $avg
- $min
- $max

```jsx
const pipeline = [{
	$group: {
		"_id": "$age",
		"nombre_personnes": { $sum: 1 }
	}
}]
db.personnes.aggregate(pipeline)
```

L’opérateur $sortByCount permet de faire ce que fait la pipeline ci dessus

```jsx
const pipeline = [{
	$group: {
		"_id": null, /// Ne filtre pas les éléments de la collection
		"nombre_personnes": { $sum: 1 }
	}
}]
db.personnes.aggregate(pipeline)
```

Par exemple celà nous permets de calculer l’age moyen

```jsx
const pipeline = [
	{
		$match: {
			"age": { $exists: 1 }
		}
	},
	{
		$group: {
			"_id": null,
			"avg": {
				$avg: "$age"
			}
		}
	},
	{
		$project: {
			"_id": 0,
			"Age_moyen": {
				$ceil: "$avg"
			}
		}
	}
];
db.personnes.aggregate(pipeline)
```

# Exercices

- Questions posé en cour
    
    Afficher les comptes dont la sommes des opérations de débit est supérieur au montant du crédit.
    
    Commande correspondantes :
    
    Ce que j’avais fait (erreur synthaxe pour le contenu de $sum)
    
    ```jsx
    db.banque.find({
    	"credit": { $exists:  1 },
    	$expr: {
    		$gt: [
    			{
    				$sum: { "$debit" }
    			},
    			"$credit"			
    		]
    	}
    })
    ```
    
    Correction :
    
    ```jsx
    db.banque.find({
    	"debit": { $exists:  1 },
    	$expr: {
    		$gt: [
    			{
    				$sum: "$debit"
    			},
    			"$credit"			
    		]
    	}
    })
    ```
    
    Utiliser l’opérateur $push pour ajouter une passion à Yves:
    
    Commande correspondante :
    
    Ce que j’avais fait: (il est préférable d’utiliser l’Id pour cibler un document précis)
    
    ```jsx
    db.hobbies.updateOne(
    	{
    		"nom": "Yves"
    	},
    	{
    		$addTo: {"passions": "Tekken 7"}
    	}
    )
    ```
    
    Correction:
    
    ```jsx
    db.hobbies.updateOne(
    	{
    		"_id": 1
    	},
    	{
    		$push: {"passions": "Tekken 7"}
    	},
    )
    ```
    
    Utiliser $pull
    
    ```jsx
    db.hobbies.updateOne(
    	{
    		"_id": 3
    	},
    	{
    		$pull: {"passions": "Parrachute"}
    	},
    )
    ```
    
    Que ce passe t’il dans cette requête
    
    ```jsx
    db.achats.aggregate([{
    	$addFields: {
    		"total_achats": {
    			$sum: "$achats"
    		}, "total_reduc": {
    			$sum: "$reductions"
    		}
    	}
    }, {
    	$addFields: {
    		"total_final": {
    			$subtract: ["$total_achats", "$total_reduc"]
    		}
    	}
    }, {
    	$project: {
    		"_id": 0, "nom": 1, "prenom": 1, "Total payé": {
    			$divide: [{
    				$subtract: [{ $multiply: ['$total_final', 100] }, {
    					$mod: [{ $multiply: ['$total_final', 100] }, 1]
    				}]
    			}, 100]
    		}
    	}
    }])
    ```
    
    Le premier $addFields va ajouter deux champs, le premier correspond au nombre total d’achat et le second correspond au nombre total de réduction, les deux sont calculés grâce à l’opérateur $sum
    
    Le second $addFields va ajouter un champ total final, qui va correspondre au total des achats moins le nombre de réduction, ce qui correspond probablement au prix final à payer.
    
    Pour terminer, l’opérateur $Project va faire la projection des éléments à renvoyé. Ici le champs _id sera masqué, le champ nom prenom sera afficher, et le champ “Total payé” sera affiché avec une valeur arrondi au centième près
    
- Questions du fichier exoBook.md
    
    
    Pour créer une base de données nommée "sample_db" on utilise la commande `use sample_db`, cette commande ne créera pas la base de donnée immédiatement mais lors de l’ajout d’une preimère collection.
    
    Pour ajouter la collection “employees”, on utilise la commande
    
    ```jsx
    db.createCollection("employees")
    ```
    
    Résultat de ces deux commandes :
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/354da0f1-b598-4608-b260-d5e3af9f072e/Untitled.png)
    
     
    
    Pour insérer un document dans une collection, on utilise la commande 
    
    ```jsx
    db.employees.insertOne({ ObjectProperties })
    ```
    
    Dans notre cas, il faut insérer plusieurs éléments, il faut donc utilisé 
    
    ```jsx
    db.employees.insertMany([
    	{ ObjectProperties1 },
    	{ ObjectProperties2 }
    ])
    ```
    
    ce qui donne :
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7b8f0bdb-8914-40d8-af79-9f88dd034136/Untitled.png)
    
    Pour trouver tous les documents dans la collection “employees”, on utilise la requête : 
    
    ```jsx
    db.employees.find()
    ```
    
    Ce qui donne :
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/73d1b809-1936-41c6-89f8-e65d957bf8a4/Untitled.png)
    
    Pour récupérer les document avec la propriété âge supérieur à 33, on ajoute des secondes options à la méthode find, voici la requête correspondante :
    
    ```jsx
    db.employees.find(
    	{
    		"age":
    			{
    				$gt: 33
    			}
    	}
    )
    ```
    
    Ce qui donne :
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a22bd61e-c6f5-4795-af28-5b6b22faf7bb/Untitled.png)
    
    Ici on voit que les seuls employés renvoyé ont un âge supérieur à 33 ans.
    
    Pour trier les employés par leur salaire dans l’ordre décroissant, on pourrait utiliser l’option `$orderby`, mais cette fonction est déprécié depuis la version 3.12 de Mongo, il faut donc utiliser la méthode `find()`, voici la requête correspondante :
    
    ```jsx
    db.employees.find().sort(
    	{
    		"salary": -1
    	}
    )
    ```
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4245ba55-0f12-4bc7-8d38-45ce8ec1960c/Untitled.png)
    
    Pour récupérer uniquement certains champ d’un document, il faut utiliser une projection, dans laquel il faut spécifier les champs à inclure ou exclure dans la réponse de la requête.
    
    Dans notre cas nous voulons uniquement récupérer le name et le job de chaque employés, voici un exemple de requête permettant ce résultat :
    
    ```jsx
    db.employees.find(
    	{},
    	{
    		"name": true,
    		"job": true,
    		"_id": false
    	}
    )
    ```
    
    Requête et réponse de cette requête :
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1be08555-f662-43bf-8fdd-0b1022708510/Untitled.png)
    
    Ici le champ `"_id"` est passé à false, car il est toujours renvoyé par défaut dans la réponse, or ici nous ne voulons pas le récupérer.
    
    Pour compter le nombre d’employer par poste, une des possibiliter est d’utiliser la méthode `aggregate`, comme ceci :
    
    ```jsx
    db.employees.aggregate([
    	{
    		$group: {
    			_id: '$job',
    			itemCount: { 
    				$count: {} 
    			}
    		}
    	}
    ])
    ```
    
    Résultat de cette requête :
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e4e7ef09-c3cd-46be-b154-8f8b5bb66399/Untitled.png)
    
    Ici on retrouve bien le nombre d’employé par job
    
    Pour mettre à jour les champs des documents dans la collection, nous allons utiliser la méthode `update()`
    
    L’objectif de la requête va être de mettre a jour la propriété salaire des documents dans la collection employees à 80000, voici la requête permettant ceci :
    
    ```jsx
    db.employees.updateMany(
    	{},
    	{
    		$set:{
    			'salary': 80000
    		}
    	}
    )
    ```
    
    Résultat de cette requête avec la récupération de tous les documents afin de s’assurer que les champs ont bien été modifié :
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/517bb8cd-9968-45b4-b3cf-a48a99c87e02/Untitled.png)
    
- Questions du fichier exo.md
    
    Question 1 :
    
    ```jsx
    db.salles.find(
    	{
    		"smac": true
    	},
    	{
    		"nom": 1
    	}
    )
    ```
    
    Ici je recherche les document ayant le champ “smac” à `true`, puis je fais une projection pour retourner uniquement l’id et le nom de la salle.
    
    Question 2 :
    
    ```jsx
    db.salles.find(
    	{
    		"capacite": { $gt: 1000 }
    	},
    	{
    		"nom": 1
    	}
    )
    ```
    
    Ici j’utilise l’opérateur $gt pour récupérer le nom des salles avec une capaciter supérieur à 1000.
    
    Question 3 :
    
    ```jsx
    db.salles.find(
    	{
    		"adresse.numero": { $exists: 0 }
    	},
    	{
    		"_id": 1
    	}
    )
    ```
    
    Ici j’utilise l’opérateur $exists pour tester si le champ recherché existe.
    
    Question 4 :
    
    ```jsx
    db.salles.find(
    	{
    		"avis": {$size: 1}
    	},
    	{
    		"nom": 1
    	}
    )
    ```
    
    Ici l’opérateur $size permet de récupérer la taille du document courrant.
    
    Question 5 :
    
    ```jsx
    db.salles.find(
    	{
    		"styles": { $in: ["blues"] }
    	},
    	{
    		"styles": 1
    	}
    )
    ```
    
    Ici l’opérateur $in va permettre de chercher si un terme est contenu dans la liste du document courrant.
    
    Question 6 :
    
    ```jsx
    db.salles.find(
    	{
    		"styles.0": "blues"
    	},
    	{
    		"styles": 1
    	}
    )
    ```
    
    Ici j’utilise l’index du tableau 0 pour récupérer tous les document avec le style “blues” au premier emplacement de la liste.  
    
    Question 7 :
    
    ```jsx
    db.salles.find(
    	{
    		"adresse.codePostal": { $regex: "^[0-9][0-9].*" }
    	},
    	{
    		"nom": 1
    	}
    )
    ```
    
    Ici l’expression régulière permet de récupérer les salles avec un code postal commençant par deux nombre.
    
    Question 8 :
    
    ```jsx
    db.salles.find({
    	$or: [
    		{ "_id": {
    			$mod: [2, 0] }
    		},
    		{ "avis": {
    			$exists: 0 }
    		}
    	]
    },
    {
    	"nom": 1
    })
    ```
    
    L’opérateur $or permet de récupérer des document selon plusieurs condition si l’une d’elle est remplie, contrairement à $and, qui néccessite de remplir toutes les conditions pour renvoyer le document courant.
    
    Question 9 :
    
    ```jsx
    db.salles.find(
    	{
    		"avis": {
    			$elemMatch: {
    				"note": { $gte: 8, $lte: 10 }
    			}
    		}
    	},
    	{
    		"nom": 1
    	}
    )
    ```
    
    Ici l’opérateur $elemMatch va chercher si un élément dans la liste répond true à la condition $gte et $lte de note.
    
    Question 10 :
    
    ```jsx
    const dateNow = new Date('2019 11 15')
    db.salles.find(
    	{
    		"avis": {
    			$elemMatch: {
    				"date": { $gte: dateNow }
    			}
    		}
    	},
    	{
    		"nom": 1
    	}
    )
    ```
    
    Ici j’utilise l’objet `Date` de JavaScript pour faire une comparaison entre 2 dates.
    
    Question 11 :
    
    ```jsx
    db.salles.find({
    	$expr: {
    		$gt: [
    			{
    				$multiply: [
    					"$_id",
    					100
    				]
    			},
    			"$capacite"
    		]
    	}
    },
    {
    	"nom": 1,
    	"capacite": 1
    })
    ```
    
    Ici l’opérateur $multiply permet d’effectuer un calcul sur la propriété _id.
    
    Question 12 :
    
    ```jsx
    db.salles.find({ $where: "this.styles.lenght > 2" })
    ```
    
    ```jsx
    db.salles.find(
    	{
    		"smac": true,
    		$where: function() {
    		this.styles.lenght > 2;
    	}
    	},
    	{
    		"nom": 1
    	}
    )
    ```
    
    Question 13 :
    
    ```jsx
    db.salles.distinct("adresse.codePostal")
    ```
    
    La méthode `distinct()` permet de récupérer les codes postaux de manière distincte.
    
    Question 14 :
    
    ```jsx
    db.salles.updateMany({},
    	{
    		$inc: { "capacite" : 100}
    	}
    )
    ```
    
    Ici l’operator permet d’incrémenté le champ capacite en lui ajoutant 100.
    
    Question 15 :
    
    ```jsx
    db.salles.updateMany({},
    	{
    		$addToSet: {"styles": "jazz"}
    	}
    )
    ```
    
    Ici l’opérateur $addToSet permet d’ajouter la valeur “jazz” uniquement dans la liste de styles des documents qui ne poccède pas déjà la valeur “jazz”
    
    Question 16 :
    
    ```jsx
    db.salles.updateMany(
    	{
    		$and: [
    			{ 
    				"_id": 
    					{
    						$ne: 2 
    					}
    			},
    			{ 
    				"_id": 
    					{
    						$ne: 3
    					}
    			},
    		]
    	},
    	{
    		$pull: 
    		{
    			"styles":"funk"
    		}
    	}
    )
    ```
    
    Ici l’operateur $and permet de ne pas récupérer les document ayant l’id 2 et 3, puis l’opérateur $pull va supprimer dans les documents récupérer la valeur “funk” si elle est présente dans la liste “style”
    
    Question 17 : 
    
    ```jsx
    db.salles.updateMany(
    	{
    		"_id": 
    		{
    			$eq: 3
    		}
    	},
    	{
    		$addToSet:
    		{
    			"styles":
    			{
    				$each: ["techno", "reggae"]
    			}
    		}
    	}
    )
    ```
    
    Ici l’opérateur $each permet d’ajouter dans la liste style chaque élément de la liste qui lui est passé.
    
    Question 18 :
    
    ```jsx
    db.salles.updateMany(
    	{
    		"nom": { $regex: "^[pP].*" }
    	},
    	{
    		$inc: { "capacite" : 150},
    		$set: { "contact": { "telephone": "04 11 94 00 10" }}
    	}
    )
    ```
    
    Question 19 :
    
    ```jsx
    const currentDate = new Date()
    db.salles.updateMany(
    	{
    		"nom": { $regex: "[^aeiou]+$" }
    	},
    	{
    		$push: {
    			"avis": { "date": currentDate, "note": 10 } 
    		}
    	},
    )
    ```
    
    Question 20 :
    
    ```jsx
    db.salles.update(
    	{
    		"nom": { $regex: "^[zZ].*" }
    	},
    	{
    		$set: {
    			"nom": "Pub Z",
    			"capacite": NumberInt(50),
    			"smac": false
    		}
    	},
    	{
    		$upsert: true
    	}
    )
    ```
    
    Question 21 :
    
    ```jsx
    db.salles.aggregate(
    	[
    		{
    			$match: {
    				"_id": { $type: "objectId" }
    			}
    		},
    		{
    			$count: "document objectId"
    		}
    	]
    )
    ```
    
    L’opérateur type permet d’effectuer des vérification sur le type de la valeur.
    
    Question 22 :
    
    ```jsx
    db.salles.aggregate(
    	[
    		{
    			$match: {
    				"_id": {
    					$not: {
    						$type: "objectId"
    					}
    				}
    			}
    		},
    		{
    			$sort: { "capacite": -1 }
    		},
    		{
    			$limit: 1
    		},
    		{
    			$project: {
    				"_id": 0,
    				"nom": 1
    			}
    		}
    	]
    )
    ```
    
    Question 23 :
    
    ```jsx
    db.salles.update(
    	{
    		"_id": 4
    	},
    	{
    		$set: {
    			"nom": "Nom de salle",
    			"capacite": NumberInt(60)
    		}
    	}
    )
    ```
    
    Question 24 :
    
    ```jsx
    db.salles.deleteOne(
    	{
    		"_id": { $type: 'objectId' },
    		"capacite": { $lte: 60 }
    	}
    )
    ```
    
    Question 25 :
    
    ```jsx
    db.salles.findOneAndUpdate(
    	{
    		"adresse.ville": 'Nîmes'
    	},
    	{
    		$set: { "capacite": 15 }
    	}
    )
    ```
    
- Questions du fichier index.md
    
    Question 1
    
    ```jsx
    db.salles.find({"capacite": {$gt: 500}, "adresse.codePostal": /^30/}) 
    db.salles.find({"adresse.codePostal": /^30/, "capacite": {$lte: 400}})
    ```
    
    Si ces deux requêtes sont faites fréquemment, je pense qu’utiliser un index sur le champs capaciter serais le plus adapté, car une expression régulière est utilisé pour trouver le code postal alors que l’opérateur $gt est utiliser pour filtrer la capacité, ce qui me semble moins performant, c’est pour celà que ce champ me semble plus adapter.
    
    Mais il est également possible de créer un index sur chacun d’entre eux.
    
    Voici comment créer un index nommer “index_capaciter”
    
    ```jsx
    db.personnes.createIndex(
    	{
    		"capacite": 1
    	},
    	{
    		"name": "index_capaciter"
    	}
    )
    ```
    
    Pour supprimer un index, il faut utiliser la méthode `dropIndex()`, en passant le nom de l’index à supprimer.
    
    Code correspondant 
    
    ```jsx
    db.personnes.dropIndex("index_capaciter")
    ```
    
- Questions du fichier aggregation.md
    
    Question 1
    
    ```jsx
    const pipeline = [
    	{
    		$match: {
    			"capacite": { $gt: 50 }
    		}
    	},
    	{
    		$project: {
    			"_id": 0,
    			"ville": "$adresse.ville",
    			"grande": {
    				$gt: ["$capacite", 1000]
    			}
    		}
    	}
    ];
    db.salles.aggregate(pipeline)
    ```
    
    Question 2 :
    
    ```jsx
    const pipeline = [
    	{
    		$match: {
    			"capacite": { $exists: true }
    		}
    	},
    	{
    		$project: {
    			"_id": 0,
    			"nom": 1,
    			"avant_extension": "$capacite",
    			"apres_extension": {
    				$sum: ["$capacite", 100]
    			}
    		}
    	}
    ];
    db.salles.aggregate(pipeline)
    ```
    
    Question 3 :
    
    ```jsx
    const pipeline = [
    	{
    		$match: {
    			"adresse.codePostal": { $exists: true }
    		}
    	},
    	{
    		$sort: { "adresse.codePostal": 1 }
    	},
    	{
    		$project: {
    			"_id": 0,
    			"departement": "$adresse.codePostal",
    			"capacite": "$capacite",
    		}
    	}
    ];
    db.salles.aggregate(pipeline)
    ```
    
    Question 4 :