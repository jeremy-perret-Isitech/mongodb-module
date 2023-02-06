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
    
    Pour créer une base de données nommée "sample_db" on utilise la commande `use sample_db`, cette commande ne créera pas la base de donnée immédiatement mais lors de l’ajout d’une preimère collection.
    
    Pour ajouter la collection “employees”, on utilise la commande `db.createCollection(”employees”)`
    
    Résultat de ces deux commandes :
    
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/354da0f1-b598-4608-b260-d5e3af9f072e/Untitled.png)