# Examens TP

## Préparation des données

J’ai utilisé un jeu de donnée recensant les restaurants de la ville de Nice, trouvé sur le site [opendata.nicecotedazure.org](http://opendata.nicecotedazur.org/data/dataset/liste-des-restaurants-de-nice-geolocalises) 

Cette semaine, j’ai travaillé en hébergent la base de donnée MongoDB via un container docker ouvert sur le port 27017, mais pour ce TP j’ai eu besoin d’utiliser la commande mongoimport, pour importer les données, mais elle ne semblais pas fonctionner pour la base sur docker (l’avancement de l’importation restais bloqué à 0%), j’ai donc hébergé la base sur Atlas.

J’ai créer une base de donnée `restaurant`, contenant une collection `restaurant_data`, dans laquel j’ai importé le jeu de donnée grâce à la commande `mongoimport` cité ci-dessus.

Voici la commande utilisé :

```bash
mongoimport 
	--uri 'mongodb+srv://admin:admin@cluster0.rpjxyon.mongodb.net/restaurant'
	--collection 'restaurant_data' 
	--file 'restaurant.json' 
	--jsonArray
```

## Les requêtes MongoDB

a- Voici une commande permettant de récupérer la liste des restaurants étant ouvert à partir de 18h00

```bash
const projection = {
    "_id": 0,
    "nom.libelleFr": 1,
    "presentation.descriptifCourt.libelleFr": 1,
    "ouverture.periodesOuvertures": 1,
    "ouverture.horaireFermeture": 1
}
db.restaurant_data.find({
    $and: [
        {
            "ouverture.periodesOuvertures.horaireOuverture":
                { $exists: true }
        },
        {
            "ouverture.periodesOuvertures.horaireFermeture":
                { $exists: true }
        },
        {
            "ouverture.periodesOuvertures.horaireOuverture":
                { $gte: "18:00:00" }
        },
    ]

}
)
```

j’ai utiliser l’opérateur $exists, car pour certains restaurant les périodes d’ouvertures et de fermetures ne sont pas renseigné.

b- Voici une requête qui permet de lister les restaurant par notes, du meilleur au moins bon

```bash
const projection = {
    "_id": 0,
    "nom.libelleFr": 1,
    "presentation.descriptifCourt.libelleFr": 1,
    "metadonnees.contenus.metadonnee": 1
}
db.restaurant_data.find({
    "metadonnees.0.contenus.0.metadonnee":
        { $exists: true }
}, projection
).sort({"metadonnees.contenus.metadonnee.note": -1})
```

## L’indexation avec MongoDB

a - Voici la commande permettant de créer un index sur le champs de l’heure d’ouverture des restaurants

```bash
db.restaurant.createIndex(
	{
		"ouverture.periodesOuvertures": 1
	},
	{ 
		"name": "heureOuvertureIndex",
		"background": true
	}
)
```

b - Puis je vérifie que l’index à bien été créer, en affichant la liste des index de la collection grâce à la méthode `getIndexes()`

Ce qui donne :

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f78fd50b-9cfd-4d49-9077-bd5f4ae39791/Untitled.png)

Ici ont voit bien que l’index `heureOuvertureIndex`, pour la clé `ouverture.periodesOuvertures` à été créé.

## Requêtes géospatiales

a - Voici une requête permettant de récupérer la liste des restaurant à moins de 2 kilomètres à la ronde du domaine de Gairaut (aux coordonnées [7.26539, 43.74295]).

Tout d’abord il faut créer un index sur le champ de coordonné dans les document avec cette commande

```jsx
db.restaurant_data.createIndex({"localisation.geolocalisation.geoJson": "2dsphere" })
```

Requête pour récupérer les restaurants à moins de 2 kilomètres.

```jsx
const projection = {
    "_id": 0,
    "nom.libelleFr": 1,
    "presentation.descriptifCourt.libelleFr": 1,
    "localisation.geolocalisation.geoJson": 1
}

const currentPosition = [7.26539, 43.74295]

db.restaurant_data.find({
    "localisation.geolocalisation.geoJson": {
        $nearSphere: {
            $geometry: {
                type: "Point",
                coordinates: currentPosition
            },
            $minDistance: 0,
            $maxDistance: 2000
        }
    }
}, projection)
```

Dans le fichier `function.js` j’ai créé un petit script pour récupérer toutes les coordonnées des restaurants, afin de les afficher sur une carte.

La carte représente les restaurant de la base de donnée, ainsi que la zone couverte par la recherche.

Sur la carte il y a quatres restaurants dans la zone, comme pour la réponse de la requête.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/77a95e1d-422f-41fc-b32c-2aa4aac7159b/Untitled.png)

b - Voici un bout de code qui permet de faire des recherches avec :

- la géolocalisation
- une distance minimal
- unde distance maximal

Ces données sont stocker dans des variables afin d’êtres modifier au besoin par le programme;

Voici le code et le requête permettant celà :

```jsx
const projection = {
    "_id": 0,
    "nom.libelleFr": 1,
    "presentation.descriptifCourt.libelleFr": 1,
    "localisation.geolocalisation.geoJson": 1
}

let coordinates = [7.26539, 43.74295]
let minDistance = 0;
let maxDistance = 3000;

db.restaurant_data.find({
    "localisation.geolocalisation.geoJson": {
        $nearSphere: {
            $geometry: {
                type: "Point",
                coordinates: currentPosition
            },
            $minDistance: minDistance,
            $maxDistance: maxDistance
        }
    }
}, projection)
```

## Framework d'agrégation

a- J’ai créé un petit script pour générer une liste de `avisClients`, afin de pouvoir calculer la moyenne de tous les commentaire grâce à une requête.

Voici une requête avec la méthode `aggregate` qui permet celà :

```jsx
const pipeline = [
	{
		$match: {
			"metadonnees.contenus.metadonnee.avisClients": { $exists: 1 }
		}
	},
	{
		$project: {
			"_id": 0,
	    "nom.libelleFr": 1,
	    "presentation.descriptifCourt.libelleFr": 1,
			"noteMoyenne": { $avg: "$metadonnees.contenus.metadonnee.avisClients" }
		}
	}
];
db.restaurant_data.aggregate(pipeline)
```

b- Voici une requête permettant de trier les restaurant par note et par nombre d’avis

```jsx
const pipeline = [
    {
        $match: {
            "metadonnees.contenus.metadonnee.note": { $exists: 1 }
        }
    },
    {
        $sort: {
            "metadonnees.contenus.metadonnee.nombreAvis": -1
        }
    },
    {
        $sort: {
            "metadonnees.contenus.metadonnee.note": -1
        }
    },
    {
        $project: {
            "_id": 0,
            "nom.libelleFr": 1,
            "presentation.descriptifCourt.libelleFr": 1,
            "metadonnees.contenus.metadonnee.note": 1,
            "metadonnees.contenus.metadonnee.nombreAvis": 1
        }
    }
];
db.restaurant_data.aggregate(pipeline)
```

Voici la comande permettant d’exporter la base de donnée au format csv :

```bash
sudo mongoexport --uri 'mongodb+srv://admin:admin@cluster0.rpjxyon.mongodb.net/restaurant' --collection 'restaurant_data' --type 'csv' --fields='.' --out '/opt/export/restaurant_data_export.csv'
```

x