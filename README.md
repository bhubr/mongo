# MongoDB

MongoDB est un type de base de données fondamentalement différent de MySQL. La [page Wikipédia de MongoDB](https://fr.wikipedia.org/wiki/MongoDB) est un bon point d'entrée.

Les bases de données SQL sont des bases de données dites *relationnelles*, car chaque type de donnée est stocké dans une table qui lui est propre, et les tables peuvent avoir des *relations* entre elles.

Prenons par exemple la BDD d'une application de blog : une table `article` peut avoir une clé étrangère
`authorId` qui référence l'`id` d'un utilisateur stocké dans la table `user`.

De plus, en SQL, les champs d'une entrée dans une table sont "contraints" par la structure de la table.
On définit les champs et leurs valeurs possibles quand on crée une table, et chaque enregistrement ne peut
avoir que les champs ainsi définis.

Rien de tel dans MongoDB. MongoDB est une base "orientée documents", de la famille "NoSQL".

Avec MongoDB, on parle de collection plutôt que de table. Une collection est un ensemble de documents apparentés, et il peut y avoir plusieurs collections dans une base (comme il peut y avoir plusieurs tables
dans une base MySQL). Une entrée d'une collection s'appelle un document, et contrairement aux bases SQL, *rien n'oblige à structurer tous les documents d'une collection de la même façon*.

## Installation / Lancement

Suivre les instructions sur la [page MongoDB du Wiki Ubuntu-fr](https://doc.ubuntu-fr.org/mongodb).

Pour lancer des requêtes via le terminal, l'équivalent de la commande `mysql` est la commande `mongo`.

    wilder@ubuntu:~$ mongo

## Commandes de base

### Obtenir de l'aide

Il suffit de saisir :

    help

### Créer une base / changer de base

La même commande sert pour créer de base ou basculer vers une autre base. Pour créer la base `myapp`
ou se placer dedans :

    use myapp

### Créer une collection

Pour créer une collection `person`, afin d'y stocker des données de profils utilisateurs :

    db.createCollection("person")

## Requêter une collection

### Créer un document

On utilise `db.nomDeLaCollection.insert()`. Dans les parenthèses de `insert()`, on met... du JSON (à ceci près qu'on n'a pas besoin d'entourer les clés avec des `"`). Exemples de créations de personnes :

    db.person.insert({ firstName: "Jeannot", lastName: "Lapin", age: 10, nationality: "fr", hobbies: ["Eat carrots", "Chill out"] })
    db.person.insert({ firstName: "Bob", lastName: "Sponge", age: 25, hobbies: ["Scuba Diving"] })

Ici, j'ai deux documents qui présentent une structure similaire, mais par exemple, je n'ai pas donné de champ `nationality` à Bob l'Eponge. En poussant les choses un peu plus loin, j'aurais même pu lui donner une structure radicalement différente (même si, en pratique, c'est a priori rare de regrouper dans la même collection des documents dont la structure est totalement différente).

MongoDB donne a chaque document une clé unique, appelé `_id`, qui, contrairement à ce qui se fait avec `MySQL`, n'est pas un nombre entier, mais une chaîne hexadécimale (nombres 0-9 et lettres a-f), générée aléatoirement, et très longue, de façon à éviter d'obtenir deux clés identiques.

### Retrouver un / des document(s)

On va d'abord ajouter quelques autres documents à notre collection `person` :

    db.person.insert({ firstName: "Mary", lastName: "Poppins", age: 37, nationality: "uk", hobbies: ["Fly with an umbrella"], passport: "925665416" })
    db.person.insert({ firstName: "Bob", lastName: "Poppins", age: 39, nationality: "uk", hobbies: ["Fly in the sky with Mary"], passport: "987439032" })
    db.person.insert({ firstName: "Clark", lastName: "Kent", age: 32, nickName: "Superman", nationality: "us", hobbies: ["Get dressed with a blue outfit and fly"] })

On utilise `db.nomDeLaCollection.find()`. Dans les parenthèses de `find()`, on met des critères de recherche, et là aussi, le "langage de requêtes" (query language) est proche du JSON. Si on veut retrouver toutes les personnes dont le nom de famille est "Lapin" :

    db.person.find({ lastName: "Lapin" })

On obtient alors :

    { "_id" : ObjectId("5bf39912f0ad453292d6f1e6"), "firstName" : "Jeannot", "lastName" : "Lapin", "age" : 10, "nationality" : "fr", "hobbies" : [ "Eat carrots", "Chill out" ] }

Pour retrouver toutes les personnes dont le prénom est "Bob" :

    db.person.find({ lastName: "Bob" })

On obtient :

    { "_id" : ObjectId("5bf39912f0ad453292d6f1e7"), "firstName" : "Bob", "lastName" : "Sponge", "age" : 25, "hobbies" : [ "Scuba Diving" ] }
    { "_id" : ObjectId("5bf3991df0ad453292d6f1e9"), "firstName" : "Bob", "lastName" : "Poppins", "age" : 39, "nationality" : "uk", "hobbies" : [ "Fly in the sky with Mary" ], "passport" : "987439032" }

### Requêtes un peu plus complexes

On peut faire l'équivalent des `IN`, des `AND` et des `OR` de SQL avec MongoDB. Cela se fait avec des opérateurs.

La syntaxe paraît un peu complexe par rapport à MySQL, mais il ne faut pas oublier que les requêtes, comme les documents, s'écrivent comme du JSON ou des objets JavaScript.

#### Opérateur `$in`

Utilisé pour faire une requête sur un champ particulier, et donner un tableau de valeurs possibles pour ce champ.

    db.collection.find({ champ: { $in: ["valeur1", "valeur2", "valeur3" ]} })

Par exemple, pour retrouver toutes les personnes dont la nationalité est "uk" ou "us" :

    db.person.find({ nationality: { $in: ["uk", "us"] } })

On obtient :

    { "_id" : ObjectId("5bf3991df0ad453292d6f1e8"), "firstName" : "Mary", "lastName" : "Poppins", "age" : 37, "nationality" : "uk", "hobbies" : [ "Fly with an umbrella" ], "passport" : "925665416" }
    { "_id" : ObjectId("5bf3991df0ad453292d6f1e9"), "firstName" : "Bob", "lastName" : "Poppins", "age" : 39, "nationality" : "uk", "hobbies" : [ "Fly in the sky with Mary" ], "passport" : "987439032" }
    { "_id" : ObjectId("5bf3991df0ad453292d6f1ea"), "firstName" : "Clark", "lastName" : "Kent", "age" : 32, "nickName" : "Superman", "nationality" : "us", "hobbies" : [ "Get dressed with a blue outfit and fly" ] }

#### Opérateur `$or`

Utilisé pour faire une requête en acceptant les champs validant une condition ou une autre.

Par exemple, pour retrouver toutes les personnes dont la nationalité est "fr" *ou* dont le prénom est "Clark":

    db.person.find({ $or: [{ nationality: "fr" }, { firstName: "Clark" }] })

On obtient :

    { "_id" : ObjectId("5bf39912f0ad453292d6f1e6"), "firstName" : "Jeannot", "lastName" : "Lapin", "age" : 10, "nationality" : "fr", "hobbies" : [ "Eat carrots", "Chill out" ] }
    { "_id" : ObjectId("5bf3991df0ad453292d6f1ea"), "firstName" : "Clark", "lastName" : "Kent", "age" : 32, "nickName" : "Superman", "nationality" : "us", "hobbies" : [ "Get dressed with a blue outfit and fly" ] }

#### Opérateur `$and`

Pour retrouver toutes les personnes dont le prénom est "Bob" et dont le champ `nationality` existe (dans cet exemple on introduit aussi l'opérateur [`$exists`](https://docs.mongodb.com/manual/reference/operator/query/exists/) qui permet de vérifier l'existence d'un champ) :

    db.person.find({ $and: [{ firstName: "Bob" }, { nationality: { $exists: true } }] })

On obtient (Bob l'Eponge a été éliminé des résulats car n'a pas de champ `nationality`) :

    { "_id" : ObjectId("5bf3991df0ad453292d6f1e9"), "firstName" : "Bob", "lastName" : "Poppins", "age" : 39, "nationality" : "uk", "hobbies" : [ "Fly in the sky with Mary" ], "passport" : "987439032" }

#### Opérateurs de comparaison : `$lt`, `$lte`, `$gt`, `$gte`

Ce sont des opérateurs de comparaison numérique :

* `$lt` : **l**ess **t**han (inférieur à)
* `$lte` : **l**ess **t**han or **e**quals (inférieur ou égal à)
* `$gt` : **g**reater **t**han (supérieur à)
* `$gte` : **g**reater **t**han or **e**quals (supérieur ou égal à)

Pour trouver toutes les personnes dont l'âge est inférieur ou égal à 32 :

    db.person.find({ age: { $lte: 32 } })

On obtient :

    { "_id" : ObjectId("5bf39912f0ad453292d6f1e6"), "firstName" : "Jeannot", "lastName" : "Lapin", "age" : 10, "nationality" : "fr", "hobbies" : [ "Eat carrots", "Chill out" ] }
    { "_id" : ObjectId("5bf39912f0ad453292d6f1e7"), "firstName" : "Bob", "lastName" : "Sponge", "age" : 25, "hobbies" : [ "Scuba Diving" ] }
    { "_id" : ObjectId("5bf3991df0ad453292d6f1ea"), "firstName" : "Clark", "lastName" : "Kent", "age" : 32, "nickName" : "Superman", "nationality" : "us", "hobbies" : [ "Get dressed with a blue outfit and fly" ] }

