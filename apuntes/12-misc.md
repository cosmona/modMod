# Miscelanea

Hemos visto las funciones principales de React, y algunas librerías muy usadas
en el ecosistema React. Sin embargo, hay varias funciones útiles que todavía
no hemos mencionado, que agruparemos en este último tema.

## Referencias

Las referencias tienen varios usos, pero nos centraremos en el más habitual:
nos permiten **acceder a un nodo del DOM** sin necesidad de utilizar herramientas
estilo _querySelector_. Es especialmente útil cuando se usa con elementos como
_video_ o _audio_, que tienen métodos imperativos que podemos necesitar usar.

Para crear una referencia, usamos el hook `useRef`, y para asignarle un
elemento, tan sólo lo pasamos como parámetro `ref` al mismo. Por último, para
acceder al nodo lo haremos mediante su propiedad `current`:

```jsx
import { useRef } from 'react'

function Video() {
  const ref = useRef()
  return (
    <div>
      <video ref={ref} src="..." />
      <button onClick={() => ref.current.play()}>Play</button>
    </div>
  )
}
```

## Error boundaries

Los error boundary nos permiten _contener_ un error _no controlado_ a sólo una
parte de la aplicación, mientras el resto sigue funcionando sin interrupciones.
Por ahora no existe una versión con hooks para implementar el boundary, por lo
que tenemos que utilizar un componente _clásico_ (de tipo clase).

Habitualmente se utiliza uno como este, y se reutiliza en todas partes:

```jsx
import React from 'react'

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props)
    this.state = { hasError: false }
  }

  static getDerivedStateFromError(error) {
    return { hasError: true }
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <h1>Something went wrong.</h1>
    }
    return this.props.children
  }
}

export default ErrorBoundary
```

A la hora de utilizarlo, podemos colocarlo en cualquier lugar de la aplicación,
e incluso podemos colocar varios a diferentes niveles, para capturar errores en
distintos niveles de granularidad.

Cada boundary atrapará errores sólo en los componentes hijo, pero
**no en el propio componente que lo añade**.
Por eso, frecuentemente se crea un _wrapper_ del componente que añade el
boundary como padre:

```jsx
function MyComponent() { /* ... */ }

const MyComponentWrapper = () =>
  <ErrorBoundary fallback={<SomeFallback />}>
    <MyComponent />
  </ErrorBoundary>

export default MyComponentWrapper
```

## Suspense

Existe en React un componente especial llamado `<Suspense>` que funciona de
forma muy similar a los error boundaries, pero en este caso se centra en
manejar los **estados de cargando** de la aplicación. _Oficialmente_ por ahora
sólo se utilizan para carga de código asíncrona, pero existen muchas
[soluciones](https://github.com/CharlesStover/fetch-suspense) no oficiales que
nos permiten utilizarlo para carga de datos.

Utilizaremos el `useFetch` de la librería del enlace como ejemplo, usando un
_wrapper_ como hicimos con los _boundary_:

```jsx
function Character() {
  const character = useFetch('https://rickandmortyapi.com/api/character/1')
  // A partir de aquí, character ya está cargado, sin necesidad de esperas o checks
  return <div>{character.name}</div>
}

const CharacterWrapper = () =>
  <Suspense fallback={<Loading />}>
    <Character />
  </Suspense>

export default CharacterWrapper
```

Esto nos simplifica mucho el manejar elegantemente los estados de carga, y nos
ahorra verificar si ya están disponibles los datos, ya que nuestro componente
sólo se ejecutará en el momento que tenga los datos.

Esto es especialmente importante en **formularios donde editamos** cualquier
entidad existente, ya que nos permitirá inicializar `useState` a los valores
actuales sin necesidad de complicaciones:

```jsx
function Character() {
  const { id } = useParams()
  const character = useFetch('https://rickandmortyapi.com/api/character/' + id)

  // Como los datos ya están disponibles en el primer render,
  // useState los puede utilizar como valor inicial
  const [name, setName] = useState(character.name || '')
  const [species, setSpecies] = useState(character.species || '')

  return (
    <form onSubmit={/* ... */}>
      <input value={name} onChange={e => setName(e.target.value)} />
      <input value={species} onChange={e => setSpecies(e.target.value)} />
      <button>Guardar</button>
    </div>
  )
}
```
