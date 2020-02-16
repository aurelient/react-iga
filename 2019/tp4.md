## TP4 Distribution d’interface multi-dispositif

Nous allons maintenant travailler à la distribution de l'application sur plusieurs dispositifs et à leur synchronisation.
L'objectif est que vous puissiez changer l'état de votre application de présentation sur un dispositif (ex: mobile), et que l'état de l'application soit mis à jour partout (ex: vidéo-projection, personne qui regarde votre présentation à distance sur sa machine...)


### Définition de nouvelles routes et des vues associées

Nous allons définir une route par situations d'usage :

- `controler` : route pour dispositif mobile qui va controler la présentation et afficher les notes de présentation.
- `present` : route pour le mode présentation plein écran, seule une diapositive plein écran sera affichée (pas de toolbar).
- `edit`: mode actuel permettant l'édition des transparents

Il n'existe pas de bibliothèque à l'heure actuelle pour gérer de manière simple de la distribution d'interface, nous allons donc devoir le faire "à la main".

Rajouter des `Redirect` [(doc)](https://reacttraining.com/react-router/web/api/Redirect) à la racine de votre application pour faire une redirection vers une route en fonction du dispositif utilisé et de son état.

Vous pouvez utiliser `react-device-detect` [(doc)](https://www.npmjs.com/package/react-device-detect) pour détecter le dispositif (mobile ou non). Et la `fullscreen API` [(doc)](https://developer.mozilla.org/en-US/docs/Web/API/Fullscreen_API/Guide) pour controler le plein écran.

Déployez et tester.

### Créez une vue controler

Cette vue pour mobile affiche les notes de présentation associées à un transparet ainsi que les boutons suivant précédent.

Nous allons travailler sur la synchronisation entre les dispositifs ci-dessous. Pour l'instant la vue doit simplement afficher les notes correspondant au transparent courant.

### Gestion "à la main" des routes des transparents.

Nous allons maintenant préparer la synchronisation des dispositifs. Pour cela nous allons devoir gérer le transparent courant dans notre état (`currentSlide` dans le store).
`ReactRouter` n'est pas conçu pour bien gérer le lien entre route et état. Et les routeur alternatifs (type `connected-react-router`) ont aussi des limites. Nous allons donc gérer cette partie de la route à la main.

#### Changer l'état à partir de la route

En écoutant l'évènement `popstate` nous pouvons êtres informé d'un changement dans l'url du navigateur. Si ce changement correspond à un changement dans le numéro de transparent à afficher, nous allons déclencher l'action `setSlide`, avec le numéro de transparent approprié.

```javascript
// Update Redux if we navigated via browser's back/forward
// most browsers restore scroll position automatically
// as long as we make content scrolling happen on document.body
window.addEventListener('popstate', () => {
  // `setSlide` is an action creator that takes
  // the hash of the url and pushes to the store it.

  // TODO add parsing and checks on the location.hash
  // to make sure it is a proper slide index.
  slideIndex = window.location.hash
  store.dispatch(setSlide(slideIndex))
})
```

Si vous n'avez pas encore définit l'action `setSlide`, créez le action creator correspondant, et le traitement associé dans le reducer.

#### Changer la route à partir de l'état
En écoutant les changements dans le store nous allons pouvoir être notifiés de changement de l'état et les répercuter dans la barre d'url (utile pour la suite, quand nous allons synchroniser des dispositifs):

```javascript
// The other part of the two-way binding is updating the displayed
// URL in the browser if we change it inside our app state in Redux.
// We can simply subscribe to Redux and update it if it's different.
store.subscribe(() => {
  const hash = "#/" + store.getState().currentSlide
  if (location.hash !== hash) {
    window.location.hash = hash
    // Force scroll to top this is what browsers normally do when
    // navigating by clicking a link.
    // Without this, scroll stays wherever it was which can be quite odd.
    document.body.scrollTop = 0
  }
})
```

### Refactorisation
Avant de passer à la suite, nous allons simplifier les ACTIONS de Redux. Supprimez les actions `NEXT_SLIDE` et `PREVIOUS_SLIDE` de votre liste d'actions et de votre Reducer. Aux endroits où ces actions étaient utilisées, remplacer par l'action `SET_SLIDE` avec une incrémentation ou une décrémentation de l'index courant.

### Middleware et websockets

Pour comprendre la logique du Middleware [suivez la documentation Redux](https://redux.js.org/advanced/middleware). Faites un essai qui reprend l'idée et logue dans la console toutes les actions déclenchées (voir [ici](https://redux.js.org/advanced/middleware#the-final-approach) _sans le crashReporter_).


Nous allons maintenant faire communiquer plusieurs navigateurs entre eux gràce à [socket.io](https://socket.io/). Pour cela nous allons rajouter un middleware dédié. Sur un navigateur, quand la slide courante sera changée, un message sera envoyé aux autres navigateurs afin qu'ils changent eux aussi leur slide courante.

Côté serveur, importez `socket.io` ([tuto officiel](https://socket.io/get-started/chat/)) et mettez en place le callback permettant de recevoir les messages `set_slide` provenant d'un client et de les propager à tous les autres clients. Ce [guide permet de créer et tester une micro-application express utilisant socket.io](https://devcenter.heroku.com/articles/node-websockets#option-2-socket-io) en local et sur Heroku.


Côté client créez un [Middleware](https://redux.js.org/advanced/middleware#the-final-approach) dans lequel vous importerez `socket.io-client`. Le middleware devra, dès qu'il intercepte une action de type `SET_SLIDE`, propager un message adéquat via le socket, avant de faire appel à `next(action)`

Toujours dans le middleware, configurez la socket pour qu'à la réception des messages `set_slide`, les actions soient dispatchées au store.

```js
const propagateSocket = store => next => action => {
  if (action.meta.propagate) {
    if (action.type === SET_SLIDE) {
        socket.emit('action', {type:'set_slide', value: action.hash})
      }
  }
  next(action)
}
```

```js

socket.on('action', (msg) => {
  console.log('action',msg)
  switch (msg.type) {
    case 'set_slide':
      store.dispatch(setSlide(msg.value, false))
      break

  }
})
```

Vous remarquerez sans doute qu'au point où nous en sommes nous allons provoquer une boucle infinie d'émissions de messages. Pour éviter cela, les actions `SET_SLIDE` peuvent embarquer un information supplémentaire grâce [la propriété `meta`](https://github.com/redux-utilities/flux-standard-action#meta). Faites en sorte que seuls les dispatchs provenant d'un clic sur un bouton ou d'une modification de l'URL provoquent la propagation d'un message via Websocket.

N'oubliez pas d'utiliser `applyMiddleware` lors de la création du votre store. Si vous avez précédement installé le devtool Redux, référez-vous [à cette page](http://extension.remotedev.io/#12-advanced-store-setup) pour modifier de nouveau votre code.

```js
const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose

const store = createStore(rootReducer, composeEnhancers(applyMiddleware(propagateSocket)));
```
