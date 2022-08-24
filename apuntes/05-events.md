# Eventos

Al igual que vimos cuando trabajamos con el DOM, en React podemos capturar
eventos para reaccionar a la interacción del usuario. Veamos cómo se utilizan.

## Capturando un evento

El sistema para capturar eventos en React es muy similar a cómo funcionan en
HTML, con la principal diferencia de que al escribir todo el código en un
mismo fichero, las referencias son mucho más claras:

```jsx
const Button = () => {
  const handleClick = () => {
    alert('Click!')
  }
  return (
    <div onClick={handleClick}>
      Click me!
    </div>
  )
}
```

Es importante recordar que todos los atributos en React se escriben con
camelCase y son sensibles a mayúsculas. Los nombres de los eventos son
exactamente los mismos que en HTML y el DOM (incluyendo el `on...`).

Las funciones manejadoras de eventos pueden llamarse como queramos, o incluso
ser funciones anónimas, y podemos definirlas en cualquier lugar que en el que
sean visibles desde nuestro código, aunque es muy poco común definirlas fuera
del componente. Esto es así porque sólo escribiéndolas dentro del componente
tendrán acceso a los **props**, debido a las reglas de _scoping_.

```jsx
const Product = ({ title }) => {
  return (
    <div className="product" onClick={() => alert(title)}>
      {title}
    </div>
  )
}
```

Al igual que cuando atrapábamos eventos con _addEventListener_, nuestros
manejadores reciben un objeto _event_ con toda la información sobre el evento
que se ha producido:

```jsx
const Name = () => {
  const handleChange = e => {
    console.log('El texto ahora es:', e.target.value)
  }
  return (
    <label>
      Tu nombre:
      <input onChange={handleChange} />
    </label>
  )
}
```

## Formularios

En los formularios, los elementos _input_, _select_ y _textarea_ emiten eventos
**onChange** cada vez que el usuario cambia el valor del campo. Para acceder
al valor actual, podemos aprovechar que el evento incluye un campo _target_ con
el elemento en el que se ha producido, y dicho elemento tiene un campo _value_
con su valor actual.

```jsx
const MyForm() {
  const handleChange = e => {
    console.log('Valor:', e.target.value)
  }
  return (
    <form>
      <input onChange={handleChange} />
    </form>
  )
}
```

Por otro lado, cuando el usuario pulse intro en algún campo, o haga click en el
botón de _submit_, se producirá el evento **onSubmit** sobre el propio _form_.
Es muy importante capturar este evento, y evitar que el navegador complete un
submit real, ya que eso provocaría una recarga de la página, perdiendo todos
los datos. Para ello, utilizamos el método _preventDefault_ del evento.

```jsx
const MyForm() {
  const handleSubmit = e => {
    e.preventDefault()
    // TODO: Hacer algo con los datos...
  }
  return (
    <form onSubmit={handleSubmit}>
      ...
    </form>
  )
}
```

## Errores frecuentes

Al manejar eventos es muy importante fijarnos en qué valor pasamos en el prop,
ya que es muy fácil confundirnos y, en vez de pasar la función manejadora,
pasar el resultado de ejecutar la misma, con lo que se ejecutaría
instantáneamente en lugar de al producirse el evento.

```jsx
/* Mal: handleEvent se ejecuta antes de producirse el evento */
<button onClick={handleEvent()}>Click me</button>
/* Correcto: */
<button onClick={handleEvent}>Click me</button>
```

Además, la cosa se complica aún más cuando queremos utilizar una arrow function
para pasar datos al manejador de eventos (habitual, por ejemplo, en bucles). En
este caso es importante fijarnos para no ejecutar el manejador antes de tiempo.

```jsx
const list = ['Alice', 'Bob', 'Carol', 'Dave']
const MyList = () => {
  const handleClick = (name) => {
    console.log('Click en:', name)
  }
  return (
    <div className="my-list">
      {list.map(name =>
        <li key={name} onClick={() => handleClick(name)}>
          {name}
        </li>
      )}
    </div>
  )
}
```

En el ejemplo de arriba, necesitamos pasar a handleClick el nombre sobre el que
se ha hecho click (ya que, de otro modo, no tendría acceso), pero si pusiésemos
simplemente: `onClick={handleClick(name)}`, se ejecutaría antes de tiempo. Para
evitarlo, es habitual crear una arrow function directamente ahí. De este modo,
_handleClick_ se ejecutará al ejecutarse la arrow function, en el momento que
se produzca el evento.

## A continuación...

Para poder hacer cosas interesantes con los eventos, necesitamos empezar a
asignar un [estado](./06-state.md) a nuestros componentes.
