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

Pour démarrer, il nous faut définir une structure de donnée. Je me suis inspiré de la base de données test de MongoDB "restaurants". Nous allons donc gérer des restaurants, qui auront des notes attribuées en fonction d'avis clients.

Pour modéliser ces données, il faut créer un module Apostrophe. Pour cela, créons un dossier "restaurant" dans "lib", et insérons un fichier "index.js" avec ce contenu : 

```js
module.exports = {
  extend: 'apostrophe-pieces',
  name: 'restaurant',
  label: 'Restaurant',
  addFields: [
    {
      name: 'address',
      label: 'Address',
      type: 'string',
      required: true
    }, {
      name: 'borough',
      label: 'Borough',
      type: 'string',
      required: true
    }, {
      name: 'cuisine',
      label: 'Cuisine',
      type: 'string',
      required: true
    }, {
      name: 'grades',
      label: 'Grades',
      type: 'array',
      schema: [
        {
          name: 'date',
          label: 'Date',
          type: 'string',
          required: true
        },
        {
          name: 'grade',
          label: 'Grade',
          type: 'string',
          required: true
        },
        {
          name: 'score',
          label: 'Score',
          type: 'integer',
          required: true
        }
      ]
    }, {
      name: 'name',
      label: 'Name',
      type: 'string',
      required: true
    }
  ]
}
```

Ici, nous définissons une "piece" apostrophe, c'est-à-dire un document MongoDB qui sera de type "restaurant" \(donné par la propriété "name", ligne 3\). A l'intérieur, des champs de type chaine de caractères et obligatoirement remplis : 

* une adresse,
* le quartier de New-York,
* un type de cuisine,
* un nom. 

Et aussi un type tableau, contenu 0, 1 ou plusieurs notes. Chaque note contient une date, un niveau et un score.

Pour rendre ce module actif, il faut éditer le fichier "app.js" à la racine du projet :

```js
var apos = require('apostrophe')({
  shortName: 'test-project',
  title: 'test-project',

  // These are the modules we want to bring into the project.
  modules: {
    // This configures the apostrophe-users module to add an admin-level
    // group by default
    'apostrophe-users': {
      groups: [
        {
          title: 'guest',
          permissions: [ ]
        },
        {
          title: 'admin',
          permissions: [ 'admin' ]
        }
      ]
    },
    // This configures the apostrophe-assets module to push a 'site.less'
    // stylesheet by default
    'apostrophe-assets': {
      stylesheets: [
        {
          name: 'site'
        }
      ]
    },
    // Add your modules and their respective configuration here!
    'restaurant': {}
  }
});
```

On a simplement ajouté la dernière ligne : `'restaurant': {}`Ce faisant, Apostrophe sait qu'il doit gérer un nouveau type de pièce. En se logguant \(avec le compte admin créé lors du tutoriel officiel Apostrophe\), on peut voir une nouvelle entrée dans la barre d'administration : Restaurants \(oui, Apostrophe a ajouté tout seul le "s" final à "Restaurants" en se basant sur le label déclaré dans le module que nous venons de créer. Il est d'ailleurs possible de passer un autre pluriel si nécessaire avec l'option pluralLabel dans ce même module\) :

![](/assets/admin_bar.png)

Si on clique sur l'entrée "Resturants", pour l'instant aucun document à l'intérieur. Nous allons maintenant ajouter de la donnée.

Créons un dossier "data" dans lib, et ajoutons-y un fichier data.json avec ces données : 

```json
{"address": "1007 Morris Park Ave", "borough": "Bronx", "cuisine": "Bakery", "grades": [{"date": "01/01/2017", "grade": "A", "score": 2}, {"date": "01/01/2017", "grade": "A", "score": 6}, {"date": "01/01/2017", "grade": "A", "score": 10}, {"date": "01/01/2017", "grade": "A", "score": 9}, {"date": "01/01/2017", "grade": "B", "score": 14}], "name": "Morris Park Bake Shop"}
{"address": "469 Flatbush Avenue", "borough": "Brooklyn", "cuisine": "Hamburgers", "grades": [{"date": "01/01/2017", "grade": "A", "score": 8}, {"date": "01/01/2017", "grade": "B", "score": 23}, {"date": "01/01/2017", "grade": "A", "score": 12}, {"date": "01/01/2017", "grade": "A", "score": 12}], "name": "Wendy'S"}
{"address": "351 West Street", "borough": "Manhattan", "cuisine": "Irish", "grades": [{"date": "01/01/2017", "grade": "A", "score": 2}, {"date": 1374451200000, "grade": "A", "score": 11}, {"date": "01/01/2017", "grade": "A", "score": 12}, {"date": "01/01/2017", "grade": "A", "score": 12}], "name": "Dj Reynolds Pub And Restaurant"}
{"address": "2780 Stillwell Avenue", "borough": "Brooklyn", "cuisine": "American", "grades": [{"date": "01/01/2017", "grade": "A", "score": 5}, {"date": "01/01/2017", "grade": "A", "score": 7}, {"date": "01/01/2017", "grade": "A", "score": 12}, {"date": "01/01/2017", "grade": "A", "score": 12}], "name": "Riviera Caterer"}
```

Toujours dans ce dossier "data", nous allons créer un fichier index.js dont la fonction sera de parser les données pour l'insérer en base.

Pour visualiser l'état de notre base, personnellement j'utilise 3T Studio, mais n'importe quel client Mongo fera l'affaire, même le shell de base.

Avec Apostrophe, les données sont regroupées dans la même collection : aposDocs \(sauf les fichiers uploadés tels que les images qui seront dans aposAttachments\). Pour l'instant, il n'y a pas grand chose :![](/assets/db_start.png)

Les groupes et utilisateurs par défaut \(admin, user\), les pages de base \(home, trash\). Une fois importées, c'est ici que seront les données des restaurants aussi. Etant donné que tout est mélangé dans la même collection, c'est le type de chaque document qui le distinguera \(c'est-à-dire la propriété "type"\).

Par exemple, ici, c'est un document de type "apostrophe-user":

![](/assets/type_user.png)

Retournons dans le fichier lib/data/index.js et ajoutons-y ce code :

```js
const fs = require('fs')
const path = require('path')
const readline = require('readline')

module.exports = {
  construct: (self, options) => {

    self.afterInit = () => {
      self.apos.docs.find({ res: {} }, { type: 'restaurant' }, { _id: 1 })
      .published(null)
      .permission(false)
      .toArray(async (err, pieces) => {
        if (err) throw err

        if (pieces.length === 0) {
          try {
            const data = await read()
            await insert(data)
          } catch (err) {
            throw err
          }
        }
      })

      const read = () => {
        new Promise((resolve, reject) => {
          const rl = readline.createInterface({
            input: fs.createReadStream(path.resolve(__dirname, './data.json'))
          });

          rl.on('line', async line => {
            let data = JSON.parse(line)
            data.type = 'restaurant'
            data.title = data.name
            data.slug = self.apos.utils.slugify(data.name)
            data.published = true

            await insert(data)
          })

          rl.on('close', () => resolve())
        })
      }

      const insert = data => {
        if (data) {
          new Promise((resolve, reject) => {
            self.apos.docs.insert({}, data, { permissions: false}, (err, data) => {
              if (err) reject(err)
              resolve()
            })
          })
        }
      }
    }
  }
};
```





