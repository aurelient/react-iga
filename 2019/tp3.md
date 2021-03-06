## TP3 Redux

Nous allons maintenant gérer l'état de l'application en utilisant Redux.
L'état comprend la liste des transparents et le transparent en cours.

**Pensez à relire le cours et les ressources associées pour être au clair sur ce que vous êtes en train de faire.**

Afin de vous faciliter le debug du TP, vous pouvez activer la création d'un Source Map dans votre `webpack.config.js` : `devtool: 'eval-source-map'`.

### Redux

[Installez Redux et les dépendances associées pour React](https://redux-docs.netlify.com/introduction/installation) (`react-redux`). Par défaut Redux n'est pas lié à React et peut être utilisé avec d'autres Framework.

Créez dans `src` des dossiers pour organiser votre `store`, vos `reducers`, `actions`, et `containers`.


#### Création d'un store

Nous allons commencer par créer le store qui va gérer les états.

```js
    // src/store/index.js
    import { createStore } from "redux";
    import rootReducer from "../reducers/index";
    const store = createStore(rootReducer);
    export default store;
```

On importe `createStore` depuis redux et aussi `rootReducer`, on verra plus bas la définition des reducers.

`createStore` peut aussi prendre un état initial en entrée, mais c'est en React c'est les reducers qui produisent l'état de l'application (y compris l'état initial).


#### Création d'un reducer

On crée un premier reducer qui va initialiser l'application. Le `rootReducer` a un état initial constant (_immutable_) à compléter.

```js
    const initialState = {
      index: 1, // initialise votre presentation au transparent 1
      slides: [] // vous pouvez réutiliser votre état de slides initial.
    };
    function rootReducer(state = initialState, action) {
    switch (action.type) {
        case ADD_SLIDE:
            return ...
        case REMOVE_SLIDE:
            return ...
        case NEXT_SLIDE:
            return ...
        case PREVIOUS_SLIDE:
            return ...
        default:
            return state
    }
    return state;
    };
    export default rootReducer;
```

#### Tester Redux et le store

Dans votre `index.js` principal exposez le store pour pouvoir l'afficher via la console du navigateur.
Cela permettra d'effectuer les premiers tests de Redux, sans l'avoir branché à votre application React.

```js
    import store from "store/index"; // verifiez que le chemin est correct
    window.store = store;
```
Un devtool pour Redux est disponible pour [Chrome](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd) et [Firefox](https://addons.mozilla.org/en-US/firefox/addon/reduxdevtools/), il necessite quelques légères [modification de votre code](http://extension.remotedev.io/#11-basic-store).


#### Creation des actions

Suivre le [guide de Redux sur la création d'action](https://redux.js.org/basics/actions) en transposant la gestion de todos à la gestion de slides.

Vous aurez à définir les actions suivantes dans `actions/index.js`

```js
export const ADD_SLIDE = "ADD_SLIDE";
export const REMOVE_SLIDE = "REMOVE_SLIDE";
export const NEXT_SLIDE = 'NEXT_SLIDE'
export const PREVIOUS_SLIDE = 'PREVIOUS_SLIDE'
```

Ces actions sont utilisées comme signaux par Redux, ce ne que des objects JavaScript Simle. C'est le Reducer qui va faire le travail.

Les actions forcent principalement à définir des traitement unitaire.

Une bonne pratique Redux consiste à envelopper les actions dans une fonction pour s'assurer que la création de l'objet est bien faite. Ces fonction s'appellent `action creator`.

```js
export function addSlide(payload) {
  return { type: ADD_SLIDE, payload };
}
```

**Ne le faites pas maintenant**. Toutefois, quand votre application deviendra plus complexe, vous pourrez utiliser la convention [Redux duck](https://github.com/erikras/ducks-modular-redux) pour organiser votre code.


#### Tester les actions

Toujours dans votre `index.js` principal, exposez les actions pour vérifier qu'elles changent bien le contenu du store.
Redux n'est toujours pas branché à React, il est donc normal que l'interface ne change pas pour le moment. Mais vous pouvez observer l'état via l'extension Redux ou un simple `console.log()` dans votre Reducer.

```js
    import { nextSlide } from "actions/index"; // verifiez que le chemin est correct
    window.nextSlide = nextSlide;
```

#### Lien Redux / React

Nous allons lier votre composant principal à Redux.

Pour commencer `Redux` fournit un composant `<Provider>` à placer à la racine de votre application.

Voir l'[exemple de la documentation](https://react-redux.js.org/introduction/quick-start#provider).

Ce `provider` va rendre le store Redux disponible dans l'application.

Il faut ensuite ce connecter à ce store. Pour cela on utilise la fonction `connect()`, et en définissant une fonction `mapStateToProps()` qui va faire le lien entre le state Redux et les props de votre composant.

```js
const mapStateToProps = (state) => {
  return {
    slides: state.slides,
  }
}

... VOTRE_COMPOSANT ...

export default connect(mapStateToProps)(VOTRE_COMPOSANT)
```


Maintenant nous allons modifier l'état de `index` en cliquant sur les boutons précédent/suivant de votre `Toolbar`. Pour cela nous allons utiliser `mapDispatchToProps()` [qui va connecter notre application React aux actions Redux](https://react-redux.js.org/using-react-redux/connect-mapdispatch).

1. Importer connect de `react-redux`, et les actions depuis votre fichier de définition d'action.
```js
import { connect } from "react-redux";
import { action1, action2 } from "..../actions/index";
```

2. Créer un `mapDispatchToProps` et le connecter avec votre composant.

```js
const mapDispatchToProps = dispatch => {
  return {
    nextSlide: () => dispatch(nextSlide(true)),
    previousSlide: () => dispatch(previousSlide(true))
  }
}
// ... VOTRE_COMPOSANT

export default withRouter(connect(null, mapDispatchToProps)(VOTRE_COMPOSANT));
```

3. Enfin en cas de clic sur vos boutons avant/apres appelez vos actions  `onClick={() => {this.props.previousSlide}`


#### Lien Redux / React Router  

Normalement l'intégration avec React Router se passe bien (pas de changements nécessaire). Si jamais ce n'était pas le cas, suivez l'utilisation de Redux avec React Router telle que présentée dans la documentation de [React Router](https://reacttraining.com/react-router/web/guides/redux-integration) ou celle de [Redux](https://redux.js.org/advanced/usage-with-react-router) pour configurer votre projet.  



### Evaluation

- Fichier README.md décrivant le process de build en dev, en prod, et de déploiement.
- Fichier package.json nettoyé ne contenant que les dépendances nécessaires.
- Déploiement sur Heroku
- Composants React pour le Slideshow, les Slides, la Toolbar.
- Store / reducer qui gère l’état de l’application
- Le flux de données suit le flow React, des actions sont déclarées, et les changements d’états passent par des actions unitaires qui modifient le store.
- Les changement sont des fonctions qui renvoient un nouvel état (immutabilité) dans le reducer.
- Redux pour la gestion avancée des états
- Gestions des routes pour les transparents
- Suivant/precedent change l’URI. Changer la route dans la barre d’URL du navigateur change l’etat de l’application.
- Qualité globale du rendu (= application qui ressemble à quelque chose, un minimum de mise en page, orthographe propre, composants s’appuyant sur des librairies CSS ou stylés à la main).
