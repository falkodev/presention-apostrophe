# Utilisation

Rien de mieux qu'une petite démo pour montrer la prise en main d'Apostrophe. Pour cela, il faut installer quelques outils tel qu'expliqué dans le tutoriel sur le site officiel : [http://apostrophecms.org/docs/tutorials/getting-started/setting-up-your-environment.html](http://apostrophecms.org/docs/tutorials/getting-started/setting-up-your-environment.html)

A noter que des collègues sur Windows ont eu des difficultés avec Node. Dans la mesure du possible, un MacOs ou un Linux est préférable. D'une manière générale, suivre le tutoriel officiel d'Apostrophe constitue une bonne introduction aux possibilités du framework. Toutefois, plusieurs personnes autour de moi \(ainsi que moi-même\) ont trouvé ce tutoriel un peu confus. C'est pour clarifier cela que j'ai décidé de créer cet article.

Nous allons créer un projet test tel que décrit ici : [http://apostrophecms.org/docs/tutorials/getting-started/creating-your-first-project.html](http://apostrophecms.org/docs/tutorials/getting-started/creating-your-first-project.html)

Pour démarrer l'application, il faut lancer une ligne de commande : `node app.js` ou `npm start`\(puisque "npm start" lance en fait "node app.js"\). Mais comme à chaque modification d'un fichier, il faudra couper l'appli et relancer la ligne de commande. Pour pallier ce problème, j'aime bien installer nodemon, qui s'occupera de surveiller les modifications sur des types de fichier précis, et redémarrera l'appli comme un grand.

`npm install nodemon --save-dev`ou `yarn add nodemon --dev` pour ceux qui utilisent déjà yarn sur d'autres projets \(ce que je préfère\).

On peut ensuite modifier la ligne "start" dans package.json de la manière suivante :

```
"scripts": {
    "start": "nodemon --watch app.js --watch lib/ --ext html,js app.js"
  },
```

On dit à nodemon de surveiller le fichier app.js \(qui est le fichier principal d'Apostrophe pour charger les configurations\) et les modifications sur des fichiers dans le dossier "lib" avec les extensions .html et .js. Pas besoin de surveiller les fichiers .less, les modifications seront prises en compte avec un simple refresh du navigateur.

On peut désormais relancer `npm start`

Je propose de créer une application de gestion de produits. Le but sera de lister les produits dans une première page, puis d'afficher le détail d'un produit dans une seconde. En tant que développeur, on laissera la main à un futur auteur d'ajouter, de modifier, de supprimer des produits, et même de créer des pages et d'y ajouter du contenu texte, image ou vidéo.

Pour démarrer il nous faut de la donnée.

Créons un dossier "data" dans lib, et ajoutons-y un fichier data.json avec ces données : [https://raw.githubusercontent.com/mongodb/docs-assets/primer-dataset/primer-dataset.json](https://raw.githubusercontent.com/mongodb/docs-assets/primer-dataset/primer-dataset.json) \(il s'agit de la base de test de MongoDB qui contient une liste de restaurants\). Toujours dans ce dossier "data", nous allons créer un fichier index.js dont la fonction sera de parser les données pour l'insérer en base.

Pour visualiser l'état de notre base, personnellement j'utilise 3T Studio, mais n'importe quel client Mongo fera l'affaire, même le shell de base.

Avec Apostrophe, les données sont regroupées dans la même collection : aposDocs \(sauf les fichiers uploadés tels que les images qui seront dans aposAttachments\). Pour l'instant, il n'y a pas grand chose :![](/assets/db_start.png)

Les groupes et utilisateurs par défaut \(admin, user\), les pages de base \(home, trash\). Une fois importées, c'est ici que seront les données des restaurants aussi. Etant donné que tout est mélangé dans la même collection, c'est le type de chaque document qui le distinguera \(c'est-à-dire la propriété "type"\).

Par exemple, ici, c'est un document de type "apostrophe-user":

![](/assets/type_user.png)

Retournons dans le fichier lib/data/index.js et ajoutons-y ce code:



