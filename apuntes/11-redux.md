# Redux

Cuando el estado compartido en nuestra aplicación crece a partir de cierto
tamaño, puede ser muy complicado gestionarlo a través de uno o varios context.
En estos casos puede resultar util utilizar un sistema más estructurado para la
gestión del estado. Existen múltiples alternativas, pero sin duda la más famosa
es [Redux](https://es.redux.js.org/).

**Redux no es imprescindible para vuestros proyectos**, ya que con sólo el
context puede quedar simple y sencillo. Aún así, resulta útil conocerlo, y
si se os hace más cómodo, podéis utilizarlo.

## Cómo funciona Redux

Antes de instalar nada, analicemos el funcionamiento de Redux. Se basa en los
siguientes conceptos:

- **Store único**\
  Todo el estado _compartido_ de la aplicación se mueve a un único lugar, que
  denominamos _store_. De ahí podremos leerlo en donde sea necesario.
- **Actions**\
  Para cambiar el store, no le asignamos un nuevo valor, sino que enviamos una
  estructura llamada _action_, que representa una acción que sucede en nuestra
  aplicación, como por ejemplo: crear post, iniciar sesión, eliminar favorito.
- **Reducers**\
  El store es manejado por unas funciones _puras_ llamadas _reducers_, que
  reciben todas las acciones, y deciden si es necesario generar un nuevo estado
  o devuelven el mismo (en caso de que la acción no les afecte).

Desde un punto de vista más mecánico, podemos imaginar un único estado (el
store), manejado por trozos, a través de unas funciones que generan el
siguiente valor a partir del actual y del action que se ha producido.

Además, el store puede configurarse con _middlewares_ que le añadan
funcionalidad extra, como guardar en _local storage_, o incluso viajar en el
tiempo (rebobinando o avanzando rápido los diferentes estados).

## Configurar Redux

Para configurar Redux en React, utilizaremos
[React-Redux](https://react-redux.js.org/). Lo primero es instalar los dos
paquetes:

```sh
npm install redux react-redux
```

A continuación, desde el `index.js` creamos el store y ponemos su `Provider`.

```jsx
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import rootReducer from './reducers'
// ...

const store = createStore(rootReducer)

ReactDOM.render(
  // ...
    <Provider store={store}>
      <App />
    </Provider>
  // ...

```

El `rootReducer` deberemos definirlo en otro fichero. Es frecuente utilizar
`combineReducers` para que el store sea un objeto y manejemos cada _key_ del
mismo mediante un reducer más pequeño. En ese caso, el código quedaría:

```jsx
const userReducer = (state = null, action) => {
  // your logic here...
}

const rootReducer = combineReducers({
  user: userReducer,
  // ...
})

export default rootReducer
```

## Usando Redux

Una vez tengamos todo configurado, podemos acceder al store mediante
`useSelector`. Este hook recibe una función que, dado un estado del store,
accede al trozo que nos interesa y lo devuelve. Por ejemplo, para hacer un
selector que obtenga el usuario actual, haríamos:

```jsx
function User() {
  const currentUser = useSelector(s => s.user)
  // ...
}
```

Por otro lado, para lanzar una action, debemos utilizar `useDispatch` para
obtener la función que nos permite lanzarlas:

```jsx
function Login() {
  const dispatch = useDispatch()
  const handleSubmit = e => {
    // ... fetch y todo eso ...
    dispatch({ type: 'login', payload: data })
  }
}
```

Finalmente, es importante asegurarnos de que nuestro reducer pueda entender
esta acción y modifique el estado según sea necesario:

```jsx
const userReducer = (state = null, action) => {
  switch (action.type) {
    case 'login':
      return action.payload
    case 'logout':
      return null
    default:
      return state
  }
}
```

## Usando middlewares

Los middlewares son relativamente complejos, por lo que normalmente mientras
estamos aprendiendo nos limitaremos a utilizar algunos de los que nos ofrecen
en la web de Redux, o en npm. Uno muy popular que es posible copiar es el de
recordar la sesión de usuario:

```jsx
const localStorageMiddleware = store => next => action => {
  let result = next(action)
  localStorage.setItem('session', JSON.stringify(store.getState().user))
  return result
}

const saved = localStorage.getItem('session')
const initialStore = { user: saved ? JSON.parse(saved) : undefined }
const store = createStore(rootReducer, initialStore, applyMiddleware(localStorageMiddleware))
```

## A continuación...

A partir de aquí, nos falta ver algunos otros [elementos](./12-misc.md) de
React que todavía no hemos utilizado.
