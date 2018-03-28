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

Avant qu'un auteur puisse créer une page de ce type, on peut surcharger les templates : par défaut, la librairie "apostrophe-pieces-pages" interne à Apostrophe \(dans node\_modules/apostrophe/lib/apostrophe-pieces-pages\) contient les templates index.html \(pour le listing des produits\) et show.html \(pour le détail d'un produit\) de son module interne. Mais on veut personnaliser un minimum l'affichage, nous créons donc un dossier "views" dans "lib/modules/restaurant-pages" puis à l'intérieur 2 fichiers : index.html et show.html, ce qui aura pour effet de remplacer les templates d'origine.

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

On peut noter la gestion de la pagination avec la macro "pager" à la fin du template index.

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

Un auteur pourra désormais créer une page "/restaurants" et les pages de liste et de détail sont déjà prêtes. Créons cette page de la manière suivante :

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

