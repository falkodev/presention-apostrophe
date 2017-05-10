# Utilisation

Rien de mieux qu'une petite démo pour montrer la prise en main d'Apostrophe. Pour cela, il faut installer quelques outils tel qu'expliqué dans le tutoriel sur le site officiel : [http://apostrophecms.org/docs/tutorials/getting-started/setting-up-your-environment.html](http://apostrophecms.org/docs/tutorials/getting-started/setting-up-your-environment.html)

A noter que des collègues sur Windows ont eu des difficultés avec Node. Dans la mesure du possible, un MacOs ou un Linux est préférable. D'une manière générale, suivre le tutoriel officiel d'Apostrophe constitue une bonne introduction aux possibilités du framework. Toutefois, plusieurs personnes autour de moi \(ainsi que moi-même\) ont trouvé ce tutoriel un peu confus. C'est pour clarifier cela que j'ai décidé de créer cet article.

## Création d'un projet

Nous allons créer un projet test tel que décrit ici : [http://apostrophecms.org/docs/tutorials/getting-started/creating-your-first-project.html](http://apostrophecms.org/docs/tutorials/getting-started/creating-your-first-project.html)

Pour démarrer l'application, il faut lancer une ligne de commande : `node app.js` ou `npm start`\(puisque "npm start" lance en fait "node app.js"\). Mais à chaque modification d'un fichier, il faudra couper l'appli et relancer la ligne de commande. Pour pallier ce problème, j'aime bien installer nodemon, qui s'occupera de surveiller les modifications sur des types de fichier précis, et redémarrera l'appli comme un grand.

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

## Données

Pour démarrer, il nous faut définir une structure de donnée. Je me suis inspiré de la base de données test de MongoDB "restaurants". Nous allons donc gérer des restaurants, qui auront des notes attribuées en fonction d'avis clients.

### Structure des données

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

Ici, nous définissons une "piece" apostrophe selon un "schema", c'est-à-dire un document MongoDB qui sera de type "restaurant" \(donné par la propriété "name", ligne 3\). A l'intérieur, des champs de type chaine de caractères et obligatoirement remplis :

* une adresse,
* le quartier de New-York,
* un type de cuisine,
* un nom. 

Et aussi un type tableau, contenant 0, 1 ou plusieurs notes. Chaque note contient une date, un niveau et un score.

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

On a simplement ajouté la dernière ligne : `'restaurant': {}`Ce faisant, Apostrophe sait qu'il doit gérer un nouveau type de pièce. En se logguant \(avec le compte admin créé lors du tutoriel officiel Apostrophe\), on peut voir une nouvelle entrée dans la barre d'administration : Restaurants \(oui, Apostrophe a ajouté tout seul le "s" final à "Restaurants" en se basant sur le label déclaré dans le module que nous venons de créer. Il est d'ailleurs possible de passer un autre pluriel si nécessaire avec l'option "pluralLabel" dans ce même module\) :

![](/assets/admin_bar.png)

Si on clique sur l'entrée "Restaurants", pour l'instant aucun document n'est présent à l'intérieur. Nous allons maintenant ajouter de la donnée.

Créons un dossier "data" dans lib, et ajoutons-y un fichier data.json avec ces données qui serviront de fixtures :

```json
{"address": "1007 Morris Park Ave", "borough": "Bronx", "cuisine": "Bakery", "grades": [{"date": "01/01/2017", "grade": "A", "score": 2}, {"date": "01/01/2017", "grade": "A", "score": 6}, {"date": "01/01/2017", "grade": "A", "score": 10}, {"date": "01/01/2017", "grade": "A", "score": 9}, {"date": "01/01/2017", "grade": "B", "score": 14}], "name": "Morris Park Bake Shop"}
{"address": "469 Flatbush Avenue", "borough": "Brooklyn", "cuisine": "Hamburgers", "grades": [{"date": "01/01/2017", "grade": "A", "score": 8}, {"date": "01/01/2017", "grade": "B", "score": 23}, {"date": "01/01/2017", "grade": "A", "score": 12}, {"date": "01/01/2017", "grade": "A", "score": 12}], "name": "Wendy'S"}
{"address": "351 West Street", "borough": "Manhattan", "cuisine": "Irish", "grades": [{"date": "01/01/2017", "grade": "A", "score": 2}, {"date": 1374451200000, "grade": "A", "score": 11}, {"date": "01/01/2017", "grade": "A", "score": 12}, {"date": "01/01/2017", "grade": "A", "score": 12}], "name": "Dj Reynolds Pub And Restaurant"}
{"address": "2780 Stillwell Avenue", "borough": "Brooklyn", "cuisine": "American", "grades": [{"date": "01/01/2017", "grade": "A", "score": 5}, {"date": "01/01/2017", "grade": "A", "score": 7}, {"date": "01/01/2017", "grade": "A", "score": 12}, {"date": "01/01/2017", "grade": "A", "score": 12}], "name": "Riviera Caterer"}
```

Toujours dans ce dossier "data", nous allons créer un fichier "index.js" dont la fonction sera de parser les données pour l'insérer en base.

Pour visualiser l'état de notre base, personnellement j'utilise 3T Studio, mais n'importe quel client Mongo fera l'affaire, même le shell de base.

Avec Apostrophe, les données sont regroupées dans la même collection : aposDocs \(sauf les fichiers uploadés tels que les images qui seront dans aposAttachments\). Pour l'instant, il n'y a pas grand chose :![](/assets/db_start.png)

Les groupes et utilisateurs par défaut \(admin, user\), les pages de base \(home, trash\). Une fois importées, c'est ici que seront les données des restaurants aussi. Etant donné que tout est mélangé dans la même collection, c'est le type de chaque document qui le distinguera \(c'est-à-dire la propriété "type"\).

Par exemple, ici, c'est un document de type "apostrophe-user":

![](/assets/type_user.png)

### Insertions des fixtures

Retournons dans le fichier lib/data/index.js et ajoutons-y ce code :

```js
const fs = require('fs')
const path = require('path')
const readline = require('readline')

module.exports = {
  construct: (self, options) => {

    self.afterInit = () => {
      self.apos.docs.find({ res: {} }, { type: 'restaurant' }, { _id: 1 }) // look for existing restaurants
      .published(null) // without checking if published
      .permission(false) // without checking user permissions
      .toArray(async (err, pieces) => { // Apostrophe cursor to grab pieces
        if (err) throw err

        if (pieces.length === 0) { // if no restaraunt in DB
          try {
            await read() // asynchronously read json file
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

          // read line by line
          rl.on('line', line => {
            let data = JSON.parse(line)

            // add needed fields for Apostrophe
            data.type = 'restaurant'
            data.title = data.name
            data.slug = self.apos.utils.slugify(data.name)
            data.published = true

            insert(data) 
          })

          rl.on('close', () => resolve())
        })
      }

      const insert = data => {
        if (data) {
          // insert data with an empty req and without checking permissions
          self.apos.docs.insert({}, data, { permissions: false}, (err, data) => {
            if (err) throw err
          })
        }
      }
    }
  }
}
```

Que fait-on ici ? On utilise les événements Apostrophe : `construct` et `self.afterInit`sont des méthodes du cycle de vie du CMF. On commence par rechercher si des documents de type "restaurant" existent \(selon la méthode des curseurs préconisée dans le tutoriel Apostrophe\), sinon on lit le fichier de données et on insère les données \(toujours la méthode des curseurs\).

Dernière étape : ajouter le module "data" dans app.js pour l'activer et sauvegarder \(ce qui doit redémarrer l'appli avec nodemon\) :

```js
// Add your modules and their respective configuration here!
'restaurant': {},
'data': {}
```

En cliquant sur "Restaurants" dans la barre d'administration, on voit maintenant apparaitre les données parce que leur structure correspond au schéma défini dans le module "restaurant".

![](/assets/restaurants_modal.png)

## Templates

Les utilisateurs connectés \(auteurs\) auront la possibilité d'éditer ces "pieces", d'en ajouter, d'en supprimer. Nous allons maintenant afficher la liste des restaurants pour qu'elle soit accessible à l'ensemble des utilisateurs du site. Dans cette optique, il faut un module qui étendra "apostrophe-pieces-pages", qui comme son nom l'indique est fait pour afficher des pieces sur des pages. 2 sortes de pages sont disponibles par défaut :

* index : pour lister les produits
* show : pour afficher le détail d'un produit

Créons le dossier "restaurant-pages", Apostrophe sait qu'il servira à afficher des pièces de type "restaurant". On l'active dans app.js. Il faut aussi activer un nouveau type de page pour créer une page de restaurants. Voici la version de "app.js" correspondante :

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
    'apostrophe-pages': {
      types: [
        // Default page types
        {
          name: 'default',
          label: 'Default'
        },
        {
          name: 'home',
          label: 'Home'
        },
        // Custom page type
        {
          name: 'restaurant-page', // caution: singular type
          label: 'Restaurant'
        }
      ]
    },
    // Add your modules and their respective configuration here!
    'restaurant': {},
    'restaurant-pages': {},
    'data': {}
  }
});
```

Enfin, on crée le fichier "index.js" dans "restaurant-pages" avec ce contenu :

```js
module.exports = {
  extend: 'apostrophe-pieces-pages'
}
```

On peut désormais cliquer sur le menu "New Page" dans "Page Menu" pour voir apparaitre la fenêtre modale de création de page :

![](/assets/page_menu.png)

Le nouveau type de page "Restaurant" est bien là :

![](/assets/new_page_type.png)

Avant qu'un auteur puisse créer une page de ce type, on peut surcharger les templates : par défaut, Apostrophe affichera les templates index.html \(pour le listing des produits\) et show.html \(pour le détail d'un produit\) de son module interne. Mais on veut personnaliser un minimum l'affichage, nous créons donc un dossier "views" dans "restaurant-pages" puis à l'intérieur 2 fichiers : index.html et show.html.

index.html :

```html
{% extends data.outerLayout %}

