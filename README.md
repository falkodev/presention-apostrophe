# Présentation du CMS Apostrophe

En fait de CMS, je dirais qu'il s'agit d'un CMF, d'un framework qui va pouvoir créer un CMS sur-mesure. Celui-ci est basé sur Node.js et MongoDB. A l'origine fait en Symfony donc en PHP, les créateurs, une agence web de Philadelphie, ont switché vers Node il y a quelques années.

![](/assets/import.png)

Dans cet article, nous distinguerons deux acteurs:

* le développeur, utilisateur du CMF, créateur d'une solution pour un client
* l'auteur, utilisateur du CMS, qui doit créer des pages, ajouter du contenu, changer des styles

## 

## Fonctionnalités et concepts

L'intérêt d'[Apostrophe](http://apostrophecms.org/) est sa capacité à faire du "live editing", permettant à l'auteur de modifier sa page directement, sans prévisualisation à la Wordpress. Il doit toujours publier sa page avant qu'elle ne soit accessible au grand public, mais il visualise le rendu directement.

![](/assets/import2.png)

Le CMF utilise plusieurs concepts :

* chaque composant est appelé "module" : module images, module fichiers, module assets... Il est possible de créer ses propres modules ou d'étendre ceux présents par défaut
* les données - requêtées depuis Mongo - sont des "pieces". Elles respectent un "schema" qui définit un document Mongo, et des "cursors" peuvent leur être rattachées pour les récupérer par paquets \(comme un LIMIT en SQL\)
* si une "piece" doit être rappelé à plusieurs endroits du site, on la transforme en "widget"
* en fonction de si l'administrateur est connecté ou non, le contexte change \(apparition d'une barre d'administration des "pieces"\) et des ressources sont poussées \(fichiers css et js\) en fonction de ce qu'a configuré le développeur \(autorisations par rôles\)

Toutes ces notions seront explicitées plus loin.



## Outils

Le CMF datant de quelques années, il utilise par défaut LESS en tant que compilateur CSS, et jQuery. On verra qu'il est possible d'utiliser Sass et d'inclure des applications modernes basées sur React ou Vue. Mais, Tom Boutell, le principal créateur de la solution, est fan de jQuery. Il s'en sert essentiellement pour de l'AJAX, et défend son point de vue dans le tutoriel : [http://apostrophecms.org/docs/tutorials/getting-started/reusable-content-with-pieces.html](http://apostrophecms.org/docs/tutorials/getting-started/reusable-content-with-pieces.html) \(vers la fin de la page, il explique ne pas avoir besoin de React ou d'Angular\). Il est vrai que son système de "live editing" est basé sur jQuery et fonctionne très bien, sans rechargement, à la manière de ce qui se fait dans beaucoup d'applications modernes.



## Sites utilisant Apostrophe

A part apparemment tous les sites web des administrations de Philadelphie, en France, le groupe Michelin et ses filiales commencent à utiliser Apostrophe. Par exemple : [velo.michelin.fr](http://velo.michelin.fr)





