# Hooks

Hasta ahora hemos utilizado los dos hooks más importante (`useState` y
`useEffect`) sin haber profundizado en qué son los hooks, ni qué reglas siguen.
Antes de empezar a crear nuestros propios hook, veamos estos puntos.

## Reglas de hooks

Los hooks son funciones especiales que encapsulan funcionalidad para que
podamos reutilizarla entre componentes. Si los componentes son una forma de
hacer interfaces reutilizables, los hooks son la forma de hacer comportamientos
reutilizables entre componentes.

Desde un punto de vista técnico, los hooks son funciones que cumplen las
siguientes reglas:

- **Su nombre empieza por "use..."**\
  Igual que los componentes siempre empiezan por mayúsculas, los hooks deben
  empezar siempre por `use...`, como `useEffect` o `useState`. Si no empiezan
  de este modo, serán funciones normales y no hooks.
- **Sólo pueden llamarse desde componentes o otros hooks**\
  Las funciones normales pueden llamarse desde cualquier sitio, sin
  restricciones, pero los hooks tienen ciertas limitaciones. No podremos llamar
  un hook desde, por ejemplo, un event handler o un temporizador. Sólo desde el
  cuerpo principal de un componente, o desde otro hook.
- **La llamada a un hook no puede ser condicional o repetitiva**\
  Los hooks no se pueden llamar desde un `if` ni desde un `for`. Podemos llamar
  varias veces a un hook, pero sólo de forma manual. Esto es así porque React
  internamente lleva la cuenta de cuántas veces se ha llamado el hook para
  devolver el valor que le corresponde, sin mezclar estados o efectos.

React incluye por defecto un mecanismo que tratará de avisarnos si incumplimos
alguna de estas reglas.

Estas reglas son las únicas restricciones que debemos cumplir, por lo que
tenemos total flexibilidad en todo lo demás. Por ejemplo, los hooks pueden
recibir cualquier cantidad de parámetros, o devolver cualquier tipo de valor.

## Hooks custom

Siempre que cumplamos las anteriores reglas, podremos crear hooks
personalizados para _encapsular_ funcionalidad que repitamos de forma común en
distintos componentes o proyectos. Veamos algunos ejemplos concretos:

### Carga de datos

En el anterior tema vimos que es muy común utilizar una combinación de
`useEffect` y `useFetch` para cargar datos de cualquier API al montar un
componente. Esta funcionalidad es quizá el caso más claro de función sencilla
de encapsular y reutilizar.

```jsx
const useFetch = (url) => {
  const [data, setData] = useState()
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(json => setData(json))
  }, [url])
  return data
}

const RickCharacter = ({ id = 1 }) => {
  const data = useFetch(`https://rickandmortyapi.com/api/character/${id}`)
  if (!data) return <div>Cargando...</div>
  return (
    <div className="character">
      Nombre: {data.name}<br />
      Especie: {data.species}<br />
    </div>
  )
}
```

### Eventos externos

Para hacer efectos visuales relacionados con el tamaño de pantalla o el scroll,
es común crear hooks que nos devuelvan propiedades del navegador, y nos avisen
cuando cambien. Veamos cómo extraer la funcionalidad de `ScreenWidth` del tema
anterior a un hook custom.

```jsx
const useWidth = () => {
  const [width, setWidth] = useState(window.innerWidth)
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth)
    window.addEventListener('resize', handleResize)
    return () => window.removeEventListener('resize', handleResize)
  }, [])
  return width
}

const ScreenWidth = () => {
  const width = useWidth()
  return <div>Ancho: {width}px</div>
}
```

### Temporizadores

En el caso de los temporizadores, no hay casos tan salientables como en los
apartados anteriores, pero aún así extraer la funcionalidad a un hook nos puede
ayudar a simplificar el código, y poner nombre a un comportamiento, en lugar de
tenerlo mezclado en un componente.

Veamos cómo podríamos hacerlo en el caso de `Countdown`:

```jsx
const useCountdown = (start = 10) => {
  const [remaining, setRemaining] = useState(start)
  useEffect(() => {
    const id = setInterval(() => {
      setRemaining(r => r > 0 ? r - 1 : r)
    }, 1000)
    return () => clearInterval(id)
  }, [])
  return remaining
}

const Countdown = () => {
  const remaining = useCountdown(30)
  return <div>Cuenta atrás: {remaining}</div>
}
```

## A continuación...

Llegados a este punto, hemos visto la mayor parte de funcionalidad que React
nos proporciona. En los próximos temas nos centraremos en ver algunas librerías
que es común utilizar con React, y entraremos a ver las funcionalidades que nos
faltan, que se usan con menor frecuencia, pero pueden resultarnos útiles en
algunos casos.

Empezaremos por ver cómo funciona el [routing](./09-routing.md) con
React-Router.