{% block title %}{{ data.page.title }}{% endblock %}

{% block main %}
  <div style="margin-top: 90px; margin-left: 20px">
    <h2><a href="{{ data.home._url }}">{{ data.home.title }}</a></h2><br />

    <h2>Liste des restaurants</h2><br/>
    {% for piece in data.pieces %}
      <h4><a href="{{ piece._url }}">{{ piece.title }}</a></h4>
    {% endfor %}
  </div>
  {% import 'apostrophe-pager:macros.html' as pager with context %}
  {{ pager.render({ page: data.currentPage, total: data.totalPages }, data.url) }}
{% endblock %}
```

On peut noter la gestion de la pagination par Apostrophe avec la macro "pager" à la fin du template index.

show.html :

```html
{% extends data.outerLayout %}

{% block title %}{{ data.piece.title }}{% endblock %}

{% block main %}
  <div style="margin-top: 90px; margin-left: 20px">
    <h2><a href="{{ data.page._url }}">Retour à la liste des restaurants</a></h2><br />
    <h2>Détail du restaurant {{ data.piece.name }}</h2><br/>
    <ul>
      <li>Adresse: {{ data.piece.address }}</li>
      <li>Quartier: {{ data.piece.borough }}</li>
      <li>Type de cuisine: {{ data.piece.cuisine }}</li>
      <li>Evaluations:
        <ul>
          {% for grade in data.piece.grades %}
            <li>&nbsp;&nbsp;-&nbsp;Evaluation n°{{ loop.index }}</li>
            <li>&nbsp;&nbsp;&nbsp;&nbsp;Date: {{ grade.date }}</li>
            <li>&nbsp;&nbsp;&nbsp;&nbsp;Niveau: {{ grade.grade }}</li>
            <li>&nbsp;&nbsp;&nbsp;&nbsp;Note: {{ grade.score }}</li>
            <br/>
          {% endfor %}
        </ul>
      </li>
    </ul>
  </div>
{% endblock %}
```

Les templates sont au format [Nunjucks](https://mozilla.github.io/nunjucks/) dans ce CMS.

Un auteur pourra désormais créer une page /restaurants et les pages de liste et de détail sont déjà prêtes. Créons cette page de la manière suivante :

![](/assets/new_page.png)

### Navigation

Ajoutons un lien sur la page d'accueil pour naviguer vers la page des restaurants en éditant le fichier lib/modules/apostrophe-pages/views/pages/home.html :

```html
{% extends data.outerLayout %}

