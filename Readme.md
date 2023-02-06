# Mongo DB

À faire : 

- Comment renommer une collection
- Expliquer la méthode `findAndModify`
- Lever une exception grâce aux validateurs

Collection : ensemble de documents (Equivalent d’une table dans une bdd)

Document : ensemble de donnée clé-valeur, le schéma d’un document est dit “dynamique” 

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
            
    

# Exercices

Question du fichier exoBook.md

Si les images ne s'affiche pas correctement : [View on Notion](https://www.notion.so/Mongo-DB-548c9379efcc495390dda8e8c907ddaa)

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