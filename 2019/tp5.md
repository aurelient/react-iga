## TP5 Modalité d’entrées (gestes, stylet)

### Gestion de modalités d'entrée

Nous allons maintenant ajouter des fonctions de dessin à nos slides. En utilisant un stylet, un utilisateur pourra mettre en avant des elements sur la slide courante, et ce de manière synchronisée avec les autres appareils.

**Pour simplier on ne dessine que sur la slide courante, et on efface/oublie le dessin quand on change de slide.**

#### Création d'un canvas sur lequel dessiner
Pour cette partie, nous prendrons exemple sur ce tutoriel [W. Malone](http://www.williammalone.com/articles/create-html5-canvas-javascript-drawing-app/#demo-simple).

Dans un premier temps, dans le composant `Slide` ajoutez un élément `canvas` avec avec les handlers d'événements onPointerDown, onPointerMove et onPointerUp ainsi qu'en déclarant une [Référence React](https://reactjs.org/docs/hooks-reference.html#useref). Utilisez `useRef`si vous êtes dans un 'function component', ou `createRef` si vous êtes dans un 'class-based component' ([voir ici](https://stackoverflow.com/a/54620836)):

```jsx
<canvas className="stroke"
          ref={refCanvas}
          onPointerDown={pointerDownHandler}
          onPointerMove={pointerMoveHandler}
          onPointerUp={pointerUpEvent}></canvas>
```



Ces handlers nous permettront de d'écouter les événements provenant de `pointer`.

Afin de vous faciliter la tâche, voici le code *presque* complet pour faire marcher le dessin sur le canvas.

Assurez-vous de bien faire les imports nécessaires au bon fonctionnement du code ci-dessous. Faites en sortes que l'on ne dessine que si c'est un [stylet qui est utilisé](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent/pointerType).



```js
  var clickX = new Array()
  var clickY = new Array()
  var clickDrag = new Array()
  var paint = false

  // Cette ligne permet d'avoir accès à notre canvas après que le composant aie été rendu. Le canvas est alors disponible via refCanvas.current
  // Si vous utilisez des Class Components plutôt que des function Components, voir ici https://stackoverflow.com/a/54620836
  let refCanvas = useRef(null)

  function addClick(x, y, dragging)
  {
    clickX.push(x),
    clickY.push(y),
    clickDrag.push(dragging)
  }

  function redraw(){
    let context = refCanvas.current.getContext('2d')
    let width = refCanvas.current.getBoundingClientRect().width
    let height = refCanvas.current.getBoundingClientRect().height

    //Ceci permet d'adapter la taille du contexte de votre canvas à sa taille sur la page
    refCanvas.current.setAttribute('width', width)
    refCanvas.current.setAttribute('height', height)
    context.clearRect(0, 0, context.width, context.height); // Clears the canvas

    context.strokeStyle = "#df4b26";
    context.lineJoin = "round";
    context.lineWidth = 2;

    for(var i=0; i < clickX.length; i++) {
      context.beginPath();
      if(clickDrag[i] && i){
      context.moveTo(clickX[i-1]*width, clickY[i-1]*height);
       }else{
         context.moveTo((clickX[i]*width)-1, clickY[i]*height);
       }
       context.lineTo(clickX[i]*width, clickY[i]*height);
       context.closePath();
       context.stroke();
    }
  }

  function pointerDownHandler (ev) {

    console.error('HEY ! ICI ON PEUT DIFFERENCIER QUEL TYPE DE POINTEUR EST UTILISE !')

    let width = refCanvas.current.getBoundingClientRect().width
    let height = refCanvas.current.getBoundingClientRect().height
    var mouseX = (ev.pageX - refCanvas.current.offsetLeft)/width
    var mouseY = (ev.pageY - refCanvas.current.offsetTop)/height

    paint = true
    addClick(mouseX, mouseY, false)
    redraw()
  }

  function pointerMoveHandler (ev) {
    if(paint){
      let width = refCanvas.current.getBoundingClientRect().width
      let height = refCanvas.current.getBoundingClientRect().height
      addClick((ev.pageX - refCanvas.current.offsetLeft)/width, (ev.pageY - refCanvas.current.offsetTop)/height, true)
      redraw()
    }
  }

  function pointerUpEvent (ev) {
    paint = false
  }

```


### Lien du canvas au store
Dans votre état initial, rajoutez l'attribut suivant :
```js
drawing: {
        clickX: [],
        clickY: [],
        clickDrag: []
    }
```

Et créez les actions `ADD_DRAW_POINTS` et `RESET_DRAW_POINTS`.

`ADD_DRAW_POINTS` devra accepter au moins 3 paramètres de type Array `(clickX, clickY, clickDrag)` qui seront concaténés à l'état du store.

`RESET_DRAW_POINTS` réinitialiseras les tableaux du store à vide.

Dans votre composant Slide, réalisez la connexion avec le store :

```js
const mapStateToProps = state => {
  return {
    drawing: state.drawing
  }
}

const mapDispatchProps = dispatch => {
  return {
    addPoints: (x, y, drag) => dispatch(addDrawPoints(x, y, drag, true))
  }
}
```

Une fois ceci fait, faites en sorte qu'à chaque fois qu'une ligne est finie de dessiner (`pointerUpEvent`), que vous copiez les points de la nouvelle ligne dans le store. Bien sûr, maintenant il faut aussi dessiner les lignes stockées dans le store (`props.drawing.`).

Ajoutez un bouton "Effacer" à votre toolbar, ce bouton déclenchera l'action `RESET_DRAW_POINTS`

### Syncronisation du canvas entre les appareils

Vous pouvez maintenant ajouter à votre Middleware de nouveaux cas permettant de propager les nouvelles lignes dessinées aux autres appareils.
```js
// ...
 else if (action.type === ADD_DRAW_POINTS) {
  socket.emit('action', {type: 'add_draw_points', value: {
      x: action.x,
      y: action.y,
      drag: action.drag
    }
  })
} else if (action.type === RESET_DRAW_POINTS) {
  socket.emit('action', {type: 'reset_draw_points'})
}
//...
```

Vous remarquerez qu'à l'ouverture sur un autre appareil, votre dessin n'apparait que si vous dessinez aussi sur cet appareil. Pour remédier à ce problème, utilisez [useEffect](https://reactjs.org/docs/hooks-effect.html) afin d'exécuter `redraw()` au moment opportun.



### Reconnaissance de gestes

Pour terminer, nous allons effectuer de la reconnaissance de geste lors d'évènements touch.

Pour ce faire nous allons utiliser le [$1 recognizer](http://depts.washington.edu/acelab/proj/dollar/index.html) vu en cours. Nous allons utiliser une version modifiée de [OneDollar.js](https://github.com/nok/onedollar-unistroke-coffee) pour fonctionner avec React. Il n'y a pas de module JS récent pour cette bibliothèque. Nous devrions donc le créer, mais pour plus de simplicité nous allons placer directement [la bibliothèque](../code/onedollar.js) dans le dossier `src/` pour qu'elle soit facilement bundlée par Webpack.


#### Gérer le recognizer

Au niveau de votre `Slide`, importer et initialiser votre le One Dollar Recognizer.

```js
// Voir ici pour le détails de options https://github.com/nok/onedollar-unistroke-coffee#options
const options = {
  'score': 80,  // The similarity threshold to apply the callback(s)
  'parts': 64,  // The number of resampling points
  'step': 2,    // The degree of one single rotation step
  'angle': 45,  // The last degree of rotation
  'size': 250   // The width and height of the scaling bounding box
};
const recognizer = new OneDollar(options);

// Let's "teach" two gestures to the recognizer:
recognizer.add('triangle', [[627,213],[626,217],[617,234],[611,248],[603,264],[590,287],[552,329],[524,358],[489,383],[461,410],[426,444],[416,454],[407,466],[405,469],[411,469],[428,469],[453,470],[513,478],[555,483],[606,493],[658,499],[727,505],[762,507],[785,508],[795,508],[796,505],[796,503],[796,502],[796,495],[790,473],[785,462],[776,447],[767,430],[742,390],[724,362],[708,340],[695,321],[673,289],[664,272],[660,263],[659,261],[658,256],[658,255],[658,255]]);
recognizer.add('circle', [[621,225],[616,225],[608,225],[601,225],[594,227],[572,235],[562,241],[548,251],[532,270],[504,314],[495,340],[492,363],[492,385],[494,422],[505,447],[524,470],[550,492],[607,523],[649,531],[689,531],[751,523],[782,510],[807,495],[826,470],[851,420],[859,393],[860,366],[858,339],[852,311],[833,272],[815,248],[793,229],[768,214],[729,198],[704,191],[678,189],[655,188],[623,188],[614,188],[611,188],[611,188]]);
```

#### Traitement différencié selon le type du pointerEvent.
Etendre les fonctions `pointerDownHandler`, `pointerMoveHandler`, `pointerUpHandler` pour qu'elles traite différemment les sources `touch`, `pen` et `mouse`.

Nous allons associer les gestes au `touch`. Toutefois pour débugger plus facilement, vous pouvez commencer traiter les gestes sur le pointerEvent `mouse`, et basculer sur le touch une fois que cela marche bien.

Stocker les points composants le geste dans un Array `gesturePoints`.

#### Dessiner le geste
Dans la fonction de dessin `redraw` vous pouvez ajouter un cas à la fin qui dessine en cas de geste (les points composant le geste sont stockés dans `gesturePoints`).
Vous devrez être **vigilant à convertir vos points pour être dans le référentiel du canvas**, comme dans le code fournit ci-dessus.

```js
  function redraw(){

    ...

    if (gesture) {
      context.strokeStyle = "#666";
      context.lineJoin = "round";
      context.lineWidth = 5;

      context.beginPath();
      context.moveTo(gesturePoints[0][0]*width, gesturePoints[0][1]*height);
      for(var i=1; i < gesturePoints.length; i++) {
        context.lineTo(gesturePoints[i][0]*width-1, gesturePoints[i][1]*height);
      }

      context.stroke();
    }
  }
```

#### Reconnaitre un geste prédéfinit

Quand le geste se termine (`pointerUpHandler`), vous pouvez lancer la reconnaissance du geste.

```js
      let gesture = recognizer.check(gesturePoints);
```

Inspectez l'objet gesture dans la console, et vérifiez que vous arrivez bien à reconnaiter un cercle et un triangle.

Pensez à réinitialiser `gesturePoints` une fois le geste terminé.

#### Apprendre de nouveaux gestes

Toujours dans `pointerUpHandler`, vous pouvez imprimer les trajectoires correspondants à des gestes.

```js
    console.log('[['+gesturePoints.join('],[')+']]')
```

Utiliser cette sortie pour ajouter deux nouveaux gestes: '>' et '<' (partant du haut vers le bas) à votre recognizer.


#### Associer le geste à une action

Une fois le geste exécute, s'il correspond à un de ces deux nouveaux gestes (`recognized == true`), dispatcher les actions suivant ou précédent.

Pour faire cela, nous allons nous appuyer sur un `mapDispatchToProps` qu'il faudra connecter à votre composant.

```js
const matchDispatchProps = dispatch => {
  return {
    nextSlide: () => {
      dispatch(setSlide(store.getState().currentSlide+1, true))
      dispatch(resetDrawPoints(true))
    },
    previousSlide: () => {
      dispatch(setSlide(store.getState().currentSlide-1, true))
      dispatch(resetDrawPoints(true))
    },
    resetDrawPoints: () => dispatch(resetDrawPoints(true))
  }
```

`resetDrawPoints` est l'action associée à l'effaçage des dessins effectué sur le transparent.

Vérifier que l'action est bien distribuée sur tous les dispositifs connectés.