{% block title %}Home{% endblock %}
{% block main %}
  <div class="main-content">
    <h3>Répertoire de restaurants
      {% if not data.user %}
        <a class="login-link" href="/login">Login</a>
      {% endif %}
    </h3>
    <p>
      <a href="/restaurants">Liste des restaurants</as
    </p>
  </div>
{% endblock %}
```

Après rechargement de localhost:3000, la page d'acceuil ressemblera à ceci : ![](/assets/new_home.png)

En cliquant sur le lien, on arrivera sur l'url localhost:3000/restaurants :

![](/assets/listing.png)

Et en naviguant vers un des restaurants :

![](/assets/detail.png)



### Widgets

Si on veut laisser à l'auteur la possiblité d'aller plus loin dans la personnalisation, on peut créer des areas. Par exemple, on pourrait laisser la possibilité d'ajouter des paragraphes de texte, des images, des vidéos en ajoutant ceci dans "show.html" après le dernier `</ul>`:

```html
{{ apos.area(data.piece, 'additionalField', {
  widgets: {
    'apostrophe-rich-text': {
      toolbar: [ 'Bold', 'Italic', 'Link', 'Unlink' ]
    },
    'apostrophe-images': {},
    'apostrophe-video': {}
  }
}) }}
```

On verra apparaitre \(en étant loggué\) un petit "+" vert sur la page, nous permettant d'ajouter des widgets :

![](/assets/detail_widget.png)

En cliquant sur ce "+", on voit la liste des widgets disponibles, qui correspond aux widgets qu'on vient d'ajouter dans show.html :

![](/assets/widgets.png)

L'auteur pourra ajouter autant d'élements de ces 3 widgets qu'il souhaite et réordonner ces éléments entre eux.

![](/assets/additional-widgets.png)

### Ajout de styles

Un exemple de stylisation sera de modifier la taille de l'image. On va donc créer dans "restaurant-pages" un dossier "public", puis à l'intérieur un fichier "image.less" avec ce contenu :

```css
.apos-slideshow {
  .apos-slideshow-item {
    img {
      width: initial;
    }
  }
}
```

Ensuite, il faut dire à Apostrophe d'importer ce nouveau fichier de style en modifiant le fichier index.js de "restaurant-pages" comme ceci :

```js
module.exports = {
  extend: 'apostrophe-pieces-pages',
  construct: function (self, options) {
    self.pushAsset('stylesheet', 'image', { scene: 'always' })
  }
}
```

Voilà pas mal d'éléments pour prendre en main Apostrophe rapidement. En cas de question, n'hésitez pas à créer une issue sur le [repository github de cet article.](https://github.com/falkodev/presention-apostrophe)

