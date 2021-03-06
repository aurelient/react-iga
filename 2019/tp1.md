## TP1 Mise en place de la Stack React


### Présentation du TP

L'objectif du TP est de mettre en place une Single Page Application (SPA), développée principalement côté client avec React, avec un serveur Node/Express léger. Client et serveur seront codés en JavaScript. Nous allons voir:

- La mise en place d'un serveur Express très basique
- L'automatisation et le bundling avec Webpack
- Comment configurer Babel pour la rétrocompatibilité du code
- Créer un projet React
- Générer un bundle
- Configurer ESlint et vérifier son projet
- Assembler et servir le contenu


Ce TP fera l'objet d'un premier rendu __individuel__ et d'une note. Voir les critères d'évaluation en bas de la page.

Vous ferez le rendu sur github. Créez un projet git dès maintenant, puis un projet (npm init).

Pensez à remplir le <a href="https://airtable.com/shrjtxHBCc9ZFQJu1">formulaire de rendu</a>.

### Mise en place du serveur

Structurer votre projet pour avoir un dossier serveur et un dossier client clairement nommés.

Installer [Node](https://nodejs.org/) et [Express](https://expressjs.com/) si ce n'est pas déjà fait. Si c'est le cas, pensez à les mettre à jour.


__Pensez à bien ajouter les fichiers qui n'ont pas à être versionnés à .gitignore__ (ex: node_modules, dist, ...)

Voici le coeur d'un serveur Express.

```javascript
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;
const mockResponse = {
  foo: 'bar',
  bar: 'foo'
};
app.get('/api', (req, res) => {
  res.send(mockResponse);
});
app.get('/', (req, res) => {
 res.status(200).send('Hello World!');
});
app.listen(port, function () {
 console.log('App listening on port: ' + port);
});
```

Ajouter un script à package.json qui permette de lancer votre serveur.avec la commande
`npm run start`

```json
"scripts": {
  "start": "node serveur/index.js",
  "test": "echo \"Error: pas de tests pour le moment\" && exit 1"
}
```


Vérifier que le serveur fonctionne et versionnez.


### Mise en place de Webpack

Installer [Webpack](https://webpack.js.org/) en dev (pas la peine d'avoir les dépendances pour le déploiement)  :

- `webpack` (Le bundler)
- `webpack-cli` (Command Line Interface pour lancer les commandes webpack)

Dans `package.json` ajouter une commande `build` (dans `scripts`)

```json
"scripts": {
  "build": "webpack --mode production"
}
```

Tentez un build, regarder les fichiers/dossiers générés. Corriger votre arborescence de fichier si nécessaire.

### Configuration de Babel

React s'appuie sur [JSX](https://reactjs.org/docs/introducing-jsx.html) pour
lier la logique de rendu, la gestion d'évènement et les changements d'états
pour un élément donné. Ces éléments seraient normalement séparés entre langages
et technos différentes. Babel permet de traduire ce code (et au passage de transformer du ES6 en ES6).

Installer les dépendances (de développement) suivantes:

- `@babel/core` (ES6+ vers ES5)
- `@babel/preset-env` (Preset pour les polyfills)
- `@babel/preset-react` (Preset pour React et JSX)
- `babel-loader` (pour l'intégration avec Webpack)

Configurer Babel à l'aide d'un fichier `.babelrc` à la racine de votre projet,
en indiquant les pré-configurations utilisées pour le reste du projet. 
Il doit contenir les lignes suivantes:

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```


### Projet React

Nous verrons plus en détail le fonctionnement de React lors de la prochaine séance.
Pour le moment nous allons créer un projet simple.

Installons react et react-dom

```
npm i react react-dom
```

Dans votre dossier client (`src`), créer un `index.html`.
Ce sera le seul fichier HTML du projet, il sera "peuplé" dynamiquement par React.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>React Boilerplate</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

Dans le même dossier nous allons créer un premier élément React:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
const Index = () => {
  return <div>TP1!</div>;
};
ReactDOM.render(<Index />, document.getElementById('root'));
```


### Générer un bundle avec webpack

Il faut maintenant transpiler ce code pour qu'il soit compréhensible par un navigateur. 
Le résultat ira dans le dossier `dist``



On installe le module [html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin) pour faciliter la création de fichier HTML avec Webpack.

On point vers le point d'entée React (le fichier index.js) et ou l'appliquer (`template: "./src/index.html"`).


Créer un fichier `webpack.config.js` avec le contenu suivant:

```javascript
const HtmlWebPackPlugin = require("html-webpack-plugin");
const path = require('path');
const htmlPlugin = new HtmlWebPackPlugin({
  template: "./src/index.html",
  filename: "./index.html"
});
module.exports = {
  entry: "./src/index.js",
  output: { // NEW
    path: path.join(__dirname, 'dist'),
    filename: "[name].js"
  }, // NEW Ends
  plugins: [htmlPlugin],
  module: {
    rules: [
      {
        ...
      }
    ]
  }
};
```

Il faut spécifier à Webpack la transpilation Babel des fichiers .js et .jsx du projet lors du build. 
Ajouter la règle suivante dans le fichier de configuration webpack: 

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader"
        }
      }
    ]
  }
};
```

### Assembler et servir le contenu

Il faut maintenant dire à Express ou aller chercher le contenu.
Pour cela il faut lui dire que sa route '/' est maintenant `dist/index.html`

Rajouter les constantes suivantes (selon vos noms de fichiers et de dossier) dans l'index.js de votre serveur:

```js
const path = require('path');

const DIST_DIR = path.join(__dirname, '../dist');
const HTML_FILE = path.join(DIST_DIR, 'index.html');

// La route '/' pointe sur HTML_FILE
```

Nous allons aussi rajouter la commande suivante `package.json` pour distinguer un build de dev et un de production.

```
  "dev": "webpack --mode development && node server/index.js",
```


### Gérer les fichier statiques

Pour que Express trouve plus tard son chemin "de base" et les fichiers statiques générés par React (images, css...) rajouter la ligne suivante:

```js
app.use(express.static(DIST_DIR));
```

Il faut aussi installer le module `file-loader` (toujours en dev).

Et rajouter la règle suivante dans `webpack.config.js:` pour que webpack place les images dans `/static/`.
```js
{
  test: /\.(png|svg|jpg|gif)$/,
  loader: "file-loader",
  options: { name: '/static/[name].[ext]' }
}
```

Pour que Webpack bundle ces images (ou autres ressources), il faudra les importer dans vos composants.

```js
// Import de l'image
import LOGO from '<path-to-file>/logo.png';

// Utilisation
<img src={LOGO} alt="Logo" />
```

### CSS

Nous allons utiliser [react-bootstrap](https://react-bootstrap.github.io/) comme framework CSS. Installez le (`npm install ...`)

<!-- 
### Des composants Reacts

Créer deux composants basiques sans aucune logique. Le premier affichera un titre. Le deuxième affichera des images prises dans un dossier statique.
On placera ces composants dans un dossier `components`.

```js
import React from 'react';
import ReactDOM from 'react-dom';
import Header from './components/Header/index.jsx';
import Content from './components/Content/index.jsx';
const Index = () => {
  return (
    <div className="container">
      <Header />
      <Content />
    </div>
  );
};
ReactDOM.render(<Index />, document.getElementById('root'));
``` 
-->


### ESLint

Le *linting* consiste à vérifier que votre code respecte de bonnes pratiques. 

Installer `eslinst` : `npm install eslint --save-dev``

Puis configurer eslint via `eslint --init``

Voici le type de réponses à fournir pour un projet React :

- How would you like to use ESLint? **To check syntax, find problems, and enforce code style**
- What type of modules does your project use? **JavaScript modules (import/export)**
- Which framework does your project use? **React**
- Does your project use TypeScript? **No**
- Where does your code run? **Browser**
- How would you like to define a style for your project? **Use a popular style guide**
- Which style guide do you want to follow? **Airbnb: https://github.com/airbnb/javascript**
- What format do you want your config file to be in? **JSON**

Lancer `eslint "src/**/*.js"` pour vérifier le code existant

`eslint "src/**/*.js" --fix` permet de corriger certaines problèmes automatiquement


### Déployer sur Heroku
Afin de rendre notre application disponible sur les internets, nous allons la déployer sur [Heroku](https://heroku.com).

Suivre le guide de Heroku pour déployer une application via git :
[https://devcenter.heroku.com/articles/git#creating-a-heroku-remote](https://devcenter.heroku.com/articles/git#creating-a-heroku-remote)

N'oubliez pas de désactiver l'option `watch` de webpack si vous lancez Webpack en `--mode production` [voir ici](https://webpack.js.org/configuration/mode/).


### React Developer Tools

Installez l'extension [React Developer Tools](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/) dans votre navigateur préféré.

Inspectez l'application.


### Fin


- Le rendu est à faire sur le <a href="https://airtable.com/shrjtxHBCc9ZFQJu1">formulaire</a> suivant.
- Le projet ne contient que des éléments nécessaire (.gitignore est bien défini)
- Les dépendances sont bien définies dans package.json 
- `npm run build` construit le projet
- `npm run start` lance le serveur et permet de tester le projet.
