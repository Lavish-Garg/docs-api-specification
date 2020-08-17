---
title: Spécification API
---

Les API de Gatsby sont conçues conceptuellement dans une certaine mesure après React.js pour améliorer la cohérence entre les deux systèmes.

Les deux principales priorités de l'API sont a) permettre un écosystème de plugins large et robuste et b) en plus de cela, un écosystème de thèmes large et robuste.

## Conditions préalables

Si vous ne connaissez pas le cycle de vie de Gatsby, consultez la présentation [Gatsby Lifecycle APIs](/docs/gatsby-lifecycle-apis/).

## Plugins

Les plugins peuvent étendre Gatsby de plusieurs manières:

- Recherche de données (par exemple à partir du système de fichiers ou d'une API ou d'une base de données)
- Transformer des données d'un type à un autre (par exemple, un fichier de démarque en HTML)
- Création de pages (par exemple, un répertoire de fichiers markdown est tous transformé en pages avec des URL dérivées de leurs noms de fichiers).
- Modification de la configuration du webpack (par exemple pour les options de style, ajout de la prise en charge d'autres langages de compilation vers js)
- Ajouter des éléments au HTML rendu (par exemple, des balises meta, des extraits de code JS analytiques comme Google Analytics)
- Rédaction d'éléments pour créer un répertoire en fonction des données du site (par exemple, technicien de service, plan du site, flux RSS)

Un seul plugin peut utiliser plusieurs API pour atteindre son objectif. Par exemple. le plugin pour la bibliothèque CSS-in-JS [Glamor](/packages/gatsby-plugin-glamor/):

1.  modifie la configuration du webpack pour ajouter son plugin
2.  ajoute un plugin Babel pour remplacer le createElement par défaut de React
3.  modifie le rendu du serveur pour extraire le CSS critique pour chaque page rendue et intégrer le CSS dans le `<head>` de cette page HTML.

Les plugins peuvent également dépendre d'autres plugins. [The Sharp plugin](/packages/gatsby-plugin-sharp/) expose un certain nombre d'API de haut niveau pour transformer des images que plusieurs autres les plugins d'image Gatsby dépendent de. [gatsby-transformer-remark](/packages/gatsby-transformer-remark/) effectue une transformation basique markdown-> html mais expose une API à permettre à d'autres plugins d'intervenir dans le processus de conversion, par ex. [gatsby-remark-prismjs](/packages/gatsby-remark-prismjs/) ce qui ajoute une mise en évidence aux blocs de code.

Les plugins Transformer sont découplés des plugins source. Les plugins Transformer examinent le type de média des nouveaux nœuds créés par les plugins source pour décider s'ils peuvent le transformer ou non. Ce qui signifie qu'un plugin de transformateur de démarque peut transformer le démarquage de n'importe quelle source sans aucune autre configuration, par exemple. à partir d'un fichier, d'un commentaire de code ou d'un service externe comme Trello qui prend en charge le démarquage dans certains de ses champs de données.

Voir [the full list of (official only for now — adding support for community plugins later) plugins](/docs/plugins/).

## API

### Concepts

- _Page_ - une page de site avec un chemin, un composant de modèle et une requête GraphQL facultative.
- _Page Component_ - Composant React.js qui rend une page et peut éventuellement spécifier une requête GraphQL
- _Component extensions_ —  extensions qui peuvent être résolues en tant que composants. `.js` et `.jsx` sont pris en charge par core. Mais les plugins peuvent ajouter la prise en charge d'autres langages de compilation en js.
- _Dependency_ — Gatsby suit automatiquement les dépendances entre différents objets, par ex. une page peut dépendre de certains nœuds. Cela permet le rechargement à chaud, la mise en cache, les reconstructions incrémentielles, etc.
- _Node_ — un objet de données
- _Node Field_ — un champ ajouté par un plugin à un nœud qu'il ne contrôle pas
- _Node Link_ — une connexion entre les nœuds qui est convertie en relations GraphQL. Peut être créé de différentes manières et automatiquement déduit. Les liens parent / enfant des nœuds et leurs nœuds dérivés transformés sont des liens de première classe.

_D'autres définitions et termes sont définis dans le [Glossary](/docs/glossary/)_

### Les opérateurs

- _Create_ — faire une nouvelle chose
- _Get_ — obtenir une chose existante
- _Delete_ — supprimer un élément existant
- _Replace_ — remplacer une chose existante
- _Set_ — fusionner dans une chose existante

### API d'extension

Gatsby a plusieurs processus. Le plus important est le processus de "bootstrap". Il comporte plusieurs sous-processus. Une partie délicate de leur conception est qu'ils s'exécutent une fois pendant le bootstrap initial, mais restent également en vie pendant le développement pour continuer à répondre aux changements. C'est ce qui pousse au rechargement à chaud que toutes les données Gatsby sont «vivantes» et réagissent aux changements de l'environnement.

Le processus d'amorçage est le suivant:

load site config -> load plugins -> source nodes -> transform nodes -> create graphql schema -> create pages -> compile component queries -> run queries -> fin

Once the initial bootstrap is finished, a `webpack-dev-server` and express server are started for serving files for the development workflow with live updates. For a production build, Gatsby skips the development server and instead builds the CSS, then JavaScript, then static HTML with webpack.

During these processes there are various extension points where plugins can intervene. All major processes have an `onPre` and `onPost` e.g. `onPreBootstrap` and `onPostBootstrap` or `onPreBuild` or `onPostBuild`. During bootstrap plugins can respond at various stages to APIs like `onCreatePages`, `onCreateBabelConfig`, and `onSourceNodes`.

At each extension point, Gatsby identifies the plugins which implement the API and calls them in serial following their order in the site's `gatsby-config.js`.

In addition to extension APIs in a node, plugins can also implement extension APIs in the server rendering process and the browser e.g. `onClientEntry` or `onRouteUpdate`.

The three main inspirations for this API and spec are React.js' API specifically [@leebyron's email on the React API](https://gist.github.com/vjeux/f2b015d230cc1ab18ed1df30550495ed), this talk ["How to Design a Good API and Why it Matters" by Joshua Bloch](https://www.youtube.com/watch?v=heh4OeB9A-c&app=desktop) who designed many parts of Java, and [Hapi.js](https://hapijs.com/api)' plugin design.
