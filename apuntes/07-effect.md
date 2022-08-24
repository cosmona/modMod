# Efectos secundarios

La mayoría de componentes tan sólo deben preocuparse de lo que muestran en la
página (es decir: el valor que devuelven), y quizá de algún estado propio o de
sus hijos. Sin embargo, unos pocos componentes tendrán que, además, interactuar
con elementos externos, como librerías, APIs del navegador, etc.

Llamamos efectos secundarios a las llamadas que un componente hace a elementos
externos como consecuencia de ser _montados_ en la página, o de actualizarse
debido a un cambio.

Inicialmente, podríamos pensar en incluir estos efectos secundarios como parte
de la función de nuestro componente, pero esto causaría que se vuelvan a
ejecutar siempre que el componente se actualice por cualquier motivo. Para
tener un control más granular de esto, aparece el hook `useEffect`.

## Cómo funciona `useEffect`

Al igual que con `useState`, el primer paso será importar `useEffect` del
módulo React. A continuación, para usarlo se le pasa una función que lleve a
cabo el efecto secundario de nuestro componente, y un array de _dependencias_
que indique cuando debe volverse a ejecutar dicha función. Es muy frecuente
utilizar un _array vacío_ como valor cuando no existe ninguna dependencia.

```jsx
import { useEffect } from 'react'

const MyComponent = () => {
  useEffect(() => {
    // Esto sólo se ejecuta una vez, aunque el componente se re-renderice.
    console.log('Entrando...')
  }, []) // No hay dependencias: utilizamos array vacío
  return <div>...</div>
}
```

## Dependencias

Casi siempre que utilicemos un valor dentro del _callback_ de useEffect, y este
valor cambie (por ejemplo, si es un _prop_ o _state_), querremos _actualizar_
la anterior llamada a useEffect, ejecutando de nuevo la función. Para ello,
sólo necesitamos incluir dichos valores en el array de dependencias, y React
volverá a ejecutar automáticamente nuestro callback cada vez que algún valor
del array cambie.

```jsx
const MyComponent = ({ itemId }) => {
  useEffect(() => {
    console.log('Cargando datos para item:', itemId)
    // Esta función se volverá a ejecutar cada vez que itemId cambie,
    // pero no se ejecutará si cambia otro prop o state.
  }, [itemId])
  return <div>...</div>
}
```

## Cleanup o limpieza

Algunos efectos secundarios requieren ser _"limpiados"_ cuando ya no son
necesarios, para que no sigan consumiendo recursos, o para detener su
comportamiento. En estos casos, desde nuestro _callback_ de _useEffect_ podemos
**devolver otra función** para indicar a React que debe ejecutarla únicamente
cuando el componente desaparezca, o vaya a volver a ejecutarse el callback con
nuevos valores.

```jsx
const MyComponent = () => {
  useEffect(() => {
    // Se imprimirá esto cuando el componente es montado
    console.log('Entrando...')
    return () => {
      // Se imprimirá esto cuando el componente es desmontado
      console.log('Saliendo...')
    }
  }, [])
  return <div>...</div>
}
```

## Casos típicos

Existen varios casos típicos en los que es necesario `useEffect`, que siempre
se manejan de la misma forma, y es bueno conocer a modo de _recetas_ para
aplicar cuando nos hagan falta. Veremos los más frecuentes a continuación.

### Temporizador: Cuenta atrás

Para mostrar una cuenta atrás, necesitaremos un **state** con el tiempo que
falta, y un useEffect para manejar un temporizador que lo actualice cada cierto
tiempo. Es muy importante acordarnos de **limpiar cualquier temporizador**
desde la función de cleanup.

En este caso, desde el temporizador necesitamos llamar a un setState, para lo
cual es necesario tener el valor actual, pero una vez creado el temporizador,
la función que ejecuta no se actualiza, y por tanto va a
**seguir viendo los valores de cuando se estableció**.
Para evitarlo tenemos dos opciones:

- Poner el valor actual como dependencia.
  Esto hará que, cada vez que cambie el valor actual, se elimine el anterior
  temporizador (con el cleanup), y se ponga uno nuevo (ejecutando el callback).
  Esto funciona, pero es muy ineficiente, y no se recomienda si se puede evitar.
- Usar la versión _callback_ de setState
  Al utilizar esta opción, no necesitamos acceso al valor actual,ya que React
  nos lo pasará cada vez que sea necesario. Esto evita añadir una dependencia
  y es en general la opción recomendada.


```jsx
const Countdown = () => {
  const [remaining, setRemaining] = useState(10) // de 10 a 0
  useEffect(() => {
    const id = setInterval(() => {
      setRemaining(r => r > 0 ? r - 1 : r) // No dejamos que baje de 0
    }, 1000)
    return () => clearInterval(id)
  }, [])
  return <div>Cuenta atrás: {remaining}</div>
}
```

### Temporizador: mensaje tras cierto tiempo

En ocasiones queremos que un componente muestre algo diferente cuando haya
pasado cierto tiempo. Este caso es relativamente sencillo con setTimeout.

```jsx
const Delayed = () => {
  const [finished, setFinished] = useState(false)
  useEffect(() => {
    const id = setTimeout(() => {
      setFinished(true)
    }, 10 * 1000)
    return () => clearTimeout(id)
  }, [])
  return finished ? <div>Se acabó el tiempo!</div> : <div>Espera 10 seg.</div>
}
```

### Carga de datos

Tal vez el caso más frecuente de todos: un componente que necesita cargar datos
para mostrarlos. Necesitamos un state para guardar los datos que vamos a
mostrar, y es imprescindible **controlar el estado cargando**, ya que el fetch
no es instantáneo, y si intentamos acceder a los datos antes de tenerlos, nos
dará un error.

```jsx
const RickCharacter = ({ id = 1 }) => {
  const [data, setData] = useState()
  useEffect(() => {
    fetch(`https://rickandmortyapi.com/api/character/${id}`)
      .then(res => res.json())
      .then(json => setData(json))
  }, [id])
  if (!data) return <div>Cargando...</div>
  return (
    <div className="character">
      Nombre: {data.name}<br />
      Especie: {data.species}<br />
    </div>
  )
}
```

### Manejo de eventos externos

Cuando manejamos eventos en nuestros componentes, React se ocupa de hacer las
llamadas a `addEventListener` por nosotros. Sin embargo, hay ocasiones donde
esto no es posible, por ejemplo si el elemento que emite los eventos no es
manejado por React. Los casos más frecuentes son con los eventos de _scroll_ y
_resize_ del navegador.

En estos casos vuelve a ser imprescindible **eliminar el event listener**
cuando ya no sea necesario, y es frecuente que necesitemos guardar algún dato
sobre el evento en un _state_.

```jsx
const ScreenWidth = () => {
  const [width, setWidth] = useState(window.innerWidth)
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth)
    window.addEventListener('resize', handleResize)
    return () => window.removeEventListener('resize', handleResize)
  }, [])
  return <div>Ancho: {width}px</div>
}
```

## A continuación...

Una vez conocemos en profundidad los dos hooks más importantes, podemos
comprender mejor las reglas de los hooks, y empezar a crear nuestros propios
[hooks](./08-hooks.md) personalizados.
