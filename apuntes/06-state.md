# State

Para poder hacer que nuestros componentes puedan mantener valores, así como
actualizarse cuando cambien, necesitamos introducir el concepto de _estado_.

## Qué es el estado

El _estado_ de un componente no es más que el conjunto de valores almacenados de
los que es responsable, en contraposición a los _props_, que son valores que
recibe de otros componentes.

Por ejemplo, un componente `<Switch>`, que muestra un interruptor que se puede
activar y desactivar, tendrá como estado su valor actual (encendido o apagado),
mientras que un componente `<Login>` podría tener como estado dos valores: el
nombre de usuario y la contraseña que el usuario introduce.

## Cómo manejar el estado en React

De primeras podríamos pensar que utilizar una variable normal (o varias) para
guardar el estado sería suficiente, pero si lo hiciésemos así, React no podría
saber cuando cambian para actualizar nuestro componente. Para poder monitorizar
estos valores, React nos proporciona un _hook_ llamado `useState`.

Veremos más detalle qué son los _hooks_ en un tema posterior, ya que ahora
podemos utilizar este sin necesidad de profundizar más.

El hook `useState` recibe el **valor inicial** del estado (si es que lo hay), y
nos devuelve un array con dos elementos: el **valor actual** del estado, y una
**función de actualización** para cambiar su valor. Lo habitual es capturar
ambos valores mediante destructuring, poniéndoles como nombres algo como
_valor/setValor_.

Veamos un ejemplo:

```jsx
import { useState } from 'react'
// Es necesario importar el hook en cada fichero que lo vayamos a utilizar

const Contador = () => {
  const [count, setCount] = useState(0) // Valor inicial 0
  // count nos indica el valor de la cuenta en este instante
  // setCount nos permite actualizar el valor de count

  // Al recibir un click, sumamos uno a count
  const handleClick = () => setCount(count + 1)

  return (
    <button onClick={(handleClick)}>
      {count}
    </button>
  )
}
```

Es aconsejable utilizar siempre `const` para las variables que nos devuelve
`useState`, como en el ejemplo anterior, para evitar errores. De primeras
podría parecer que al hacer que el valor actual sea una constante, no podrá
cambiar, pero no es así: en realidad, cuando llamemos a `setCount`, nuestro
componente **volverá a ser ejecutado** desde el principio, con un nuevo valor
en dicha constante.

## Múltiples estados

Nada impide a un componente tener múltiples estados independientes:

```jsx
const Reactions = () => {
  const [likes, setLikes] = useState(0)
  const [loves, setLoves] = useState(0)
  const [sads, setSads] = useState(0)

  return (
    <div className="reactions">
      <button onClick={() => setLikes(likes + 1)}>{likes} 👍</button>
      <button onClick={() => setLoves(loves + 1)}>{loves} ❤️</button>
      <button onClick={() => setSads(sads + 1)}>{sads} 😞</button>
    </div>
  )
}
```

Cada vez que llamamos a `useState`, se crea un nuevo estado, con su propio
valor y nombres, que podemos utilizar en el componente.  Cada vez que
cualquiera de los estados se actualice, el componente se refrescará,
manteniendo el valor del resto de estados.

## State y efectos visuales

Es muy común cambiar las clases CSS en función del estado para producir
un efecto visual. Por ejemplo, el componente `<Spoiler>` que mencionamos
anteriormente podría implementarse así:

```jsx
const Spoiler = ({ children }) => {
  const [show, setShow] = useState(false) // Por defecto está oculto
  // show nos indica si ahora mismo este componente está "revelado"
  // setShow nos permite actualizar el valor de show

  const className = 'spoiler ' + (show ? 'visible' : 'hidden')
  const handleClick = () => setShow(!show)

  return (
    <div className={className} onClick={handleClick}>
      {children}
    </div>
  )
}
```

Este componente por defecto tendrá la clase `spoiler hidden` hasta que se haga
click en él, que pasará a `spoiler visible`. Añadiendo un CSS que vele el
contenido mientras esté `hidden`, tendríamos el funcionamiento completado.

## Input y formularios

Aunque existen alternativas, es habitual sincronizar cada input con un state,
de forma que tengamos el valor disponible donde sea necesario (por ejemplo,
en el handler de _submit_) sin leerlo a través del DOM. Aunque con sólo manejar
el _onChange_ funcionaría, para asegurarnos que el valor del input se actualiza
también si es cambiado externamente, solemos utilizar el atributo value.

```jsx
const LoginForm = () => {
  const [username, setUsername] = useState('')
  const [password, setPassword] = useState('')
  const handleSubmit = e => {
    e.preventDefault()
    console.log('Iniciar sesión:', username, password)
    // fetch(...)
  }
  return (
    <form onSubmit={handleSubmit}>
      <label>
        Usuario:
        <input value={username} onChange={e => setUsername(e.target.value)} />
      </label>
      <label>
        Contraseña:
        <input value={password} onChange={e => setPassword(e.target.value)} />
      </label>
      <button>Iniciar sesión</button>
    </form>
  )
}
```

## State y props

En ocasiones, varios componentes necesitan compartir un mismo estado. Dado que
cada estado sólo existe en un componente, en estos casos es necesario _subir_
el estado al componente padre de todos los que necesiten el valor, y pasarlo a
cada lugar donde se necesita mediante _props_.

```jsx
const App = ({ children }) => {
  const [user, setUser] = useState()

  return (
    <div className="app">
      <Header user={user} />
      {user ? <Welcome /> : <Login setUser={setUser} />}
    </div>
  )
}
```

## Usos avanzados de _useState_

En algunos casos, es posible que no podamos utilizar `useState` de la forma
habitual. Para esas situaciones, existen algunas opciones extra que podemos
utilizar:

- **Estado inicial costoso**\
  Cuando el estado inicial es muy lento de calcular (por ejemplo, listas muy
  grandes), podemos pasar como valor una función que lo calcule, para que React
  la ejecute una sóla vez, y no cada vez que el componente se refresca.

  ```jsx
  const [list, setList] = useState(() => {
    // Esta función sólo se ejecutará una vez
    return slowFunctionReturningList()
  })
  ```

- **Estado actual no disponible**\
  Si trabajamos con librerías externas, o ciertas funciones del navegador,
  es posible que no tengamos disponible el valor actual del estado para
  actualizarlo. En estos casos, podemos llamar el _setter_ del estado con una
  función que reciba el valor actual, y devuelva el nuevo. React se ocupará de
  pasarle el valor actualizado, y recoger el que devuelva.

  ```jsx
  const LibWrapper = () => {
    const [count, setCount] = useState(0)
    const handleUpdate = () => setCount(c => c + 1)
    // Este handleUpdate sumará uno al state, incluso si no
    // se actualiza cada vez que cambie de valor.
    return (
      <SomeExternalLib onUpdate={handleUpdate} />
    )
  }
  ```

## A continuación...

El manejo de estado de React es una de las partes más complicadas de
interiorizar, ya que supone un cambio muy grande respecto a la programación
imperativa tradicional a la que estamos acostumbrados. Sin embargo, sin él
es imposible hacer cualquier aplicación que no sea completamente trivial.

Es aconsejable hacer **muchos** ejercicios trabajando con `useState` antes
de pasar al siguiente _hook_: [useEffect](./07-effect.md).
